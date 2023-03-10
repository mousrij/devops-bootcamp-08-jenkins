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
cd ..
git init 
git add .
git commit -m "Initial commit"

# create git repository on GitHub push your newly created local repository to it
git remote add origin git@github.com:fsiegrist/devops-bootcamp-08-jenkins.git
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

**Steps to solve the tasks:**

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

</details>

******

<details>
<summary>Exercise 3: Manually deploy new Docker Image on server</summary>
<br />

**Tasks:**

After the pipeline has run successfully, you:
- Manually deploy the new docker image on the droplet server.

**Steps to solve the tasks:**

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
