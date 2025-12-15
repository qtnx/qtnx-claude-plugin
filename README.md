# qtnx-claude-plugin

Claude Code plugin with skills for task delegation, PR review workflows, and code review iteration.

## Skills Included

| Skill | Trigger | Description |
|-------|---------|-------------|
| `clink-codex-delegate` | "clink codex" | Delegate coding tasks to clink codex/gemini with structured task briefs |
| `gh-pr-review` | "PR review", "show PR comments" | Manage GitHub PR inline comments from terminal using gh-pr-review extension |
| `iterate-review-with-codex` | "codex review", "fix pr code review" | Iterate code review workflow with codex |

## Installation

### Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed and configured
- Git

### Method 1: Install from GitHub Marketplace (Recommended)

```bash
# Start Claude Code
claude

# Add the marketplace
/plugin marketplace add qtnx/qtnx-claude-plugin

# Install the plugin
/plugin install qtnx-plugin@qtnx-marketplace
```

### Method 2: Install from Local Directory

```bash
# Clone the repository
git clone https://github.com/qtnx/qtnx-claude-plugin.git

# Start Claude Code
claude

# Add local marketplace
/plugin marketplace add ./qtnx-claude-plugin

# Install the plugin
/plugin install qtnx-plugin@qtnx-marketplace
```

### Verify Installation

After installation, restart Claude Code and verify:

```bash
# Check available commands
/help

# Check plugins
/plugin
```

## Usage

Once installed, the skills activate automatically based on trigger keywords:

### Delegate Tasks to Clink Codex

```
Use clink codex to implement the user authentication module
```

If codex fails, switch to gemini:

```
Use clink gemini instead of codex for this task
```

### PR Review with gh-pr-review

```
Show me the unresolved PR review comments
```

```
Reply to the review thread about the null check
```

### Iterate Code Review

```
Fix the codex review comments on this PR
```

## Plugin Structure

```
qtnx-claude-plugin/                  # Marketplace root (this repo)
├── .claude-plugin/
│   └── marketplace.json             # Marketplace manifest
├── README.md
└── qtnx-plugin/                     # Plugin directory
    ├── .claude-plugin/
    │   └── plugin.json              # Plugin metadata
    ├── commands/                    # Custom slash commands
    └── skills/
        ├── clink-codex-delegate/    # Task delegation skill
        ├── gh-pr-review/            # PR review management skill
        └── iterate-review-with-codex/ # Review iteration skill
```

## Requirements for Skills

### gh-pr-review Skill

Requires the gh-pr-review extension:

```bash
gh extension install agynio/gh-pr-review
```

### clink-codex-delegate Skill

Requires clink CLI with codex/gemini configured.

## License

MIT
