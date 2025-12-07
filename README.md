# claude-plugins

Claude Code plugin marketplace by Minuk Hwang.

## Installation

```bash
# Add this marketplace to Claude Code
/plugin marketplace add minukHwang/claude-plugins

# Browse and install plugins
/plugin
```

## Available Plugins

### git

GitHub commit, PR, branch, and CI automation.

| Command | Description |
|---------|-------------|
| `/git:commit` | Stage files + generate commit with deep analysis |
| `/git:commit-light` | Stage files + generate commit (light mode, saves tokens) |
| `/git:branch` | Create branch with proper naming convention |
| `/git:pr` | Create PR with deep analysis (file reading + conversation context) |
| `/git:pr-light` | Create PR with git commands only (saves tokens) |
| `/git:ci` | Monitor GitHub Actions CI status and analyze failures |

**Install:**
```bash
/plugin install git@minukHwang-plugins
```

**[View Documentation](./plugins/git/README.md)**

## Plugin Structure

```
claude-plugins/
├── .claude-plugin/
│   └── marketplace.json     # Marketplace definition
├── plugins/
│   └── git/                 # Git automation plugin
│       ├── .claude-plugin/
│       │   └── plugin.json
│       ├── commands/
│       │   ├── commit.md        # /git:commit (deep)
│       │   ├── commit-light.md  # /git:commit-light
│       │   ├── branch.md        # /git:branch
│       │   ├── pr.md            # /git:pr (deep)
│       │   ├── pr-light.md      # /git:pr-light
│       │   └── ci.md            # /git:ci
│       └── README.md
└── README.md
```

## Contributing

Feel free to open issues or submit PRs for improvements.

## License

MIT
