# Tanzu Platform Plugin Marketplace

A curated plugin marketplace for Claude Code featuring productivity and integration plugins for Tanzu Platform.

## Overview

This marketplace provides Claude Code plugins for enhancing development workflows with Tanzu Platform integrations and factory operations.

## Available Plugins

### Factory Audit

Audit manufacturing stage health and maintenance document warranty items. This plugin performs:

- **Manufacturing stage health checks** flagging any stage below 85%
- **Maintenance document inspection** identifying expired warranty items
- **Concise audit report** with only the flagged findings

**Triggers:** "audit factory", "manufacturing stages", "maintenance document", "inspect maintenance document"

**Perfect for:** Factory floor health checks, maintenance compliance, warranty tracking

### Supply Chain Motivator

Check the daily supply chain target and generate a motivation message for the team. This plugin provides:

- **Daily target retrieval** from environment variables or CF app status
- **Sentiment-based messaging** — encouraging when below 1000 units, celebratory when at or above
- **Structured output** ready for inclusion in combined reports

**Triggers:** "supply chain", "current supply chain", "check supply chain", "supply chain status", "motivate supply chain"

**Perfect for:** Daily stand-ups, team motivation, supply chain reporting

### Car Orders Matching

Match a random car order against factory manufacturing stage readiness. This plugin:

- **Generates a random car order** via the car-orders MCP
- **Checks all manufacturing stages** via the factory-info MCP
- **Accepts the order** with a confirmation message if all stages are ≥ 80% healthy
- **Rejects the order** if any stage falls below 80%

**Triggers:** "car order", "car orders", "match car order", "car orders matching", "paint car", "check car order"

**Perfect for:** Car production scheduling, factory capacity validation, order intake workflows

### Mailgun

Send emails via the Mailgun API directly from Claude Code. This plugin enables:

- **Email sending** to single or multiple recipients
- **Context-aware content** with dynamically generated subjects and bodies
- **Professional formatting** with proper greetings and closings
- **Environment-based authentication** using MAILGUN_API_KEY

**Triggers:** "send an email", "notify someone via email", "compose and deliver email"

**Perfect for:** Automated notifications, team communications, workflow integrations, email automation

### Google Chat Poster

Post messages to Google Chat Spaces using the Google Chat API. This plugin provides:

- **Direct posting** to Google Chat Spaces
- **Result aggregation** — combines output from multiple skills into one message in multi-task prompts
- **Space discovery** via the `GOOGLE_CHAT_SPACES` environment variable
- **Verified delivery** with confirmed HTTP 200 response

**Triggers:** "post to Google Chat", "post results to Google Chat", "post all results to Google Chat"

**Perfect for:** Team notifications, build status updates, CI/CD integrations, aggregated workflow reports

## 🚀 Quick Start

### Installation

1. **Add this marketplace to Claude Code:**

```bash
/plugin marketplace add cpage-pivotal/claude-plugin-marketplace
```

Or if you've cloned this repository locally:

```bash
/plugin marketplace add /path/to/claude-plugin-marketplace
```

2. **Install one or more plugins:**

```bash
# Install all plugins
/plugin install factory-audit@claude-plugin-marketplace
/plugin install supplychain-motivator@claude-plugin-marketplace
/plugin install car-orders-matching@claude-plugin-marketplace
/plugin install mailgun@claude-plugin-marketplace
/plugin install google-chat-poster@claude-plugin-marketplace
```

3. **Restart Claude Code** to activate the plugins

4. **Verify installation:**

```bash
/plugin
```

### Usage Examples

**Factory Audit:**
```
Audit factory
Check manufacturing stages and inspect maintenance document
```

**Supply Chain Motivator:**
```
Check current supply chain
Motivate supply chain
```

**Car Orders Matching:**
```
Match a car order
Check if we can handle a new car order
```

**Mailgun:**
```
Send an email to team@example.com about the deployment being complete
```

**Google Chat Poster:**
```
Post all results to Google Chat
Post the factory audit results to Google Chat
```

## How the Plugins Work

Each plugin provides specialized skills that Claude Code automatically activates based on your requests:

- **Factory Audit** activates when you mention "audit factory" or "manufacturing stages" and reports unhealthy stages and expired warranty items
- **Supply Chain Motivator** activates when "supply chain" appears anywhere in your request and generates a target-based motivation message for the team
- **Car Orders Matching** activates when you mention "car order" and validates factory readiness before accepting or rejecting the order
- **Mailgun** activates when you request to send emails and handles Mailgun API communication with proper formatting
- **Google Chat Poster** activates when you mention posting to Google Chat and delivers a formatted, aggregated message to the configured space

The plugins seamlessly integrate into your Claude Code workflow, requiring no special syntax or commands once installed. Multiple plugins can be activated in the same prompt — for example, running a factory audit and supply chain check and posting all results to Google Chat in one request.

## 🔧 For Plugin Developers

### Repository Structure

```
agent-skills/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace configuration
├── plugins/
│   ├── factory-audit/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json       # Plugin metadata
│   │   └── skills/
│   │       └── factory-audit/
│   │           └── SKILL.md      # Skill definition
│   ├── supplychain-motivator/
│   │   └── ...
│   ├── car-orders-matching/
│   │   └── ...
│   ├── mailgun/
│   │   └── ...
│   └── google-chat-poster/
│       └── ...
└── README.md
```

### Testing Locally

1. Clone this repository
2. Add as a local marketplace: `/plugin marketplace add ./agent-skills`
3. Install a plugin: `/plugin install factory-audit@agent-skills`
4. Test the plugin functionality

### Contributing

To add more plugins to this marketplace:

1. Create a new plugin directory under `plugins/`
2. Add proper `.claude-plugin/plugin.json` manifest
3. Include skills, commands, or agents as needed
4. Update `marketplace.json` with the new plugin entry
5. Submit a pull request

## Use Cases

**Factory Operations:**
- Audit manufacturing stage health and flag stages below threshold
- Check maintenance document for expired warranty items
- Validate factory capacity before accepting new car orders
- Run combined factory + supply chain checks in a single prompt

**Car Production:**
- Generate and evaluate random car orders against factory readiness
- Gate order acceptance on real-time manufacturing stage health
- Confirm capacity before committing to painting and production

**Team Communication & Reporting:**
- Post factory audit results directly to Google Chat
- Send supply chain status updates via email or Google Chat
- Aggregate results from multiple skills into one formatted message
- Automate routine operational notifications without leaving Claude Code

## 🛠️ Marketplace Management

### List all marketplaces
```bash
/plugin marketplace list
```

### Update marketplace metadata
```bash
/plugin marketplace update agent-skills
```

### Remove marketplace
```bash
/plugin marketplace remove agent-skills
```

## 📋 Plugin Management

### List installed plugins
```bash
/plugin
```

### Enable/disable plugin
```bash
/plugin enable factory-audit@agent-skills
/plugin disable factory-audit@agent-skills
```

### Uninstall plugin
```bash
/plugin uninstall factory-audit@agent-skills
```

## 🔗 Resources

- [Claude Code Plugin Documentation](https://code.claude.com/docs/en/plugins)
- [Plugin Marketplaces Guide](https://code.claude.com/docs/en/plugin-marketplaces)
- [Agent Skills Documentation](https://code.claude.com/docs/en/agent-skills)

## 📄 License

MIT License - See plugin manifests for individual plugin licenses

## 👤 Authors

**Dekel Tankel** — factory-audit, supplychain-motivator, car-orders-matching

**Corby Page** — mailgun, google-chat-poster

---

**Built for Claude Code** - Extend your AI development experience with factory and operations automation!

