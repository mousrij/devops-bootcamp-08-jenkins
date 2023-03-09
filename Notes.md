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



</details>

*****