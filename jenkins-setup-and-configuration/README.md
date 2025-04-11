# Jenkins Installation and Configuration Guide

This guide outlines how to install and configure Jenkins as a Docker container, configure necessary tools like Maven, Node.js, and Git, and set up various credentials and Docker integration for Jenkins.

## Table of Contents
1. [Install Jenkins as a Docker Container](#install-jenkins-as-a-docker-container)
2. [Install Tools in Jenkins Container](#install-tools-in-jenkins-container)
   - [Install Maven](#install-maven)
   - [Install Node.js and npm](#install-nodejs-and-npm)
3. [Configure Git in Jenkins](#configure-git-in-jenkins)
4. [Configure GitLab, DockerHub, and Nexus Credentials in Jenkins](#configure-gitlab-dockerhub-and-nexus-credentials-in-jenkins)
5. [Make Docker Available in Jenkins](#make-docker-available-in-jenkins)
6. [Configure Insecure Nexus Repositories](#configure-insecure-nexus-repositories)
7. [License](#license)

---

## 1. Install Jenkins as a Docker Container

To install Jenkins, follow these steps:

1. **Pull the Jenkins image**:
   ```bash
   docker pull jenkins/jenkins:lts
   ```

2. **Run Jenkins in a Docker container**:
   ```bash
   docker run -p 8080:8080 -p 50000:50000 -d \
   -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
   ```
   - This runs Jenkins on port `8080`. Port `50000` is for Jenkins master and worker/slave nodes.
   - A named volume `jenkins_home` is mapped to persist Jenkins data and configurations.

3. **Inspect Jenkins volume**:
   ```bash
   docker volume inspect jenkins_home
   ```

4. Access Jenkins by visiting `http://localhost:8080` and proceed with the configuration.

---

## 2. Install Tools in Jenkins Container

To install the necessary tools like Maven and Node.js inside the Jenkins Docker container:

### a. Install Maven

1. Update the package index:
   ```bash
   sudo apt update
   ```

2. Install Java (required for Maven):
   ```bash
   sudo apt install openjdk-11-jdk -y
   ```

3. Navigate to the `/opt` directory:
   ```bash
   cd /opt
   ```

4. Download Maven:
   ```bash
   sudo wget https://dlcdn.apache.org/maven/maven-3/3.9.6/binaries/apache-maven-3.9.6-bin.tar.gz
   ```

5. Extract the Maven archive:
   ```bash
   sudo tar -xvzf apache-maven-3.9.6-bin.tar.gz
   ```

6. Create a symbolic link for Maven:
   ```bash
   sudo ln -s apache-maven-3.9.6 maven
   ```

7. Create Maven environment variables:
   ```bash
   sudo nano /etc/profile.d/maven.sh
   ```

   Add the following lines:
   ```bash
   export M2_HOME=/opt/maven
   export MAVEN_HOME=/opt/maven
   export PATH=${M2_HOME}/bin:${PATH}
   ```

8. Make the script executable:
   ```bash
   sudo chmod +x /etc/profile.d/maven.sh
   ```

9. Apply the environment variables:
   ```bash
   source /etc/profile.d/maven.sh
   ```

10. Verify Maven installation:
    ```bash
    mvn -v
    ```

### b. Install Node.js and npm

1. Update the package index:
   ```bash
   sudo apt update
   ```

2. Install curl (if not already installed):
   ```bash
   sudo apt install curl -y
   ```

3. Add the Node.js 18.x repository:
   ```bash
   curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
   ```

4. Install Node.js and npm:
   ```bash
   sudo apt install nodejs -y
   ```

5. Verify Node.js and npm installation:
   ```bash
   node -v
   npm -v
   ```

---

## 3. Configure Git in Jenkins

### Steps to Configure Git in Jenkins Pipeline:

1. Go to **Jenkins Dashboard**.
2. Click **"New Item"**.
3. Enter a name for your job (e.g., `MyGitPipeline`).
4. Select **"Pipeline"** → Click **OK**.
5. Scroll down to the **Pipeline** section.
6. Under **Definition**, select `Pipeline script from SCM`.
7. In the **SCM** dropdown, choose `Git`.
8. In **Repository URL**, enter the URL of your repository:
   - HTTPS: `https://gitlab.com/your-username/your-repo.git`
   - OR SSH: `git@gitlab.com:your-username/your-repo.git`
9. Click **"Add"** next to **Credentials**:
   - Choose **Username/Password** for HTTPS
   - OR **SSH Username with Private Key** for SSH
10. Select the credential from the dropdown.
11. In **Branches to build**, use `*/main` or `*/master` (or any other branch you want).
12. Set **Script Path** (default is `Jenkinsfile`, change if needed).
13. Click **Save**.
14. Click **"Build Now"** to test the pipeline.

---

## 4. Configure GitLab, DockerHub, and Nexus Credentials in Jenkins

### Add GitLab, DockerHub & Nexus Credentials in Jenkins

1. Go to **Jenkins Dashboard**.
2. Click **Manage Jenkins**.
3. Click **Credentials**.
4. Choose a **domain** (usually `(global)`).
5. Click on **"Add Credentials"** (left sidebar).

### A. Add GitLab Credentials

- **Kind**: `Username with password` (for HTTPS) OR `SSH Username with private key` (for SSH access)
- **Username**: Your GitLab username
- **Password**: GitLab personal access token (for HTTPS)
- **ID** (optional): e.g., `gitlab-creds`
- **Description**: e.g., `GitLab Access`
- Click **OK**.

### B. Add DockerHub Credentials

- **Kind**: `Username with password`
- **Username**: Your DockerHub username
- **Password**: Your DockerHub password or access token
- **ID** (optional): e.g., `dockerhub-creds`
- **Description**: e.g., `DockerHub Login`
- Click **OK**.

### C. Add Nexus Credentials

- **Kind**: `Username with password`
- **Username**: Nexus repository username
- **Password**: Nexus password or token
- **ID** (optional): e.g., `nexus-creds`
- **Description**: e.g., `Nexus Repo Access`
- Click **OK**.

### Using Credentials in Jenkinsfile

You can refer to the credentials in your pipeline like this:

```groovy
withCredentials([usernamePassword(credentialsId: 'gitlab-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
    sh 'git clone https://${GIT_USER}:${GIT_TOKEN}@gitlab.com/your/repo.git'
}

withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
}

withCredentials([usernamePassword(credentialsId: 'nexus-creds', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
    sh 'curl -u $NEXUS_USER:$NEXUS_PASS https://your-nexus-url/repository/...'
}
```

---

## 5. Make Docker Available in Jenkins

To make Docker commands available in Jenkins inside the container, you need to mount Docker binaries and runtime. Follow these steps:

1. Stop or remove the existing Jenkins container (if running).
2. Run a new Jenkins container with Docker integration:
   ```bash
   docker run -d -p 8080:8080 -p 50000:50000 \
   -v jenkins_home:/var/jenkins_home \
   -v /var/run/docker.sock:/var/run/docker.sock \
   -v $(which docker):/usr/bin/docker jenkins/jenkins:lts
   ```

3. Give the appropriate permissions to the Docker runtime file inside the Jenkins container:
   ```bash
   docker exec -it -u 0 <container_id> bash
   ls -l /var/run/docker.sock
   chmod -R 666 /var/run/docker.sock
   ```

---

## 6. Configure Insecure Nexus Repositories

If your Nexus server doesn't have SSL configured, you can allow Docker to communicate with insecure registries by modifying the Docker daemon settings:

1. Edit the Docker daemon configuration:
   ```bash
   sudo vim /etc/docker/daemon.json
   ```

2. Add the following:
   ```json
   {
     "insecure-registries": ["staging5.rolustech.com:10482"]
   }
   ```

3. Restart Docker:
   ```bash
   sudo systemctl restart docker
   ```

4. Reapply permissions to the Docker runtime file:
   ```bash
   chmod -R 666 /var/run/docker.sock
   ```

---

## 7. License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
