# KaoJai GitHub Actions

KaoJai.ai maintains a collection of open-source automations that improve the visibility of GitHub Action results. Our flagship
composite action, **Workflow Notify**, turns every workflow build result into a polished Slack notification so teams can react to
pipeline changes immediately. The action uses the official Slack API behind the scenes, ensuring dependable delivery without
custom web service maintenance.

## Overview

`workflow-notify` is a reusable GitHub Action that translates any workflow status into a rich Slack message. It collects
metadata about the run—repository, branch, triggering actor, commit SHA, optional semantic version, and more—before pushing a
message that clearly communicates the job outcome. Whether you need to broadcast a successful deploy or escalate a failed build,
the action keeps the entire team aligned.

### Why send GitHub Action results to Slack?

- **Close the feedback loop.** Ship build results straight to Slack so engineers, product managers, and stakeholders understand
  the current workflow status without opening GitHub.
- **Automate incident awareness.** Pair the action with `failure()` conditions to instantly notify on-call responders about
  critical deploy or test regressions.
- **Maintain historical context.** Slack threads capture the full narrative around each notification, including follow-up
  discussions and remediation steps.

## Workflow Notify action

`workflow-notify` is intentionally lightweight yet expressive, offering sensible defaults with room for customization.

### Key capabilities

- ✅ Works with any workflow, job, or step—wrap it in `success()`, `failure()`, or `always()` conditionals to target specific
  GitHub Action results.
- ✅ Ships with opinionated formatting for Slack notifications, including emoji, colors, and metadata blocks that highlight the
  workflow build result.
- ✅ Runs entirely on GitHub-hosted infrastructure and communicates with Slack through the trusted [`slackapi/slack-github-action`](https://github.com/slackapi/slack-github-action).

### Slack notification preview

Success notification example:

![Successful workflow notification](workflow-notify/images/success.png)

Failure notification example:

![Failed workflow notification](workflow-notify/images/failure.png)

### How the Slack API integration works

1. The action checks out the repository so it can read git metadata and detect tags associated with the commit.
2. It prepares a JSON payload that maps each workflow build result (success, failure, cancelled, etc.) to a Slack color and
   emoji, adds repository links, and optionally references the release version.
3. Finally, it calls the Slack API via the official Slack GitHub Action, sending the payload to the configured channel.

This end-to-end flow means you can rely on consistent Slack notify behavior across every project without custom scripting.

## Usage

### Prerequisites

1. Create an [Incoming Webhook](https://api.slack.com/messaging/webhooks) in the target Slack workspace.
2. Store the webhook URL in your repository settings as an encrypted secret, for example `SLACK_WEBHOOK_URL`.

### Basic workflow example

```yaml
data-platform-notify:
  runs-on: ubuntu-latest
  steps:
    - name: Run build
      run: |
        npm ci
        npm run build

    - name: Slack Notify build result
      uses: kaojai-ai/github-action/workflow-notify@main
      with:
        slack_webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
        channel: deployment
        prefix: "🚀 Deploy"
```

### Customize the notification message

Add optional inputs to tailor the Slack message without editing the composite action:

```yaml
- name: Workflow result notifier
  if: ${{ always() }}
  uses: kaojai-ai/github-action/workflow-notify@main
  with:
    slack_webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
    channel: builds
    prefix: "⚙️ Workflow"
    bot_name: "kj-ops-bot"
    message: |
      *Environment:* production
      *Status reason:* Smoke tests failed on Checkout stage
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
| `status` | _empty_ | Overrides GitHub's computed job status (`success`, `failure`, `cancelled`, etc.). Leave empty to autodetect. |
| `version` | _empty_ | Overrides the version shown in the notification. Useful for custom release tags. |

### Pro tips

- Guard the action with `success()`, `failure()`, or `always()` depending on the desired behavior.
- Combine the optional `message` input with multi-line Markdown to summarize release notes, deployment environments, or changelog
  highlights.
- Surface build numbers, environments, or feature flags by referencing `${{ github.ref_name }}`, `${{ github.run_number }}` or
  other context variables.
- Pair the action with scheduled workflows to send daily GitHub Action result digests to Slack if real-time alerts are too noisy.

## FAQ

### Does the action support other notification providers?

The action is designed to be provider-agnostic. Today it focuses on Slack notifications, but the `provider_name` input ensures
future integrations can co-exist without breaking existing workflows.

### Can I send multiple workflow build results in a single message?

Dispatch the action multiple times within the same workflow or job if you need separate notifications for build, test, and deploy
stages. Each invocation sends one payload through the Slack API, giving you full control over the message cadence.

## License

Distributed under the [MIT](LICENSE).
