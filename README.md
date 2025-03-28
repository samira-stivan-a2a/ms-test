# MS Teams Notification Action

[![GitHub release](https://img.shields.io/badge/release-v1.1.0-blue)](https://github.com/haroldelopez/ms-teams-notification-action/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Send customizable notifications to Microsoft Teams channels for various events including pull requests and deployments.

## What's New in v1.1.0

- Support for multiple notification types: PR, deployment start, deployment completion, and custom notifications
- Dynamic color coding based on status (success, failure, in_progress)
- Enhanced customization options with additional facts
- Improved error handling and validation

## 📋 Features

- Notify on pull request events (created, updated, merged)
- Notify when deployments start, succeed, or fail
- Custom notifications with flexible formatting
- Clickable action buttons linking to GitHub or custom URLs
- Color-coded messages based on event status
- Add extra data fields to your notifications

## 🚀 Getting Started

### Prerequisites

- A Microsoft Teams webhook URL (see [Microsoft's documentation](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook) on how to create one)
- A GitHub repository with Actions enabled

### Installation

Add the action to your GitHub workflow file:

```yaml
steps:
  - name: Send MS Teams Notification
    uses: haroldelopez/ms-teams-notification-action@v1.1.0
    with:
      webhook_url: ${{ secrets.MS_TEAMS_WEBHOOK_URL }}
      notification_type: 'pr'
      title: 'New Pull Request'
      message: 'A new pull request has been created'
      url: ${{ github.event.pull_request.html_url }}
      creator: ${{ github.event.pull_request.user.login }}
      repo_name: ${{ github.repository }}
```

## ⚙️ Input Parameters

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `webhook_url` | Yes | - | Microsoft Teams webhook URL |
| `notification_type` | Yes | `custom` | Type of notification (`pr`, `deployment_started`, `deployment_finished`, `custom`) |
| `title` | Yes | - | Notification title |
| `message` | Yes | - | Main notification message |
| `url` | No | - | URL to link in the notification |
| `creator` | No | - | Person who triggered the event |
| `repo_name` | No | - | Repository name |
| `environment` | No | - | Deployment environment (dev, staging, production, etc.) |
| `status` | No | `success` | Status of the event (`success`, `failure`, `in_progress`) |
| `theme_color` | No | `0076D7` | Theme color for the Teams message card (hex code without #) |
| `activity_image` | No | GitHub logo | Custom activity image URL |
| `additional_facts` | No | `[]` | JSON string of additional facts to include in the notification |

## 🔍 Usage Examples

### Pull Request Notification

```yaml
- name: Notify Teams about Pull Request
  uses: haroldelopez/ms-teams-notification-action@v1.1.0
  with:
    webhook_url: ${{ secrets.MS_TEAMS_WEBHOOK_URL }}
    notification_type: 'pr'
    title: ${{ github.event.pull_request.title }}
    message: 'A new pull request has been opened in the repository.'
    url: ${{ github.event.pull_request.html_url }}
    creator: ${{ github.event.pull_request.user.login }}
    repo_name: ${{ github.repository }}
```

### Deployment Started Notification

```yaml
- name: Notify Teams about Deployment Start
  uses: haroldelopez/ms-teams-notification-action@v1.1.0
  with:
    webhook_url: ${{ secrets.MS_TEAMS_WEBHOOK_URL }}
    notification_type: 'deployment_started'
    title: 'Deployment Started'
    message: 'A deployment to ${{ env.ENVIRONMENT }} has been initiated.'
    url: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
    creator: ${{ github.actor }}
    repo_name: ${{ github.repository }}
    environment: ${{ env.ENVIRONMENT }}
    status: 'in_progress'
    theme_color: 'ffcc00'  # Yellow for in progress
```

### Deployment Finished Notification

```yaml
- name: Notify Teams about Deployment Completion
  uses: haroldelopez/ms-teams-notification-action@v1.1.0
  with:
    webhook_url: ${{ secrets.MS_TEAMS_WEBHOOK_URL }}
    notification_type: 'deployment_finished'
    title: 'Deployment Completed'
    message: 'The deployment to ${{ env.ENVIRONMENT }} has been completed successfully.'
    url: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
    creator: ${{ github.actor }}
    repo_name: ${{ github.repository }}
    environment: ${{ env.ENVIRONMENT }}
    status: 'success'
    theme_color: '00cc00'  # Green for success
    additional_facts: '[{"name":"Deployment Time","value":"${{ steps.deploy_time.outputs.duration }} minutes"}]'
```

### Deployment Failure Notification

```yaml
- name: Notify Teams about Deployment Failure
  if: failure()
  uses: haroldelopez/ms-teams-notification-action@v1.1.0
  with:
    webhook_url: ${{ secrets.MS_TEAMS_WEBHOOK_URL }}
    notification_type: 'deployment_finished'
    title: 'Deployment Failed'
    message: 'The deployment to ${{ env.ENVIRONMENT }} has failed.'
    url: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
    creator: ${{ github.actor }}
    repo_name: ${{ github.repository }}
    environment: ${{ env.ENVIRONMENT }}
    status: 'failure'
    theme_color: 'ff0000'  # Red for failure
```

### Custom Notification

```yaml
- name: Send Custom Notification
  uses: haroldelopez/ms-teams-notification-action@v1.1.0
  with:
    webhook_url: ${{ secrets.MS_TEAMS_WEBHOOK_URL }}
    notification_type: 'custom'
    title: 'Weekly Build Summary'
    message: 'The weekly build process has completed with 42 successful tests.'
    url: ${{ github.server_url }}/${{ github.repository }}/actions
    repo_name: ${{ github.repository }}
    additional_facts: '[{"name":"Total Tests","value":"42"},{"name":"Code Coverage","value":"87%"},{"name":"Build Time","value":"12 minutes"}]'
```

## 🎨 Customizing Notifications

### Using Additional Facts

The `additional_facts` parameter allows you to add extra information to your notification. It accepts a JSON array of objects with `name` and `value` properties:

```yaml
additional_facts: '[{"name":"Total Tests","value":"42"},{"name":"Code Coverage","value":"87%"}]'
```

### Custom Colors

You can use the `theme_color` parameter to set a custom color for your notification. The value should be a hex color code without the `#` symbol:

- Green (success): `00cc00`
- Red (failure): `ff0000`
- Yellow (in progress): `ffcc00`
- Blue (default): `0076D7`

### Custom Images

Use the `activity_image` parameter to change the icon displayed with your notification:

```yaml
activity_image: 'https://example.com/path/to/your/image.png'
```

## 🔧 Complete Workflow Example

Here's a complete workflow that sends notifications when a deployment starts and finishes:

```yaml
name: Deploy and Notify

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set environment variable
        run: echo "ENVIRONMENT=production" >> $GITHUB_ENV
      
      - name: Start Deployment Notification
        uses: haroldelopez/ms-teams-notification-action@v1.1.0
        with:
          webhook_url: ${{ secrets.MS_TEAMS_WEBHOOK_URL }}
          notification_type: 'deployment_started'
          title: 'Deployment Started'
          message: 'Deploying latest changes to production'
          url: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
          creator: ${{ github.actor }}
          repo_name: ${{ github.repository }}
          environment: ${{ env.ENVIRONMENT }}
          status: 'in_progress'
      
      - name: Deploy
        id: deploy
        run: |
          # Your deployment commands here
          echo "::set-output name=duration::3.5"
      
      - name: Successful Deployment Notification
        if: success()
        uses: haroldelopez/ms-teams-notification-action@v1.1.0
        with:
          webhook_url: ${{ secrets.MS_TEAMS_WEBHOOK_URL }}
          notification_type: 'deployment_finished'
          title: 'Deployment Successful'
          message: 'The deployment to production has completed successfully.'
          url: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
          creator: ${{ github.actor }}
          repo_name: ${{ github.repository }}
          environment: ${{ env.ENVIRONMENT }}
          status: 'success'
          additional_facts: '[{"name":"Deployment Time","value":"${{ steps.deploy.outputs.duration }} minutes"}]'
      
      - name: Failed Deployment Notification
        if: failure()
        uses: haroldelopez/ms-teams-notification-action@v1.1.0
        with:
          webhook_url: ${{ secrets.MS_TEAMS_WEBHOOK_URL }}
          notification_type: 'deployment_finished'
          title: 'Deployment Failed'
          message: 'The deployment to production has failed.'
          url: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
          creator: ${{ github.actor }}
          repo_name: ${{ github.repository }}
          environment: ${{ env.ENVIRONMENT }}
          status: 'failure'
```

## 📝 Notes and Troubleshooting

- **Webhook URL**: Always store your Teams webhook URL as a GitHub secret.
- **Message Size**: Microsoft Teams has a limit for message card size. If your notification is too large, it might not be delivered.
- **Escaping**: The action automatically escapes special characters in your inputs to prevent JSON formatting issues.
- **Error Handling**: If the notification fails to send, the action will fail with an error message.

## 📄 License

MIT License - see the LICENSE file for details.

## 👥 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request