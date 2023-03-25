Your team members want to collaborate on your NodeJS application, where you list developers with their projects. So they ask you to set up a git repository for it.

Also, you think it's a good idea to add tests, to test that no one accidentally breaks the existing code.

Moreover, you all decide every change should be immediately built and pushed to the Docker repository, so everyone can access it right away.

For that they ask you to **set up a continuous integration pipeline**.

<details>
<summary>Exercise 0: Create Git Repository</summary>
<br />

**Tasks:**

- clone the git repository `https://gitlab.com/devops-bootcamp3/node-project.git`
- create your own project/git repo from it

**Steps to solve the tasks:**

```sh
git clone https://gitlab.com/devops-bootcamp3/node-project.git
cd node-project

# remove remote repo reference
rm -rf .git
# create your own local repository and commit its content
git init 
git add .
git commit -m "Initial commit"

# create git repository on GitHub push your newly created local repository to it
git remote add origin git@github.com:fsiegrist/devops-bootcamp-node-project.git
# rename master branch of original Gitlab repository to main (default on GitHub)
git branch -M main
# push your newly created local repository to it
git push -u origin main
```

</details>

******

<details>
<summary>Exercise 1: Dockerize your NodeJS App</summary>
<br />

**Tasks:**

Configure your application to be built as a Docker image.
- Dockerize your NodeJS app

**Steps to solve the tasks:**\
Step 1: Create a Dockerfile with the following content in the project root:
```sh
FROM node:13-alpine

RUN mkdir -p /usr/app
COPY app/images/* /usr/app/images/
COPY app/index.html /usr/app/
COPY app/package.json /usr/app/
COPY app/server*.js /usr/app/

WORKDIR /usr/app
EXPOSE 3000

RUN npm install

CMD ["node", "server.js"]
```

Commit and push the Dockerfile.

</details>

******

<details>
<summary>Exercise 2: Create a full pipeline for your NodeJS App</summary>
<br />

**Tasks:**

You want the following steps to be included in your pipeline:
- Increment version\
  The application's version and docker image version should be incremented.
- Run tests\
  You want to test the code, to be sure to deploy only working code. When tests fail, the pipeline should abort.
- Build docker image with incremented version
- Push to Docker repository
- Commit to Git\
  The application version increment must be committed and pushed to a remote Git repository.

**Steps to solve the tasks:**
Step 1: Prerequisites (tools and credentials)\
For the build pipeline we need Node and NPM to be installed in the Docker container running Jenkins. We further need credentials for accessing GitHub and DockerHub. All of these have already been installed and configured on Jenkins for [demo project 2](./demo-projects/2-create-ci-pipeline/).\
To read the updated version from the package.json file, we need JSON support. That's why we install the "Pipeline Utility Steps" plugin. It provieds a `readJSON` function.

Now we can start writing the Jenkinsfile.

Step 2: Add a stage for incrementing the application version\
The patch version is incremented using the command `npm version patch`. To read the updated version from the package.json file, we use the `readJSON` function provided by the "Pipeline Utility Steps" plugin:
```groovy
#!/usr/bin/env groovy

pipeline {
    agent any
    stages {
        stage('Bump Version') {
            steps {
                script {
                    echo 'incrementing patch version...'
                    dir('app') {
                        sh 'npm version patch'

                        def packageJson = readJSON file: 'package.json'
                        def version = packageJson.version

                        env.IMAGE_VERSION = "$version-$BUILD_NUMBER"
                    }
                }
            }
        }
    }
}
```

Step 3: Add a stage for running the tests\
```groovy
stage('Run Tests') {
    steps {
        script {
            dir('app') {
                sh 'npm install'
                sh 'npm run test'
            } 
        }
    }
}
```

Step 4: Add a stage for building and pushing the Docker image\
```groovy
stage('Build and Push Docker Image') {
    steps {
        withCredentials([usernamePassword(credentialsId: 'DockerHub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
            sh "docker build -t fsiegrist/fesi-repo:devops-bootcamp-node-project-${IMAGE_VERSION} ."
            sh "echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin"
            sh "docker push fsiegrist/fesi-repo:devops-bootcamp-node-project-${IMAGE_VERSION}"
        }
    }
}
```

Step 5: Add a stage for committing the package.json file with the incremented version\
```groovy
stage('Commit Version Update') {
    steps {
        script {
            withCredentials([usernamePassword(credentialsId: 'GitHub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                sh 'git config user.email "jenkins@example.com"'
                sh 'git config user.name "jenkins"'

                sh "git remote set-url origin https://${USERNAME}:${PASSWORD}@github.com/fsiegrist/devops-bootcamp-node-project.git"
                sh 'git add app/package.json'
                sh 'git commit -m "ci: version bump"'
                sh 'git push origin HEAD:main'

                sh 'git config --unset user.email'
                sh 'git config --unset user.name'
            }
        }
    }
}
```

</details>

******

<details>
<summary>Exercise 3: Manually deploy new Docker Image on server</summary>
<br />

**Tasks:**

After the pipeline has run successfully, you:
- Manually deploy the new docker image on the droplet server.

**Steps to solve the tasks:**
Step 1: ssh into a DigitalOcean droplet

Step 2: Execute the following commands:
```sh
docker login
# enter username and password for Docker-Hub

docker run -p3000:3000 -d fsiegrist/fesi-repo:devops-bootcamp-node-project-1.0.2-11
```

Step 3: Open port 3000\
Configure a firewall opening the port 3000 for all IP addresses.

Step 4: Test the application\
Open a browser and enter the URL `http://<droplet-ip>:3000/`.

</details>

******

<details>
<summary>Exercise 4: Extract into Jenkins Shared Library</summary>
<br />

**Tasks:**

A colleague from another project tells you, they are building a similar Jenkins pipeline and they could use some of your logic. So you suggest creating a Jenkins Shared Library to make your Jenkinsfile code reusable and shareable.

Therefore, you do the following:
- Extract all logic into Jenkins-shared-library with parameters and reference it in Jenkinsfile.

**Steps to solve the tasks:**

</details>

******
