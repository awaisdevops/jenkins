# Jenkins Maven Auto Versioning and Deployment

This repository explains how to configure a Jenkins pipeline to automatically increment the Maven application version, build a Docker image, deploy to EC2, and commit the version change back to the Git repository.

## Increment Application Version Locally

How versioning happens automatically? Every build tool or package manager has its own plugin for automatic version increment. When used, it reads the code and finds the current version of the application and bumps up the version as directed by the developer using the plugin.

## Commit Version Bump from Jenkins

Jenkins successfully runs a build and increments the application version in the `pom.xml` file locally. However, after a developer commits code and the Jenkins job is triggered again, the version increment does not persist. This happens because Jenkins updates the version only locally and does not push it to the repository.

To fix this, update the Jenkins pipeline to commit and push the version bump back to the Git repository. This ensures that future pipeline runs use the latest version as a base.

When a developer commits code to the repo, the build is triggered. The version is incremented but remains local on Jenkins. We need to push the updated `pom.xml` to the remote repo so that version increment is stored. This ensures the next pipeline trigger will increment from the latest version.

## Ignore Jenkins Commit for Jenkins Pipeline Trigger

Now we have two types of commits that can trigger a build:
- A developer’s commit
- Jenkins’ commit to push the version bump

This can create an infinite loop of triggers.

### How to prevent the loop:

Install and configure the "Ignore committer strategy" plugin in Jenkins.

Steps:
1. Go to Jenkins Plugin Manager
2. Search and install "Ignore committer strategy"
3. Go to job configuration → Branch Sources → Build Strategies
4. Add "Ignore committer strategy"
5. Enter the Jenkins user email in "ignored author emails"
6. (Optional) Enable "Allow builds for non-ignored authors"
7. Save and run

## Jenkinsfile

```groovy
#!/usr/bin/env groovy

# Jenkins Declarative Pipeline Definition
pipeline {
    agent any  # Run on any available agent

    tools {
        maven 'Maven'  # Use the configured Maven installation
    }

    stages {

        # Stage: Increment Version
        stage('increment version') {
            steps {
                script {
                    echo 'incrementing app version...'
                    
                    # Update project version using Maven Build Helper and Versions plugin
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    
                    # Extract version from pom.xml
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    
                    # Set version for Docker image
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }

        # Stage: Build Application
        stage('build app') {
            steps {
                script {
                    echo "building the application..."
                    
                    # Build the Maven project
                    sh 'mvn clean package'
                }
            }
        }

        # Stage: Build Docker Image
        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."
                    
                    # Login and push Docker image using credentials
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "docker build -t dockerhub-url/demo-app:${IMAGE_NAME} ."
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push dockerhub-url/demo-app:${IMAGE_NAME}"
                    }
                }
            }
        }

        # Stage: Deploy to EC2
        stage('deploy') {
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                    # Add deployment logic here
                }
            }
        }

        # Stage: Commit Version Update to Git
        stage('commit version update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'gitlab-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        
                        # Git configuration
                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "jenkins"'
                        
                        # Push version bump to remote Git repository
                        sh "git remote set-url origin https://${USER}:${PASS}@gitlab.com/nanuchi/java-maven-app.git"
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push origin HEAD:jenkins-jobs'
                    }
                }
            }
        }
    }
}
```

## License

MIT License
