# Visiono Claude Plugin

Claude Code plugin for Visiono API integration assistance.

## Installation

1. Add the marketplace:
   ```
   /plugin marketplace add visiono-io/visiono-claude-plugin
   ```

2. Install the plugin:
   ```
   /plugin install visiono-api@visiono
   ```

## Features

- **Photo Requests API** - Create expiring photo collection links
- **Smart Links API** - Create permanent, reusable photo collection links
- **Webhooks** - Real-time notifications when photos are submitted
- **Code examples** in PHP, JavaScript, Python
- **Signature verification** patterns for secure webhook handling
- **Best practices** and common error solutions

## Usage

The skill activates automatically when you mention "Visiono API" or ask about:

- Photo requests
- Smart links
- Webhooks
- Download URLs
- QR codes

### Example Prompts

```
Help me create a photo request with the Visiono API
```

```
How do I verify Visiono webhook signatures in Node.js?
```

```
Create a Smart Link for vehicle inspections
```

## What's Included

The plugin provides expert guidance on:

| Feature | Description |
|---------|-------------|
| Photo Requests | Single-use, expiring photo collection links |
| Smart Links | Permanent, reusable links with custom slugs |
| Webhooks | Event notifications (created, submitted, expired) |
| Download URLs | Signed URLs for accessing submitted photos |
| QR Codes | Ready-to-use QR code URLs (no generation needed) |

## Links

- [Visiono Documentation](https://visiono.io/docs)
- [API Reference](https://www.visiono.io/docs/api/v1)
- [OpenAPI Spec](https://www.visiono.io/docs/api/v1.openapi)

## License

MIT License - see [LICENSE](LICENSE) file.
