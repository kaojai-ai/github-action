# Workflow Notify

`workflow-notify` collects run metadata (repository, branch, triggering actor, commit, and optional semver tags) and emits a rich Slack message that highlights the workflow status. The action is intentionally provider-agnostic so additional destinations can be added later.

### Features

- ✅ Works with any workflow, job, or step—simply guard the action with GitHub's `success()` / `failure()` conditionals.
- ✅ Ships with expressive defaults for titles, message formatting, and emoji while remaining fully configurable through inputs.
- ✅ Uses only official GitHub and Slack actions under the hood.

### Slack notification demo

Success notification example:

![Successful workflow notification](workflow-notify/images/success.png)

Failure notification example:

![Failed workflow notification](workflow-notify/images/failure.png)

### Supported providers

| Provider | Status | Notes |
| --- | --- | --- |
| Slack | ✅ Available | Powered by [`slackapi/slack-github-action`](https://github.com/slackapi/slack-github-action). |
| Others | 🚧 Planned | The `provider_name` input is future-proofed for new integrations. |

### Usage

#### Prerequisites

1. Create an [Incoming Webhook](https://api.slack.com/messaging/webhooks) in the target Slack workspace.
2. Store the webhook URL in your repository settings as an encrypted secret, e.g. `SLACK_WEBHOOK_URL`.

#### How to use

```yaml
jobs:
  build-and-notify:
    runs-on: ubuntu-latest
    steps:
      - name: Run build
        run: |
          npm ci
          npm run build

      - name: Slack Notify
        uses: kaojai-ai/github-action/workflow-notify@main
        with:
          slack_webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
          channel: deployment
          prefix: "🚀 Deploy"
```


### Inputs

#### Mandatory

| Input | Description |
| --- | --- |
| `slack_webhook_url` | Slack Incoming Webhook URL. Store the value in a secret and reference it with `secrets.<NAME>`. |
| `channel` | Slack channel or conversation to post to. |

#### Optional

| Input | Default | Description |
| --- | --- | --- |
| `provider_name` | `slack` | Future-facing provider toggle. Currently only Slack is implemented. |
| `prefix` | `⚙️ Workflow` | Title prefix shown next to the workflow status. |
| `bot_name` | `kj-ops-bot` | Display name of the Slack bot. |
| `bot_icon_url` | `https://avatars.githubusercontent.com/u/227843191` | Optional avatar URL for the Slack bot. |
| `message` | _empty_ | Rich Slack Markdown body appended under the workflow metadata. |
| `status` | _empty_ | Overrides GitHub's computed job status (`success`, `failure`, `cancelled`, etc.). Leave empty to auto-detect. |
| `version` | _empty_ | Overrides the version shown in the notification. Useful for custom release tags. |

### Tips

- Guard the action with `success()`, `failure()`, or `always()` depending on the desired behavior.
- Combine the optional `message` input with multi-line Markdown to summarize release notes, deployment environments, or changelog highlights.
- When using GitHub environments, consider including `${{ github.ref_name }}` or `${{ github.run_number }}` inside the `message` input for extra context.

### License

Distributed under the [MIT](LICENSE).
