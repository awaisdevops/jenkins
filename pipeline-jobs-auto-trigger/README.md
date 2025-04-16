# Jenkins and GitLab Integration for Automated Pipeline Triggering

This guide explains how to set up automatic triggering of a Jenkins pipeline job using the GitLab Plugin in Jenkins and GitLab's built-in CI/CD integration.

## Step 1: Install Required Jenkins Plugins

In Jenkins:

1. Go to Manage Jenkins > Plugins
2. Install the following plugins if they are not already installed:
   - Git plugin
   - GitLab plugin
   - GitLab Hook plugin
   - Pipeline plugin (for scripted or declarative Jenkinsfiles)
3. Restart Jenkins if prompted.

## Step 2: Add GitLab Credentials to Jenkins

1. Go to Manage Jenkins > Credentials
2. Under the appropriate domain (usually "Global"), click Add Credentials
3. Configure the credentials:
   - Kind: Secret text
   - Secret: Paste your GitLab Personal Access Token
     - Token must have the following scopes: api, read_user, read_repository, write_repository
   - ID: gitlab-token (or any ID you prefer)

## Step 3: Configure GitLab Connection in Jenkins

1. Go to Manage Jenkins > Configure System
2. Scroll to the GitLab section
3. Click Add GitLab Connection and set:
   - Name: GitLab
   - GitLab Host URL: https://gitlab.com (or your GitLab server URL)
   - Credentials: Select the token you added earlier
4. Click Test Connection to confirm setup

## Step 4: Configure Jenkins Pipeline Job

1. Go to Jenkins > New Item > Pipeline
2. Enter a job name and click OK

In the job configuration:

- Under General:
  - Check GitLab project
  - Select the GitLab connection
  - Set the project URL or use autocomplete

- Under Build Triggers:
  - Check Build when a change is pushed to GitLab
  - Enable Push events, Merge request events, or others as needed

- Under Pipeline:
  - Definition: Pipeline script from SCM
  - SCM: Git
  - Repository URL: GitLab repository HTTPS URL
  - Credentials: GitLab credentials if the repo is private
  - Branch: */main or your target branch
  - Script Path: Jenkinsfile or another path if custom

## Step 5: Add Jenkins Integration in GitLab

1. Go to your GitLab repository > Settings > Integrations (or Webhooks)
2. In the Jenkins CI section:
   - Set the Jenkins URL (e.g., http://jenkins.yourdomain.com/)
   - Set the Jenkins job name as the project path
   - Enable the events you want to trigger the job (e.g., push, merge request)
   - Add the authentication token if you configured one
3. Click Add Integration
4. Use the Test button to confirm the webhook works (should return HTTP 200 OK)

## Step 6: Test the Setup

1. Push a commit to your GitLab repository
2. Jenkins should receive the webhook and start the pipeline job
3. Check Jenkins build logs to verify it triggered correctly

## License

This project is licensed under the MIT License.
