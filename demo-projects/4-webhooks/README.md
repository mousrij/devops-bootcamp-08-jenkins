## Demo Project - Jenkins Webhooks

### Topics of the Demo Project
Configure Webhook to trigger CI Pipeline automatically on every change

### Technologies Used
- Jenkins
- GitLab / GitHub
- Git
- Docker
- Java
- Maven

### Project Description
- Install GitLab / GitHub Plugin in Jenkins
- Configure GitLab access token and connection to Jenkins in GitLab project settings
- Configure GitHub webhook and connection to Jenkins in GitHub project settings
- Configure Jenkins to trigger the CI pipeline, whenever a change is pushed to GitLab / GitHub

#### Steps to install GitLab / GitHub plugin in Jenkins
**GitLab**: Go to "Dashboard" > "Manage Jenkins" > "Manage Plugins" > "Available plugins" and search for "gitlab". Select the "GitLab (Build Triggers)" plugin and press the "Install without restart" button.

**GitHub**: If you installed the recommended plugins during the initial setup of Jenkins, the GitHub plugin is already installed.

#### Steps to configure a GitLab access token and the connection to Jenkins in GitLab
Log in to your GitLab account, open your profile and select "Access Token" on the left. Enter a name of the personal access token (e.g. jenkins), an expiration date, and select the "api" scope, before pressing the "Create personal API token" button. Copy the generated access token.

Leave the profile section, go to your GitLab project and click on "Settings" > "Integrations" > "Jenkins CI". Enable the integration, select "Push" for the trigger, enter the Jenkins URL (`http://<ip-address>:8080`), the Jenkins pipeline name (`devops-bootcamp-pipeline`) as project name, and username / password of your Jenkins account. Press the "Save changes" button.

#### Steps to configure GitHub webhook and connection to Jenkins in GitHub
Log in to your GitHub account, go to your project/repository page and open "Settings" > "Webhooks" and press the "Add webhook" button. Enter the "Payload URL" `http://<jenkins-ip>:8080/github-webhook/` and press the "Add webhook" button.

#### Steps to configure Jenkins to trigger the CI pipeline, whenever a change is pushed
**GitLab**: Go back to Jenkins and open "Dashboard" > "Manage Jenkins" > "Configure System", where you will find a "GitLab" section. Make sure "Enable authentication for `/project` endpoint" is checked, enter a connection name (e.g. gitlab-conn) and the GitLab host URL (`https://gitlab.com`). To add the API token for GitLab access, click on "Add" > "Jenkins". Select the kind "GitLab API token" and paste the copied access token into the "Api token" form field. Enter an ID (e.g. GitLab API token) and press the "Add" button. Now you can select the token from the credentials dropdown and press the "Save" button.

When you open the configuration of a build pipeline, you'll find a GitLab connection configured as well as automatically enabled GitLab build triggers.

**GitHub**: In Jenkins open your pipeline project (`devops-bootcamp-pipeline`), open the configuration and scroll down to the "Build Triggers" section. Check the "GitHub hook trigger for GITScm polling" checkbox and press the "Save" button.

#### Additional Configurations for Multibranch Pipelines
To enable automatic triggering of builds for multibranch pipeline projects, some additional steps are required. Go to "Dashboard" > "Manage Jenkins" > "Manage Plugins" > "Available plugins" and search for "multibranch scan". Select the "Multibranch Scan Webhook Trigger" plugin and press the "Install without restart" button.

Now open the configuration of your multibranch pipeline project and scroll down to the "Scan Multibranch Pipeline Triggers" section. There you'll find an additional checkbox "Scan by webhook", that was added by the plugin. Select it and enter a trigger token. This can be any name (e.g. `gitlabtoken` / `githubtoken`). Click on the question mark on the right border belonging to the trigger token and copy the webhook URL description (starting with JENKINS_URL). Save the configuration.

**GitLab**: Next go back to your GitLab account and open "Settings" > "Webhooks". Paste the copied URL to the "URL" field and replace `JENKINS_URL` and `[Trigger token]` with the correct values (`http://<jenkins-ip>:8080/multibranch-webhook-trigger/invoke?token=gitlabtoken`). Select the "Push events" trigger and press the "Add webhook" button.

**GitHub**: On GitHub go to your project/repository page and open "Settings" > "Webhooks" and press the "Add webhook" button. Enter the "Payload URL" `http://<jenkins-ip>:8080/multibranch-webhook-trigger/invoke?token=githubtoken` and press the "Add webhook" button.
