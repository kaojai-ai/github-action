# Slack notification GitHub Action

A reusable composite GitHub Action that formats and sends workflow notifications to Slack using the official [`slackapi/slack-github-action`](https://github.com/slackapi/slack-github-action).

## Inputs

| Name | Required | Default | Description |
| ---- | -------- | ------- | ----------- |
| `slack_webhook_url` | ✅ | – | Incoming Slack webhook URL. |
| `provider_name` | | `slack` | Provider label that appears in the message fields. |
| `channel` | | `package` | Slack channel or target to deliver the message to. |
| `prefix` | | `⚙️ Workflow` | Title prefix rendered in the header block. |
| `bot_name` | | `kj-ops-bot` | Slack bot display name. |
| `bot_icon_url` | | `https://avatars.githubusercontent.com/u/227843191` | Avatar URL for the bot. |
| `message` | | _(auto-generated)_ | Optional Markdown message body. |
| `status` | | _(detected from `JOB_STATUS`)_ | Optional status override (`success`, `failure`, `cancelled`, ...). |

> [!TIP]
> When calling this action from a workflow, pass the job status through the `JOB_STATUS` environment variable so that the message colour and emoji reflect the result:
>
> ```yaml
> - name: Slack notification
>   if: ${{ always() && secrets.SLACK_WEBHOOK != '' }}
>   uses: ./.github/actions/slack-notification
>   env:
>     JOB_STATUS: ${{ job.status }}
>   with:
>     slack_webhook_url: ${{ secrets.SLACK_WEBHOOK }}
> ```

The action automatically enriches the Slack message with repository, branch, workflow, actor and run link metadata. It also generates a sensible default message based on the job status when a custom `message` is not provided.
