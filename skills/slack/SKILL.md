---
name: slack
description: Send a message to Slack. Use this skill when asked to send to or notify a user in Slack.
allowed-tools: Bash
user-invocable: true
---

# Slack

Send a single message to Slack using the `PANBOTICON_SLACK_INCOMING_WEBHOOK_URL` or `SLACK_INCOMING_WEBHOOK_URL` found in the environment. Use this skill when asked to send to or notify a user in Slack.

Run `./slack` (in this skill directory) and provide the full JSON payload on stdin. The simplest payload is `{"text":"TEXT","type":"mrkdwn"}`, substituting the message you want to send for `TEXT`. This command will print "ok" in case of success; any other outcome should be considered a failure.

Refer to <https://docs.slack.dev/messaging/sending-messages-using-incoming-webhooks/> for advanced message formatting capabilities.
