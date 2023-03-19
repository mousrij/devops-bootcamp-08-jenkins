## Demo Project - Jenkins Shared Library

### Topics of the Demo Project
Create a Jenkins Shared Library

### Technologies Used
- Jenkins
- Groovy
- Docker
- Git
- Java
- Maven

### Project Description
Create a Jenkins Shared Library to extract common build logic:
- Create a separate Git repository for the Jenkins Shared Library project
- Create functions in the JSL to use in the Jenkins pipeline
- Integrate and use the JSL in Jenkins Pipeline (globally and for a specific project in Jenkinsfile)

#### Steps to create a Git repository for the Shared Library project
Step 1: Create repository on GitHub\
Login to your account on [GitHub](https://github.com/fsiegrist) and create a new private repository called `devops-bootcamp-jenkins-shared-library`.

Step 2: Create a project for the JSL and push it to the repository\
Create a project called `jenkins-shared-library`, add a README file and execute the following commands:
```sh
git init
git add README.md
git commit -m "Initial commit"
git remote add origin https://github.com/fsiegrist/devops-bootcamp-jenkins-shared-library.git
git push -u origin main
```

#### Steps to create functions in the Shared Library
Step 1: Create a shared library function for the `Build Application JAR` stage\ 
Add a `vars` folder to the JSL project and within this folder a file called `buildJar.groovy` with the following content:
```groovy
#!/usr/bin/env groovy
def call() {
  echo "building the application..."
  sh 'mvn package'
}
```

Step 2: Create a shared library function for the `Build and Publish Docker Image` stage\ 
Add a `src/com/example` folder to the JSL project and within this folder a file called `Docker.groovy` with the following content:
```groovy
#!/usr/bin/env groovy
package com.example

class Docker implements Serializable {
  
  def script

  Docker(script) {
    this.script = script
  }

  def buildImage(String imageName) {
    script.echo 'building the docker image...'
    script.sh "docker build -t $imageName ."
  }

  def login() {
    script.echo 'login to DockerHub...'
    script.withCredentials([
      script.usernamePassword(credentialsId: 'DockerHub', usernameVariable: 'USERNAME', passwordVariable: 'PWD')
    ]) {
      script.sh "echo $script.PWD | docker login -u $script.USERNAME --password-stdin"
    }
  }

  def publishImage(String imageName) {
    script.echo 'publishing the docker image...'
    script.sh "docker push $imageName"
  }
}
```

Now add a second file called `buildAndPublishImage.groovy` to the `vars` folder with the following content:
```groovy
#!/usr/bin/env groovy
import com.example.Docker

def call(String imageName) {
  docker = new Docker(this)
  docker.buildImage(imageName)
  docker.login()
  docker.publishImage(imageName)
}
```

Commit the changes and push to the JSL Git repository.

#### Steps to define the JSL globally and use it in the Jenkins Pipeline
Step 1: Define the JSL globally\
In Jenkis go to "Dashboard" > "Manage Jenkins" > "Configure System" > "Global Pipeline Libraries" and add a new configuration. Enter a name (e.g. `jenkins-shared-library`) and `main` as the default version. Under "Retrieval method" select "Modern SCM". Under "Source Code Management" select "Git", enter the repository URL and select the cedentials. Press the "Save" button.

Step 2: Use the library in the Jenkins Pipeline\
Replace the content of the Jenkinsfile in the Java-Maven-App with the following:
```groovy
#!/usr/bin/env groovy

@Library('jenkins-shared-library')_

pipeline {
  agent any
  tools {
    maven 'maven-3.9'
  }
  stages {
    stage("Build Application JAR") {
      steps {
        script {
          buildJar()
        }
      }
    }
    stage("Build and Publish Docker Image") {
      steps {
        script {
          buildAndPublishImage 'fsiegrist/fesi-repo:devops-bootcamp-java-maven-app-1.0.2'
        }
      }
    }
  }
}
```

#### Steps to use the JSL in a Jenkins Pipeline (Project Scope)
Step 1: 

Step 2: 