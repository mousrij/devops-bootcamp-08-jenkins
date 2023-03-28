## Notes on the videos
<br />

<details>
<summary>Video: 1 - Intro to Build Automation</summary>
<br />

Every time you push code changes to the remote git repository, the following steps should be executed automatically:

- checkout the source code
- build the application
- run tests
- build artifacts (e.g. docker image)
- publish artifacts (e.g. to docker registry) -> CI
- deploy artifacts -> CD
- (send notifications)
- (...)

All of these tasks can be controlled/managed by a build automation tool like [Jenkins](https://www.jenkins.io/). There are other similar tools like [Travis](https://www.travis-ci.com/), [GitLab](https://about.gitlab.com/), [Bamboo](https://www.atlassian.com/de/software/bamboo), [TeamCity](https://www.jetbrains.com/teamcity), etc.

</details>

*****

<details>
<summary>Video: 2 - Install Jenkins</summary>
<br />

### Install Jenkins as a Docker container

See the [Jenkins Documentation](https://www.jenkins.io/doc/book/installing/docker/).

**Create a Droplet on DigitalOcean**

Login to your DigitalOcean account an create a new Droplet (4GB RAM). Jenkins needs at least 1GB RAM. Change the Droplet's name to something like 'jenkins-server' and attach a firewall rule-set to it opening port 22 for SSH from your machine's IP address and port 8080 (Type=Custom) for Jenkins from all IP addresses.

SSH into the Droplet (`ssh root@<droplet-ip>`) and install Docker (since we want to run Jenkins in a Docker container):
```sh
apt update
apt install docker.io
```

Start Jenkins in a Docker container:
```sh
docker run -p 8080:8080 -p 50000:50000 -d -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
# Jenkins master and worker nodes communicate over port 50000
```

Copy the initial administrator password:
```sh
# in the docker container
docker exec -it <container-id> bash
cat /var/jenkins_home/secrets/initialAdminPassword

# or directly on the host
docker volume inspect jenkins_home
# the path is displayed as "Mountpoint" -> /var/lib/docker/volumes/jenkins_home/_data
cat /var/lib/docker/volumes/jenkins_home/_data/secrets/initialAdminPassword
```

Open Jenkins in the browser under `http://<droplet-ip>:8080`, enter the initial administrator password and install the suggested plugins. After creating a first admin user, the initialization of Jenkins is done and the welcome screen is displayed.

</details>

*****

<details>
<summary>Video: 3 - Introduction to Jenkins UI</summary>
<br />

- **Administrators** see the "Manage Jenkins" section, from where they can set up Jenkins clusters, install plugins, create users, backup data etc.
- **Users** use the "New Item" section to create jobs, and pipelines to run the required workflows.

</details>

*****

<details>
<summary>Video: 4 - Install Build Tools in Jenkins</summary>
<br />

To execute maven builds or run npm tests, these tools have to be installed. There are two ways to install tools used by Jenkins:
- install a plugin for that tool (through the Jenkins UI)
- install the tool directly on the server on which Jenkins is running (or in the container if Jenkins is running inside a container)

### Configure Maven Plugin
For most of the usual build tools a related plugin is already installed. For Maven this is the case too. So we just have to configure the already installed plugin.

Go to the "Manage Jenkins" section and click on "Global Tool Configuration".
Click on the "Add Maven" buttton, enter a name (e.g. maven-3.6) and click on "Save". Now you have the maven command available in all Jenkins jobs.

### Install npm and Node in the Jenkins container
Enter the docker container as root user (because we need the privilege to install tools):
```sh
docker exec -u 0 -it <container-id> bash
```

Install curl:
```sh
apt update
apt install curl
```

Install node:
```sh
# get information about the current Linux distribution
cat /etc/issue 
# => e.g. 'Debian GNU/Linux 9 \n \l'

# lookup the matching URL to download a node install script and execute the following curl command
curl -sL https://deb.nodesource.com/setup_10.x -o nodesource_setup.sh

# execute the install script and install node and npm
bash nodesource_setup.sh
apt install nodejs
```

</details>

*****

<details>
<summary>Video: 5 - Jenkins Basics Demo - Freestyle Job</summary>
<br />

### Create simple freestyle job 
Click on "New Item" and enter a name for the new job (e.g. my-job). Select "Freestyle Project" and click "Ok".\
Under the "Build" section at the bottom, select "Execute shell" from the drop down list. You can execute any shell command you would also be able to execute directly on the shell of the server/container where Jenkins is installed. Enter the command `npm --version`.\
Add another build step from the drop down: "Invoke top-level Maven targets". Select the version (only maven-3.6 is available) and enter the "goal" `--version`. Click "Save" to save the freestyle job.

In the Jenkins main view (click "Jenkins" at the top left of the screen) you see the new job called "my-job". Click on it and click on "Build Now" in the menu on the left. When the build has finished, click on the build item (build number) on the bottom left and then on "Console Output" in menu on the left to see the output of the two commands (`npm --version` and `mvn --version`).

### Plugin configuration
In order for a tool to appear in the list of available build tools, it has to be installed as a plugin first. Go to the Jenkins main view, select "Manage Jenkins" > "Manage Plugins" > "Available" and search for 'nodejs', select it and click on "Install without restart".\
Go to "Dashboard" > "Manage Jenkins" > "Global Tool Configuration" where you will find the additional NodeJS build tool. To make it available in build jobs, you first have to configure it. Click on "Add NodeJS" and configure it similar to how you configured the maven plugin before.

### Configure Git Repository
Go to "Dashboard" > "my-job" > "Configure" > "Source Code Management" and select the "Git" radio button. Enter the repository URL and select the credentials. If you don't have configured the credentials yet, you can add them by clicking the "Add" drop down and selecting "Jenkins". This will open a dialog where you can add the credentials for the repository. Select the kind "Username with password", enter the credentials, enter an ID (e.g. gitlab-credentials) and click on "Add". Now the credentials are available in the drop down, so select them and finish by clicking on "Save" at the bottom.

If you run the build again and read the console output, you can see that Jenkins fetched the content of the repository before executing the build commands.

### Jenkins Directory Structure
- The job related files (like for example build log files) are stored in /var/jenkins_home/jobs/my-job.
- The sources checked out from the git repository are stored in /var/jenkins_home/workspace/my-job.

### Do something from Git Repo in Jenkins Job
If the checked out files contain a shell script `<script-file.sh>`, it can be executed during the build. Go to "Dashboard" > "my-job" > "Configure" > "Build" and add the command to execute the shell script ("Execute shell" > "Command": `./<script-file.sh>`). For Jenkins to have the permissions to execute the script, you have to provide them first (`chmod +x <script-file.sh>`).

### Run Tests and build Java Application
Create a new freestyle job (called 'java-maven-build'). Configure the git repository URL and add two maven plugin build steps executing the goals `test` and then `package`. After the build has run, you can find the jar file under /var/jenkins_home/workspace/java-maven-build/target.

</details>

*****

<details>
<summary>Video: 6 - Docker in Jenkins</summary>
<br />

### Make Docker available in Jenkins
To create Docker images during the builds, Jenkins needs to have access to the docker command. Instead of installing Docker inside the Jenkins container, we can mount the Docker runtime of the host system as a volume.

To do that, stop the running Jenkins container and start it again with the following command:
```sh
docker run -p 8080:8080 -p 50000:50000 -d \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v $(which docker):/usr/bin/docker \
  jenkins/jenkins:lts
```

Now the docker command is available in the Jenkins container too. However, the user `jenkins` (under which Jenkins is running) has no read/write permissions on the file `/var/run/docker.sock`. So we have to enter the Jenkins container as root user and provide the missing permissions to the user `jenkins`:
```sh
# enter the docker container as root user
docker exec -u 0 -it <container-id> bash

  # provide missing permissions and exit
  chmod 666 /var/run/docker.sock
  exit

# check if jenkins user can execute docker commands
docker exec -it <container-id> bash
  docker pull hello-world
  exit
```

Now Jenkins can use the `docker` command in builds.

### Build Docker Image
Add a Dockerfile to your project sources (in the Git repository), which builds a Docker image from the final (maven, gradle, npm, etc.) build artifact.

In the Jenkins job add an "Execute shell" step to the build steps and enter the command `docker build -t java-maven-app:1.0 .`.

### Push image to DockerHub
Sign in to your account on [DockerHub](https://hub.docker.com/) and create a private repository (if you don't already have one).

For Jenkins to be able to push images to this repository, we need to configure the credentials. Go to "Dashboard" > "Manage Jenkins" > "Manage Credentials" > "Stores scoped to Jenkins" > "Jenkins" > "Global credentials" > "Add credentials" and enter your DockerHub username and password and an ID (e.g. docker-hub-repo).

Now go back to the Jenkins build configuration and jump to the "Build Environment" section, select "Use secret text(s) or file(s)", add a "Username and password (separated)" binding, define the names of the environment variables holding the username and password (e.g. DOCKER_HUB_USERNAME and DOCKER_HUB_PASSWORD) and select the correct credentials. Now scroll down to the "Build" section (Execute shell), adjust the tag name of the applications image (`docker build -t <your/private-repo-name:version> .`) and add commands to login and push the image to the private repository:
```sh
echo $DOCKER_HUB_PASSWORD | docker login -u $DOCKER_HUB_USERNAME --password-stdin
docker push <your/private-repo-name:version>
```

Save the build configuration and run the build. Then go to your private repository on [DockerHub](https://hub.docker.com/) and check if the new image got pushed.

### Push Docker Image to Nexus Repository
Because our Nexus repository is accessible via http (not https) we have to add the "insecure-registries" configuration to the host running the Jenkins container. SSH into the droplet running the Jenkins container and create a file `/etc/docker/daemon.json` with the following content:
```sh
{
  "insecure-registries": ["<nexus-droplet-ip>:8083"]
}
```

Now restart Docker executing `systemctl restart docker` and restart the Jenkins container (it was stopped when restarting Docker): `docker start <container-id>`. Finally we have to re-grant read/write permissions on the file /var/run/docker.sock (this change was lost when the container was stopped):
```sh
# enter the docker container as root user
docker exec -u 0 -it <container-id> bash

  # provide missing permissions and exit
  chomd 666 /var/run/docker.sock
  exit
```

To let Jenkins push images to the Nexus repository, we have to configure the credentials (as we did before for DockerHub). In the shell command we adjust the image name to `<nexus-ip:8083/image-name:version>` and add `<nexus-ip:8083>` at the end of the login command. We also have to bind the username and password environment variables with the correct credentials (and maybe rename them).

</details>

*****

<details>
<summary>Video: 7 - Freestyle to Pipeline Job</summary>
<br />

At the end of the build configuration page of a freestyle job you'll find a dropdown "Add post-build action" with the option "Build other project". Using this option you can chain together multiple freestyle jobs. This lets you divide the whole build workflow into smaller pieces.

However there is a better way to achieve this goal: Pipeline projects. They are more suitable for creating CI/CD pipelines and let you specify the whole build workflow in a script instead of using the Jenkins GUI to assemble various build plugins.

</details>

*****

<details>
<summary>Video: 8 - Introduction to Pipeline Job</summary>
<br />

Go to the Jenkins main view and click on "New Item", enter a name (e.g. my-pipeline), and select the "Pipeline" project.

In the build configuration page jump to the "Pipeline" section to connect the build to a Git repository. Pipeline jobs are defined and configured using a Groovy script. You can write the script directly on the configuration page choosing "Pipeline script" from the "Definition" dropdown. However it is recommended to have the script in your application project and let Jenkins execute it after it has been checked out from the SCM (source code managment). This is the second option "Pipeline script from SCM" in the "Definition" dropdown.\
Select "Git" from the "SCM" dropdown, enter the repository URL, select the credentials and enter the branch you want to check out. In the "Script path" form field leave the pre-set value "Jenkinsfile" unchanged. This will let Jenkins search for a file called "Jenkinsfile" in the root folder of the project and execute it.

### Jenkinsfile
Jenkinsfiles can be written in scripted style or in declarative style.

Scripted style:
```groovy
node {
  // any Groovy script
}
```

Declarative style (predefined structure):
```groovy
pipeline {
  agent any // agent defines where this script should be executed (relevant on Jenkins clusters)
  stages {
    stage("build") {
      steps {
        echo 'building the application...'
      }
    }
    stage("test") {
      steps {
        echo 'testing the application...'
      }
    }
    stage("deploy") {
      steps {
        echo 'deploying the application...'
      }
    }
  }
}
```

After running the build process the status of the different stages are displayed in the UI.

</details>

*****

<details>
<summary>Video: 9 - Jenkinsfile Syntax</summary>
<br />

### Attributes in Jenkinsfile
**Post actions**
```groovy
pipeline {
  agent ...
  stages {
    ...
  }
  post { // execute some logic after all stages have completed
    always {
      // e.g. send an email
    }
    success {
      ...
    }
    failure {
      ...
    }
  }
}
```

**Define conditionals for each stage**
```groovy
stages {
  stage("test") {
    when {
      expression {
        env.BRANCH_NAME == 'dev' || env.BRANCH_NAME == 'master'
      }
    }
    steps {
      ...
    }
  }
}
```

**Environment variables**\
What variables are available from Jenkins?\
Open the URL `http(s)://<jenkins-host-ip>:8080/env-vars.html` in your browser.

You can define your own variables available for all stages in the environment block:
```groovy
environment {
  NEW_VERSION = calculateVersion()
}
stages {
  stage("build") {
    steps {
      echo 'building the application...'
      echo "building version ${NEW_VERSION}"
    }
  }
}
```

**Using credentials**\
Precondition: The credentials plugin and the credentials binding plugin must be intalled.
```groovy
environment {
  SERVER_CREDENTIALS = credentials('<credentials-id>')
}
stages {
  stage("build") {
    steps {
      sh "... ${SERVER_CREDENTIALS} ..."
    }
  }
  stage("deploy") {
    steps {
      echo 'deploying the application...'
      withCredentials([
        usernamePassword(credentials: '<credentials-id>', usernameVariable: 'USER', passwordVariable: 'PWD')
      ]) {
        sh "... ${USER} ... ${PWD}..."
      }
    } 
  }
}
```

**Access build tools (maven, gradle, jdk)**
```groovy
tools {
  maven 'maven-3.6'
  gradle ...
  jdk ...
}
stages {
  stage("build") {
    steps {
      sh "mvn clean package"
    }
  }
}
```

**Parameterize your build**
```groovy
parameters {
  string(name: 'VERSION', defaultValue: '1', description: '...')
  choice(name: 'VERSION', choices: ['1.1', '1.2', '1,3'], description: '...')
  boolenParam(name: 'executeTests', defaultValue: true, description: '...') 
}
stages {
  stage("test") {
    when {
      expression {
        params.executeTests
      }
    }
    steps {
      ...
    }
  }
}
```

If parameters are defined in the Jenkinsfile, the menu item "Build" will change to "Build with Parameters" and provide a possibility to set these parameters before executing the build.

### Using external Groovy scripts
Externalize build logic in separate Groovy scripts. At the end of the script you have to add the command `return this`, otherwise the script cannot be imported into the Jenkinsfile:
```groovy
def build() {
  echo 'building the application...'
}
def test() {
  echo 'testing the application...'
}
return this
```

In the Jenkinsfile you can import and use the script like this:
```groovy
def gv

pipeline {
  agent ...
  stages {
    stage("init") {
      script {
        gv = load "script.groovy"
      }
    }
    stage("build") {
      steps {
        script {
          gv.build()
        }
      }
    }
    stage("test") {
      steps {
        script {
          gv.test()
        }
      }
    }
  }
}
```

Note: When you open a build, that has already been executed, there is a "Replay" menu item on the left. This lets you edit the Jenkinsfile *and all imported external Groovy scripts* before re-executing the build. This comes in very handy when you want to try out changes on the Jenkinsfile/Groovy scripts without having to push them into the Git repository.

### User Input
```groovy
stage("deploy") {
  input {
    message "Select the environment to deploy to.
    parameters {
      choice(name: 'ENV', choices: ['dev', 'stage', 'prod'], description: '...')
    }
  }
  steps {
    script {
      echo "Deploying to ${ENV}" // without the params-prefix here
    }
  }
}
```

When the build is executed and reaches the deploy stage, it is paused and waits for user input. To provide the input, hover over the paused stage's area in the stages view and enter the required values.

</details>

*****

<details>
<summary>Video: 10 - Create complete Pipeline</summary>
<br />

Lets create a pipeline doing the same steps as the freestyle job in videos 5 and 6.

```groovy
pipeline {
  agent any
  tools {
    maven 'maven-3.6'
  }
  stages {
    stage("build jar") {
      steps {
        script {
          echo 'building the application...'
          sh 'mvn package'
        }
      }
    }
    stage("build image") {
      steps {
        script {
          echo 'building the docker image...'
          withCredentials([
            usernamePassword(credentials: '<credentials-id>', usernameVariable: 'USERNAME', passwordVariable: 'PWD')
          ]) {
            sh 'docker build -t <your/private-repo-name:version> .'
            sh "echo $PWD | docker login -u $USERNAME --password-stdin"
            sh 'docker push <your/private-repo-name:version>'
          }
        }
      }
    }
    stage("deploy") {
      steps {
        script {
          echo 'deploying the application...'
        }
      }
    }
  }
}
```

</details>

*****

<details>
<summary>Video: 11 - Introduction to Multibranch Pipeline</summary>
<br />

Most of the time development is done on multiple branches in parallel: the main development is done on the master branch, while there may be branches for bug-fixes or features. While you want to execute tests on all these branches, only one branch is going to be deployed.

So we need pipelines for all the branches, but different behaviour based on the branch that is being built.

We also want a new pipeline to be created automatically as soon as a new branch is pushed to the version control system.

That's exactly what the "Multibranch Pipeline" project type is for. Create one clicking on "New Item" > "Multibranch Pipeline". In the "Branch Sources" section you can enter the Git repository URL, the credentials, and add a behaviour for branch discovery (e.g. "filter by name (with regular expression)").

After saving the new project, Jenkins scans all the branches in the specified repository for a Jenkinsfile. If a Jeninsfile is found, a build pipeline is created based on the content of the Jenkinsfile. To suppress building a branch either adjust the regular expression used to select the branches or remove/rename the Jenkinsfile from the branch.

### Branch-based logic for Multibranch Pipeline
You don't have to write different Jenkinsfiles for each branch. All branches may have the same Jenkinsfile, and if you need to perform different build logic based on the branch that is currently built, it is possible to do so in the Jenkinsfile:

```groovy
pipeline {
  agent any
  stages {
    stage("build") {
      steps {
        script {
          echo 'building the application...'
        }
      }
    }
    stage("deploy") {
      when {
        expression {
          // BRANCH_NAME is an env variable which is set by Jenkins in multibranch pipelines
          BRANCH_NAME == 'master'
        }
      }
      steps {
        script {
          echo 'deploying the application...'
        }
      }
    }
  }
}
```

</details>

*****

<details>
<summary>Video: 13 - Credentials in Jenkins</summary>
<br />

Credentials are associated to three different scopes:
- System: They are created from wherever they are needed ("Add"-button) or in "Manage Jenkins" > "Manage Credentials" > "Stores scoped to Jenkins" > "Jenkins (Store) / global (Domains)" > "Add Credentials" > "Scope: System". System credentials are only available on Jenkins server, but not from any jobs/projects. They are defined by Jenkins administratores.
- Global: They are created in the same way as system credentials, except for the last step where the "Scope: Global" is chosen. Global credentials are visible in all the jobs/projects. They are defined by Jenkins users creating jobs/projects.
- Multibranch Pipeline: They are created from within a multibranch pipeline project, where you have a "Credentials" item in the menu on the left. It opens a credentials overview similar to the one where you manage the system credentials or global credentials, but with an additional section "Stores scoped to my-multibranch-pipeline". Clicking on the domain-link (global) in this section and then "Add Credentials" opens the same form to enter the credentials, just without the option to choose a scope, as the scope is the multibranch-pipeline folder. Credentials defined here are only visible from within pipelines of this project. Other projects cannot access them in their build steps.

</details>

*****

<details>
<summary>Video: 14 - Jenkins Shared Library</summary>
<br />

Shared libraries are used to store build logic  that can be reused in different build pipelines of different projects. They are an extension to the pipeline, are written in Groovy as the Jenkinsfile, use their own (Git) repository, and are referenced from within the Jenkinsfiles of the different projects.

Create a new Groovy project (in your IDE) with its own repository. The structure of Shared Libraries projects is the following:
- `vars` folder: Groovy functions that are called from the Jenkinsfile; each function has to be in its own individual Groovy file
- `src` folder: helper code
- `resources` folder: external libraries, non Groovy files

Add a `vars` folder to the project and within this folder a file called `buildJar.groovy`. The name of the file (without extension) is the same as the name of the function to be called from within the Jenkinsfile. Write the following file content:
```groovy
#!/usr/bin/env groovy
def call() {
  echo "building the application..."
  sh 'mvn package'
}
```

Add a second file called `buildAndPublishImage.groovy` to the `vars` folder with the following content:
```groovy
#!/usr/bin/env groovy
def call() {
  echo 'building the docker image...'
  withCredentials([
    usernamePassword(credentials: '<credentials-id>', usernameVariable: 'USERNAME', passwordVariable: 'PWD')
  ]) {
    sh 'docker build -t <your/private-repo-name:version> .'
    sh "echo $PWD | docker login -u $USERNAME --password-stdin"
    sh 'docker push <your/private-repo-name:version>'
  }
}
```

Create a Git repository (e.g. on GitHub or GitLab) and push the shared libraries project to it.

### Make the Shared Library globally available in Jenkins
Go to "Dashboard" > "Manage Jenkins" > "Configure System" > "Global Pipeline Libraries" and add a new configuration. Enter a name (e.g. `jenkins-shared-library`) and a default version (branch, commit hash or tag). Under "Retrieval method" select "Modern SCM". Under "Source Code Management" select "Git", enter the repository URL and select the cedentials. Press the "Save" button.

### Use Shared Library in Jenkinsfile
Add `@Library('jenkins-shared-library')_` at the beginning of your Jenkinsfile (before `pipeline {`). If you have other definitions before the pipeline (e.g. definition of a local Groovy script `def gv`) you can omit the trailing underscore. If you want to override the default version of the library, you can add the required version like this: `@Library('jenkins-shared-library@2.0')_`

Now you can call the shared library function directly by name, e.g.
```groovy
stage("build jar") {
  steps {
    script {
      buildJar()
    }
  }
}
stage("build image") {
  steps {
    script {
      buildAndPublishImage()
    }
  }
}
```

### Using Parameters in Shared Library
Let's say we want to pass in the `<your/private-repo-name:version>` string as a paramater to the shared library function `buildAndPublishImage`. To do this, replace the content of the file called `buildAndPublishImage.groovy` with the following:
```groovy
#!/usr/bin/env groovy
def call(String imageName) {
  echo 'building the docker image...'
  withCredentials([
    usernamePassword(credentialsId: '<credentials-id>', usernameVariable: 'USERNAME', passwordVariable: 'PWD')
  ]) {
    sh "docker build -t $imageName ."
    sh "echo $PWD | docker login -u $USERNAME --password-stdin"
    sh "docker push $imageName"
  }
}
```
Now the image name can be passed in as a parameter when calling it from within the Jenkinsfile: `buildAndPublishImage '<your/private-repo-name:version>'`

Note: All **environment variables**, that are available in a Jenkinsfile are also available in a shared library function.

### Extract Logic into Groovy Classes
In order to avoid duplicate code in the various shared library functions inside the `vars` folder, we can extract logic into Groovy classes inside the `src` folder.

Example class holding the logic of the `buildAndPublishImage` function:
```groovy
#!/usr/bin/env groovy
package com.example

class Docker implements Serializable {
  
  def script

  Docker(script) {
    this.script = script
  }

  def buildAndPublishImage(String imageName) {
    script.echo 'building the docker image...'
    script.withCredentials([
      script.usernamePassword(credentialsId: '<credentials-id>', usernameVariable: 'USERNAME', passwordVariable: 'PWD')
    ]) {
      script.sh "docker build -t $imageName ."
      script.sh "echo $script.PWD | docker login -u $script.USERNAME --password-stdin"
      script.sh "docker push $imageName"
    }
  }
}
```

Let the classes implement `Serializable` to support saving the state if a pipeline is paused and resumed. Note that the Jenkinsfile DSL is not available in Groovy classes. That's why we have to pass in a `script` parameter to the constructor. Via this parameter the DSL functions are accessible. The same is true for environment variables.

Now the content of the `buildAndPublishImage.groovy` file can be replaced with the following:
```groovy
#!/usr/bin/env groovy
import com.example.Docker

def call(String imageName) {
  return new Docker(this).buildAndPublishImage(imageName)
}
```

### Make the Shared Library available only in Project Scope
You can also directly import shared libraries into your Jenkinsfiles without making them globally available. Instead of adding `@Library('jenkins-shared-library')_` at the beginning of your Jenkinsfile, do the following:
```groovy
#!/usr/bin/env groovy

library identifier: 'jenkins-shared-library@1.0', retriever: modernSCM(
  [
    $class: 'GitSCMSource',
    remote: '<repository URL>',
    credentialsId: '<credentials-id>'
  ]
)

pipeline {
  ...
}
```

</details>

*****

<details>
<summary>Video: 15 - Webhooks: Trigger Pipeline Jobs automatically</summary>
<br />

Usually you want the build pipelines to be triggered automatically whenever changes are pushed to the Git repository. To reach that goal the Git repository (GitLab, GitHub, etc.) has to be configured accordingly.

Automatic triggering of the build pipeline may also be configured to happen on a scheduler basis (e.g. once an hour). This may make sense for builds including long running tests.

Manually starting the build may make sense for a build that deploys the build artifact to a production server.

### Automatically Trigger the Build whenever Changes Happen in the Git Repository
Go to "Dashboard" > "Manage Jenkins" > "Manage Plugins" > "Available plugins" and search for "gitlab". Select the "GitLab (Build Triggers)" plugin and press the "Install without restart" button.

Now that the plugin is installed, go to "Dashboard" > "Manage Jenkins" > "Configure System" where you will find a "GitLab" section. Make sure "Enable authentication for '/project' endpoint" is checked, enter a connection name (e.g. gitlab-conn) and the GitLab host URL ('https://gitlab.com'). To add an API token for GitLab access, click on "Add" > "Jenkins". Select the kind "GitLab API token". To get the access token go to your GitLab account, open your profile and select "Access Token" on the left. Enter a name of the personal access token (e.g. jenkins), an expiration date, and select the "api" scope, before pressing the "Create personal API token" button. Copy the generated access token and paste it into the Jenkins form field "Api token". Enter an ID (e.g. GitLab API token) and press the "Add" button. Now you can select the token from the credentials dropdown and press the "Save" button.

When you open the configuration of a build pipeline, you'll find a GitLab connection configured as well as automatically enabled GitLab build triggers.

The second part is to configure GitLab so that it triggers Jenkins whenever code changes are pushed to the repository. So go back to your GitLab project and click on "Settings" > "Integrations" > "Jenkins CI". Enable the integration, select "Push" for the trigger, enter the Jenkins URL (port incl.), the Jenkins pipeline name as project name, and username / password of your Jenkins account. Press the "Save changes" button.

#### Additional Configurations for Multibranch Pipelines
To enable automatic triggering of builds for multibranch pipeline projects, some additional steps are required. Go to "Dashboard" > "Manage Jenkins" > "Manage Plugins" > "Available plugins" and search for "multibranch scan". Select the "Multibranch Scan Webhook Trigger" plugin and press the "Install without restart" button.

Now open the configuration of your multibranch pipeline project and scroll down to the "Scan Multibranch Pipeline Triggers" section. There you'll find an additional checkbox "Scan by webhook", that was added by the plugin. Select it and enter a trigger token. This can be any name (e.g. gitlabtoken). Click on the question mark on the right border belonging to the trigger token and copy the webhook URL description (starting with JENKINS_URL). Save the configuration.

Next go back to your GitLab account and open "Settings" > "Webhooks". Paste the copied URL to the URL field and replace `JENKINS_URL` and `[Trigger token]` with the correct values. Select the "Push events" trigger and press the "Add webhook" button.

</details>

*****

<details>
<summary>Video: 16 - Dynamically Increment Application Version in Jenkins Pipeline (Part 1)</summary>
<br />

### How to increment the version locally using Maven
```sh
# parse the version string of the current maven project and set properties containing
# the component parts of the version. This mojo sets the following properties:
# parsedVersion.majorVersion
# parsedVersion.minorVersion
# parsedVersion.incrementalVersion
# parsedVersion.qualifier
# parsedVersion.buildNumber
# parsedVersion.nextMajorVersion
# parsedVersion.nextMinorVersion
# parsedVersion.nextIncrementalVersion
# parsedVersion.nextBuildNumber
mvn build-helper:parse-version

# set the version of the current maven project to the given value
mvn versions:set -DnewVersion=1.0.1-SNAPSHOT

# combining these two goals we can automatically increment any part of the version
mvn build-helper:parse-version versions:set \
  -DnewVersion=\${parsedVersion.majorVersion}.\${parsedVersion.minorVersion}.\${parsedVersion.nextIncrementalVersion}

# the last command will increment the bugfix version in the pom.xml;
# it will also store the last version in a pom.xml.versionsBackup file;
# revert the changes with
mvn versions:revert
# commit the changes with 
mvn versions:commit
# both commands will remove the pom.xml.versionsBackup file;
# after that, revert is no longer possible
```

### How to increment the version on Jenkins using Maven
In the Jenkinsfile add a stage before the build stage, that increments the version:
```groovy
stages {
  stage("Increment Version") {
    steps {
      script {
        echo 'incrementing the bugfix version of the application...'
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

We also want to use the new version for tagging the Docker image created later in the pipeline. To do this, we add the following commands to the script in the "Increment Version" stage (after the `sh` command):
```groovy
def matcher = readFile('pom.xml') =~ '<version>(.*)</version>'
def version = matcher[0][1] // first match, second group (group 1 is the whole expression)
env.IMAGE_VERSION = "$version-$BUILD_NUMBER" // BUILD_NUMBER is an env varibale provided by Jenkins
```

These are just regular Groovy features. The `matcher` variable holds an array of all `<version>` tags in the pom.xml file. We are just interested in the first (`matcher[0]`). Since the regular expression contains a group (`(.*)`) every item in the matcher array is itself an array containing the matcher groups. Group 0 is the whole matching string, group 1 is the first group in the regEx, and so on. So `matcher[0][1]` just contains the content of the version tag.\
We also append the current build number to the application version. `$BUILD_NUMBER` is an environment varibale provided by Jenkins.

**Note** by Felix Siegrist:\
A better way to read the version (without parsing XML using regular expressions):
```groovy
def version = sh script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true
```

In the "Build and Publish Docker Image" stage we replace the hardcoded image version with `${IMAGE_VERSION}`.

In the Dockerfile we have to replace the hardcoded JAR version 
```sh
COPY ./target/java-maven-app-1.0.0.jar /opt/bootcamp-java-maven-app

CMD ["java", "-jar", "/opt/bootcamp-java-maven-app/java-maven-app-1.0.0.jar"]
```

with the following:
```sh
COPY ./target/java-maven-app-*.jar /opt/bootcamp-java-maven-app

CMD java -jar /opt/bootcamp-java-maven-app/java-maven-app-*.jar
```

The second command will only work if there is just one jar file. To enforce this we have to replace the command `mvn package` in the "Build Application JAR" stage with `mvn clean package`.

Now we are ready to commit all the changes and trigger the build.

</details>

*****

<details>
<summary>Video: 17 - Dynamically Increment Application Version in Jenkins Pipeline (Part 2)</summary>
<br />

After Jenkins incremented the bugfix version in the pom.xml, it has to commit this change to the Git repository, otherwise it would start from the same original version and increment it in every build to the same next version.

Add a new stage "Commit Version Update" to the Jenkinsfile:
```groovy
stage('Commit Version Update') {
  steps {
    script {
      withCredentials([usernamePassword(credentialsId: 'GitHub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
        sh "git remote set-url origin https://${USERNAME}:${PASSWORD}@github.com/fsiegrist/devops-bootcamp-java-maven-app.git"
        sh 'git add .'
        sh 'git commit -m "ci: version bump"'
        sh 'git push origin HEAD:main'
      }
    }
  }
}
```
We have to use `git push origin HEAD:main` (`<src>:<dest>`) instead of `git push origin main` or just `git push` because Jenkins does not check out a branch but a commit.

To prevent Git from complaining (when doing a commit) that no author's email has been configured, we have to ssh into the Jenkins host server and execute the following commands:
```sh
docker exec -it <jenkins-container-id> bash
  git config --global user.email "jenkins@example.com"
  git config --global user.name "jenkins"
  exit
```

If we configured Jenkins to automatically trigger a new build on any push to the Git repository, we would end up in an endless build-push-build-push loop. In order to prevent this we have to detect that a commit was made by Jenkins and ignore the trigger in this case.\
To do this, we install a plugin in Jenkins called "Ignore Committer Strategy". This plugin lets you configure an email address of a committer that will be ignored for triggering a build (`jenkins@example.com` in our case). Open the configuration page for the multibranch pipeline project and scroll down to the "Branch Sources" > "Git" section. Open the "Add" dropdown for "Build strategies", select "Ignore Committer Strategy" and enter the email address of the committer, whose commits are to be ignored: `jenkins@example.com`. Also make sure the "Allow builds when a changeset contains non-ignored author(s)" checkbox is selected.

### Additional Notes by Felix Siegrist
If you only use standard pipelines (no multibranch pipelines) it is not necessary to install this plugin. Just go to the pipeline configuration and scroll down to "Additional Behaviours" in the Git configuration. Click the "Add" dropdown and choose "Polling ignores commits from certain users" and enter the username of the committer to be ignored for triggering a build (`jenkins` in our case).

Don't use the "GitHub Commit Skip SCM Behaviour" plugin or the "Skip SCM" plugin. They both seem not to work for multibranch pipelines.

Committing changes back to the project repository is problematic. And it will fail if a developer pushed commits to the repo while a Jenkins build was running. When Jenkins then tries to push its version bump to the repo, it would first have to pull the newer commit from the repo.

So maybe setting the version - even if it is just the patch version - should be something that is explicitly done by a developer. There are other ways to make sure, every build artifact (jar, docker image) gets its unique version/tag. E.g. use a pattern like `<version>-<timestamp>-<buildNumber>`.

</details>

*****
