# Jenkins Pipeline with Groovy Script

This repository contains a Jenkins pipeline for building a Java application, creating a Docker image, and deploying the application. The pipeline is defined using a `Jenkinsfile` and a Groovy script.

## Overview

The pipeline consists of the following steps:

1. **Build Jar**: Packages the Java application using Maven.
2. **Build Docker Image**: Builds a Docker image of the application and pushes it to Docker Hub.
3. **Deploy Application**: Placeholder for deploying the application.

## Files

### `script.groovy`

The Groovy script contains the logic for building the application, creating the Docker image, and deploying the application.

```// Function to build the application JAR file
def buildJar() {
    // Log a message indicating the start of the build process
    echo "Building the application..."
    
    // Run Maven to package the application into a JAR file
    sh 'mvn package'
} 

// Function to build and push a Docker image
def buildImage() {
    // Log a message indicating the start of the Docker image build process
    echo "Building the Docker image..."
    
    // Use Jenkins credentials to authenticate with Docker Hub
    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
        // Build the Docker image and tag it with a specific name and version
        sh 'docker build -t dockerhub-repo/demo-app:jma-2.0 .'
        
        // Log in to Docker Hub using credentials stored in Jenkins
        sh "echo $PASS | docker login -u $USER --password-stdin"
        
        // Push the Docker image to the specified repository on Docker Hub
        sh 'docker push dockerhub-repo/demo-app:jma-2.0'
    }
} 

// Function to deploy the application (currently empty)
def deployApp() {
    // Log a message indicating the deployment process (implementation can be added later)
    echo 'Deploying the application...'
} 

// Return this script object so it can be used in a Jenkins pipeline
return this
```

- `buildJar()`: Builds the Java application using Maven.
- `buildImage()`: Builds a Docker image and pushes it to Docker Hub using credentials stored in Jenkins.
- `deployApp()`: Placeholder method for deploying the application (can be customized as needed).

### `Jenkinsfile`

The `Jenkinsfile` defines the stages of the Jenkins pipeline and loads the Groovy script for execution.

```// Declare a global variable to hold the loaded Groovy script
def gv

pipeline {
    // Define the agent that will execute the pipeline (can be any available agent)
    agent any
    
    // Define the stages of the pipeline
    stages {
        // Stage 1: Initialization
        stage("init") {
            steps {
                script {
                    // Load an external Groovy script (script.groovy) and assign it to the global variable `gv`
                    gv = load "script.groovy"
                }
            }
        }

        // Stage 2: Build JAR
        stage("build jar") {
            steps {
                script {
                    // Log a message indicating the start of the JAR build process
                    echo "Building JAR"
                    
                    // Uncomment the following line to invoke the `buildJar` function from the loaded script
                    // gv.buildJar()
                }
            }
        }

        // Stage 3: Build Docker Image
        stage("build image") {
            steps {
                script {
                    // Log a message indicating the start of the Docker image build process
                    echo "Building Docker image"
                    
                    // Uncomment the following line to invoke the `buildImage` function from the loaded script
                    // gv.buildImage()
                }
            }
        }

        // Stage 4: Deploy Application
        stage("deploy") {
            steps {
                script {
                    // Log a message indicating the start of the deployment process
                    echo "Deploying application"
                    
                    // Uncomment the following line to invoke the `deployApp` function from the loaded script
                    // gv.deployApp()
                }
            }
        }
    }   
}

```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
```

### Notes:
- The `script.groovy` file should be in the same directory as the `Jenkinsfile` unless you modify the path.
- Make sure the Docker Hub credentials (or any other credentials) are set up correctly in Jenkins for the pipeline to authenticate and push images.
- The `deployApp()` function in the Groovy script is just a placeholder. You may need to add actual deployment logic based on your environment.
