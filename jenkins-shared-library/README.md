# Jenkins Shared Library

## Overview

A **Jenkins Shared Library** is a powerful feature that allows developers to share common code across multiple Jenkins jobs and pipelines, promoting code reuse and reducing duplication. This mechanism is particularly useful in large projects where similar build steps or functions are required in various jobs. 

The Jenkins Shared Library is written in **Groovy** language.

## Directory Structure

A typical shared library has the following structure:

- **`vars/`** folder: Contains Groovy scripts/functions that can be called directly in Jenkins pipelines. 
  - This is the main folder in a Jenkins shared library. It includes all the functions that will be called from the Jenkinsfile like application version increment function, build jar file function, build docker image function, push docker image function, etc.
  
- **`src/`** folder: Contains Groovy source files for reusable classes.
  - This folder stores any helper code like utility functions written in Groovy.

- **`resources/`** folder: Holds static files needed by the library.
  - This includes external libraries, non-Groovy files, bash scripts, PowerShell scripts, shell scripts, JSON files, etc.

## Create Jenkins Shared Library Project

To create a Jenkins shared library using the above structure, follow these steps:

1. **Create a Repository**: To centrally manage your Jenkins shared library, create a GitLab repository.
2. **Create Groovy Scripts**: Write the necessary Groovy functions or classes you want to share across Jenkins pipelines.
3. **Store in Git Repository**: Push your scripts to a Git repository while maintaining the correct directory structure.
4. **Configure in Jenkins**: 
   - Navigate to **Manage Jenkins > Configure System**.
   - In the **Global Pipeline Libraries** section, add a new library by specifying its name, retrieval method (e.g., Git), and default version (branch/tag).
5. **Use in Pipelines**: In your Jenkinsfile, load the shared library using the `@Library` annotation:

    ```groovy
    @Library('my-shared-library') _
    ```

## Create a New Remote Repo for Jenkins Shared Library

To create a new remote repository for Jenkins Shared Library:

1. Create a new Git repository for your shared library.
2. Store the `jenkins-shared-library.yaml` configuration to define your Jenkins environment.

### `jenkins-shared-library.yaml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<module type="JAVA_MODULE" version="4">
  <component name="NewModuleRootManager" inherit-compiler-output="true">
    <exclude-output />
    <content url="file://$MODULE_DIR$">
      <sourceFolder url="file://$MODULE_DIR$/src" isTestSource="false" />
    </content>
    <orderEntry type="inheritedJdk" />
    <orderEntry type="sourceFolder" forTests="false" />
  </component>
</module>
```

## Example Groovy Scripts

Below are the key Groovy functions used in the `vars/` folder of your shared library.

### `vars/buildImage.groovy`

```groovy
#!/usr/bin/env groovy

# Import the Docker class to be able to call Docker-related functions
import com.example.Docker

# Define the main function which accepts an image name as an argument
def call(String imageName) {
    # Create an instance of the Docker class and call the 'buildDockerImage' function
    return new Docker(this).buildDockerImage(imageName)
}
```
This script builds a Docker image by calling the `buildDockerImage` method from the `Docker` class.

### `vars/buildJar.groovy`

```groovy
#!/usr/bin/env groovy

# Define the main function for building a JAR file
def call() {
    # Print which branch is being built
    echo "Building the application for branch $BRANCH_NAME"
    # Execute the Maven build command to package the application
    sh 'mvn package'
}
```
This script builds the application using Maven (`mvn package`) for the specified branch.

### `vars/dockerLogin.groovy`

```groovy
#!/usr/bin/env groovy

# Import the Docker class to be able to call Docker-related functions
import com.example.Docker

# Define the main function for Docker login
def call() {
    # Create an instance of the Docker class and call the 'dockerLogin' function
    return new Docker(this).dockerLogin()
}
```
This script logs into Docker using the credentials stored in Jenkins.

### `vars/dockerPush.groovy`

```groovy
#!/usr/bin/env groovy

# Import the Docker class to be able to call Docker-related functions
import com.example.Docker

# Define the main function which accepts the image name as an argument
def call(String imageName) {
    # Create an instance of the Docker class and call the 'dockerPush' function
    return new Docker(this).dockerPush(imageName)
}
```
This script pushes the Docker image to a Docker registry.

### `src/com/example/Docker.groovy`

```groovy
#!/usr/bin/env groovy

# Define the package where the class is located
package com.example

# Define the Docker class implementing the Serializable interface
class Docker implements Serializable {

    # Declare the 'script' variable to hold the pipeline script
    def script

    # Constructor to initialize the 'script' variable with the pipeline context
    Docker(script) {
        this.script = script
    }

    # Method to build a Docker image
    def buildDockerImage(String imageName) {
        # Print a message to indicate the Docker image is being built
        script.echo "Building the Docker image..."
        # Execute the shell command to build the Docker image
        script.sh "docker build -t $imageName ."
    }

    # Method to log in to Docker
    def dockerLogin() {
        # Use Jenkins credentials for Docker login
        script.withCredentials([script.usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
            # Execute the shell command to login using the credentials
            script.sh "echo $script.PASS | docker login -u $script.USER --password-stdin"
        }
    }

    # Method to push a Docker image to the registry
    def dockerPush(String imageName) {
        # Execute the shell command to push the Docker image
        script.sh "docker push $imageName"
    }
}
```
This class contains the necessary methods for interacting with Docker:
- `buildDockerImage`: Builds the Docker image using the provided name.
- `dockerLogin`: Logs into Docker using stored credentials.
- `dockerPush`: Pushes the Docker image to the Docker registry.

## Make Shared Library Globally Available

To make a Jenkins shared library globally available for use in Jenkinsfiles:

1. Log into Jenkins and navigate to **Manage Jenkins > Configure System**.
2. Scroll down to the **Global Pipeline Libraries** section and click **Add**.
3. Enter the library name and set the retrieval method to **Git**.
4. Enter the repository URL and add any necessary credentials if the repository is private.
5. (Optional) Check **Load implicitly** to make the library available without explicitly loading it in the Jenkinsfile.

Once configured, the library can be loaded using the `@Library` annotation in Jenkinsfiles:

```groovy
@Library('your-library-name') _
```

> **Important**: Be careful with the versioning of your shared library. It is best to pin it to a specific branch, commit hash, or tag to ensure stability. Avoid using the `master` branch in production.

## Jenkinsfile

Below is our jenkinsfile using shared library in it for various stages:

```groovy
#!/usr/bin/env groovy

# Load the shared library from the repository
@Library ('jenkins-shared-library')  # Load the 'jenkins-shared-library' into this Jenkins pipeline

# Define a global variable 'gv' that will be used later
def gv

# Define the pipeline block
pipeline {
    agent any  # Use any available agent for this pipeline
    tools {
        maven 'maven'  # Define the Maven tool for use in the pipeline
    }
    environment {
        IMAGE_NAME = 'awaisakram11199/demo-app:jma444'  # Define a custom environment variable for the Docker image name
    }

    # Define the stages of the pipeline
    stages {

        # Stage 1: Initialization of the script
        stage('Init Script') {
            steps {
                script {
                    # Load the external script 'script.groovy' into the 'gv' variable
                    gv = load "script.groovy"
                }
            }
        }

        # Stage 2: Build the application
        stage('Build App') {
            steps {
                script {
                    # Call the 'buildJar' function from the shared library to build the app
                    buildJar()
                }
            }
        }

        # Stage 3: Build Docker image and push it to Docker registry
        stage('Build and Push Image') {
            steps {
                script {
                    # Call the 'buildImage' function to build the Docker image
                    buildImage (env.IMAGE_NAME)
                    # Call the 'dockerLogin' function to log into Docker registry
                    dockerLogin()
                    # Call the 'dockerPush' function to push the image to the Docker registry
                    dockerPush (env.IMAGE_NAME)
                }
            }
        }

        # Stage 4: Deploy the image
        stage('Deploy') {
            steps {
                script {
                    # Call the 'deployImage' method from the loaded script (gv) to deploy the image
                    gv.deployImage()
                }
            }
        }
    }

    # Define actions to take after the pipeline has completed
    post {
        success {
            # Print a success message if the deployment was successful
            echo 'Deployment completed successfully!'
        }
        failure {
            # Print a failure message if the deployment failed
            echo 'Deployment failed.'
        }
    }
}


## Benefits of Using Shared Libraries

- **Efficiency**: Reduces redundancy by allowing code reuse across different projects.
- **Maintainability**: Simplifies updates. Changing code in one place updates all dependent jobs.
- **Organization**: Helps organize complex build processes into manageable components.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
