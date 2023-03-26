## Demo Project - Dynamic Application Version Bump

### Topics of the Demo Project
Dynamically increment application version in Jenkins pipeline

### Technologies Used
- Jenkins
- Docker
- GitLab / GitHub
- Git
- Java
- Maven

### Project Description
- Configure CI step: Increment patch version
- Configure CI step: Build Java application and clean old artifacts
- Configure CI step: Build Image with dynamic Docker Image Tag
- Configure CI step: Push Image to private DockerHub repository
- Configure CI step: Commit version update of Jenkins back to Git repository
- Configure Jenkins pipeline to not trigger automatically on CI build commit to avoid commit loop

#### Steps to Increment Patch Version
In the Jenkinsfile of the Java-Maven-App add a stage before the build stage, that increments the version (on the branches using a shared library, add the logic to the shared library):
```groovy
stages {
  stage("Increment Version") {
    steps {
      script {
        echo 'incrementing the patch version of the application...'
        sh 'mvn build-helper:parse-version versions:set \
              -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
              versions:commit'
      }
    }
  }
  stage("Build Application JAR") {
    ...
  }
  stage("Build and Publish Docker Image") {
    ...
  }
}
```

#### Steps to Build Java Application and Clean Old Artifacts
Replace the command `mvn package` in the "Build Application JAR" stage with `mvn clean package`:
```groovy
stages {
  stage("Increment Version") {
    ...
  }
  stage("Build Application JAR") {
    steps {
      script {
        echo "building the application..."
        sh 'mvn clean package'
      }
    }
  }
  stage("Build and Publish Docker Image") {
    ...
  }
}
```
This step is necessary to make sure we always have just one application jar version in the target folder, which makes it easier to select that jar in the Dockerfile to copy it into the image.

#### Steps to Build Image with Dynamic Docker Image Tag and Push it to Private DockerHub Repository
Step 1: Set an environment variable holding the dynamic image tag\
We can use the new application version for tagging the Docker image created later in the pipeline. To do this, we extend the script in the "Increment Version" stage as follows:
```groovy
stage("Increment Version") {
  steps {
    script {
      echo 'incrementing the bugfix version of the application...'
      sh 'mvn build-helper:parse-version versions:set \
          -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
          versions:commit'
      
      def version = sh script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true
      env.IMAGE_TAG = "$version-$BUILD_NUMBER"
    }
  }
}
```
`mvn help:evaluate -Dexpression=project.version` evaluates the project version and prints it to stdout, together with other maven output. The `-q` (quiet) flag suppresses the output. Unfortunately the output to stdout is suppressed too. With `-DforceStdout` the final output is just the content of the version tag.\
We also append the current build number to the application version. `$BUILD_NUMBER` is an environment varibale provided by Jenkins.

Step2: Use image tag\
In the "Build and Publish Docker Image" stage we replace the hardcoded image version with `${IMAGE_TAG}`:
```groovy
stage("Build and Publish Docker Image") {
  steps {
    script {
      withCredentials([usernamePassword(credentialsId: 'DockerHub', usernameVariable: 'DOCKER_HUB_USERNAME', passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
        echo "building the docker image..."
        sh "docker build -t fsiegrist/fesi-repo:devops-bootcamp-java-maven-app-${IMAGE_TAG} ."
                        
        echo "publishing the docker image..."
        sh "echo $DOCKER_HUB_PASSWORD | docker login -u $DOCKER_HUB_USERNAME --password-stdin"
        sh "docker push fsiegrist/fesi-repo:devops-bootcamp-java-maven-app-${IMAGE_TAG}"
      }
    }
  }
}
```

Step 3: Adjust Dockerfile\
In the Dockerfile we use the hardcoded JAR version 1.0.0 when copying the JAR into the image and defining the command to start the application:
```sh
COPY ./target/java-maven-app-1.0.0.jar /opt/bootcamp-java-maven-app

CMD ["java", "-jar", "/opt/bootcamp-java-maven-app/java-maven-app-1.0.0.jar"]
```

This won't work anymore since Jenkins is incrementing the version in every build. Because we cleaned the target folder in every build, we can easily select the JAR file without knowing its version as follows:
```sh
COPY ./target/java-maven-app-*.jar /opt/bootcamp-java-maven-app

CMD java -jar /opt/bootcamp-java-maven-app/java-maven-app-*.jar
```

#### Steps to Commit Version Update of Jenkins Back to Git Repository
Step 1: Add a new stage "Commit Version Update" to the Jenkinsfile\
We add the new stage after the the "Build and Publish Docker Image" stage because we don't want to commit the version bump if something went wrong while building or publishing the image.
```groovy
stage("Build and Publish Docker Image") {
  ...
}
stage('Commit Version Update') {
  steps {
    script {
      withCredentials([usernamePassword(credentialsId: 'GitHub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
        sh "git remote set-url origin https://${USERNAME}:${PASSWORD}@github.com/fsiegrist/devops-bootcamp-java-maven-app.git"
        sh 'git add .'
        sh 'git commit -m "jenkins: version bump'
        sh 'git push origin HEAD:main'
      }
    }
  }
}
```
We have to use `git push origin HEAD:main` (`<src>:<dest>`) instead of `git push origin main` or just `git push` because Jenkins does not check out a branch but a commit.

Step 2: Configure username and email for Git\
To prevent Git from complaining (when doing a commit) that no author's email has been configured, we have to ssh into the Jenkins host server and execute the following commands:
```sh
docker exec -it <jenkins-container-id> bash
  git config --global user.email "jenkins@example.com"
  git config --global user.name "jenkins"
  exit
```

#### Steps to Configure Jenkins Pipeline to Not Trigger Automatically on CI build Commit
Pipeline: Configure > Pipeline > Git > Additional Behaviours: add "Polling ignores commits from certain users" and exclude user "jenkins"
Multibranch Pipeline: 
Step 1: Install a Jenkins plugin
**GitHub** Install a plugin called "GitHub Commit Skip SCM Behaviour".\
**GitLab** Install a plugin called "Ignore Committer Strategy". 

Step 2: Configure the pipeline\
**GitHub** The installed plugin lets you configure additional behaviours in the Git configuration. Go to  choose "Polling ignores commits from certain users") and enter the username of the committer to be ignored for triggering a build (`jenkins` in our case).\
**GitLab** The installed plugin lets you configure an email address of a committer that will be ignored for triggering a build (`jenkins@example.com` in our case).
