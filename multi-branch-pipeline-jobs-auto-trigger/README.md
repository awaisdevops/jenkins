# Jenkins Multibranch Pipeline Trigger via GitLab Webhook

This guide explains how to automatically trigger a Jenkins Multibranch Pipeline job from GitLab using webhooks.

---

## 1. Jenkins Server Configuration

### a. Install Plugin

1. Go to **Jenkins → Manage Jenkins → Plugin Manager → Available**.
2. Search for: **Multibranch Scan Webhook Trigger Plugin**.
3. Install it (no restart required).

This plugin enables Jenkins to automatically scan the multibranch pipeline project when GitLab sends a webhook notification.

---

## 2. Add a Token to Your Multibranch Pipeline Job

1. Navigate to your **Multibranch Pipeline job** in Jenkins.
2. Click **"Configure"**.
3. Scroll to the **"Scan Multibranch Pipeline Triggers"** section.
4. Check **"Scan by webhook"**.
5. In the **"Token"** field, enter a secure token (e.g., `my-token`).
6. Click **Save**.

This token will be used to secure the webhook URL.

---

## 3. GitLab Configuration

### a. Get Jenkins Webhook URL

Format:

```
http://<jenkins-url>/multibranch-webhook-trigger/invoke?job=<JOB_NAME>
```

- Replace `<jenkins-url>` with your actual Jenkins server address.
- Replace `<JOB_NAME>` with the exact name of your Jenkins job.
- If your job is inside a folder, use `%2F` for slashes in the path.

Example:

```
http://jenkins.example.com/multibranch-webhook-trigger/invoke?job=my-folder%2Fmy-multibranch-project&token=my-token
```

### b. Add Webhook in GitLab

1. Go to your **GitLab repository**.
2. Navigate to **Settings → Webhooks**.
3. Paste the Jenkins webhook URL (including the `token` parameter).
4. Select the following **Trigger Events**:
   - Push events
   - Merge request events
   - (Optional: Tag push events or others if needed)
5. Enable **SSL verification** if you're using HTTPS.
6. Click **Add Webhook**.

Use **Test → Push Events** in GitLab to verify webhook delivery.

---

## 4. Jenkins Multibranch Job Behavior

When the webhook is triggered:

- Jenkins will **scan the Git repository**.
- Detect **new or updated branches**.
- Automatically run pipeline jobs based on your `Jenkinsfile`.

---

## 5. Troubleshooting

- **Check Jenkins Logs**:  
  Go to `Manage Jenkins → System Log` for webhook and scan events.

- **GitLab Webhook Logs**:  
  GitLab displays webhook delivery status and HTTP responses in the webhook settings page.

- **Folder Paths**:  
  Jenkins job names are case-sensitive and folder names must be **URL-encoded** using `%2F`.

---

## License

MIT License
