# Connectors

## How tool references work

Plugin skills reference external services by category rather than by specific product. This makes the plugin work with whatever tools you have connected in Cowork.

When you see `~~email`, `~~sms`, or `~~chat` in a skill, it means "use whatever connector you have set up for that category."

## Connectors for this plugin

| Category | Placeholder | Supported options |
|---|---|---|
| Email | `~~email` | Gmail (Google MCP), Resend, SendGrid, Mailgun, Postmark |
| SMS | `~~sms` | Twilio |
| Chat | `~~chat` | Slack, Microsoft Teams, Discord |

## Setting up connectors in Cowork

1. Open **Cowork Settings → Connectors**
2. Add the connector for your preferred service
3. Authenticate and grant permissions
4. The plugin will automatically use the connected tool when sending notifications

## Only email is required

The plugin works with email alone. SMS and chat are fully optional — just set `enabled: false` in `config.yaml` for any channel you don't want.

## No connector? No problem

If you don't have a connector set up for a notification channel, Claude will tell you when it tries to send and that channel will be skipped. Your other enabled channels will still work.
