# ChatGPT App Builder Skill

An [AgentSkills](https://agentskills.io) skill for building ChatGPT Apps using the Apps SDK and MCP protocol.

## Overview

This skill guides AI agents through the complete lifecycle of building ChatGPT Apps:

1. **Fit Evaluation** - Determine if a ChatGPT App is right for your product
2. **Architecture Design** - Plan MCP servers, tools, and widgets
3. **Implementation** - Build with Node.js, TypeScript, and React
4. **Testing** - Local development with ngrok, ChatGPT integration testing
5. **Deployment** - Production hosting and App Store submission

## Installation

### For Claude Code Users

Add to your Claude Code settings (`~/.claude/settings.json`):

```json
{
  "skills": [
    "/path/to/chatgpt-app-builder"
  ]
}
```

### For Other Agents

Use [skills-ref](https://github.com/agentskills/agentskills/tree/main/skills-ref) to generate the skill prompt:

```bash
skills-ref to-prompt /path/to/chatgpt-app-builder
```

## Skill Structure

```
chatgpt-app-builder/
├── SKILL.md                    # Main skill entry point
└── references/
    ├── fit_evaluation.md       # Know/Do/Show framework
    ├── node_chatgpt_app.md     # Node.js MCP implementation
    ├── widget_development.md   # React widget development
    ├── oauth_integration.md    # OAuth 2.1 authentication
    ├── submission_requirements.md  # App Store requirements
    ├── troubleshooting.md      # Common issues & solutions
    └── chatgpt_app_best_practices.md  # Best practices
```

## Key Features

- **Know/Do/Show Framework** - Evaluate product fit for ChatGPT
- **MCP Server Templates** - Production-ready Node.js/TypeScript code
- **Widget Development** - React components with ChatGPT theming
- **OAuth 2.1 Guide** - Self-hosted and provider-based authentication
- **Deployment Guides** - Fly.io, Railway, and other platforms
- **Submission Checklist** - Complete App Store requirements

## Triggers

The skill activates on these keywords:
- "ChatGPT app"
- "Apps SDK"
- "build for ChatGPT"
- "ChatGPT integration"
- "MCP server for ChatGPT"
- "submit to ChatGPT"

## Requirements

- Node.js 18+
- npm
- Network access (for ngrok testing)

## Validation

Validate this skill against the AgentSkills specification:

```bash
pip install skills-ref
skills-ref validate ./chatgpt-app-builder
```

## License

MIT

## Author

Bayram Annakov ([onsa.ai](https://onsa.ai))

## Links

- [AgentSkills Specification](https://agentskills.io/specification)
- [ChatGPT Apps Documentation](https://platform.openai.com/docs/apps)
- [MCP Protocol](https://modelcontextprotocol.io)
