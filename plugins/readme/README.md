# README Plugin

Documentation automation tools for project files.

## Commands

| Command | Description |
|---------|-------------|
| `/readme:init` | Generate README based on project type |
| `/readme:update` | Update README with project changes |

## /readme:init

Generates a comprehensive README.md file based on project analysis.

### Workflow

```bash
/readme:init
# → Analyzing project structure...
# → "Project status: In Development. Is this correct?"
# → Yes
# → "Project type: Frontend. Is this correct?"
# → 1. Frontend
# ✓ README.md created successfully!
```

### Features

- **Project Type Detection**: Automatically detects Frontend/Backend/Full-stack
- **Template Selection**: Uses appropriate template for project type
- **Status Detection**: Detects "In Development" vs "Production" based on version
- **Placeholder Support**: Uses placeholders for sections without content

### Supported Project Types

| Type | Detection | Special Sections |
|------|-----------|------------------|
| Frontend | react/vue/next/angular in package.json | Screenshots |
| Backend | pom.xml, build.gradle, go.mod | API Endpoints, Auth Flow |
| Full-stack | Both indicators present | Combined |
| Other | None of above | Auto-generated |

### Version Detection

| Version | Status |
|---------|--------|
| `< 1.0.0` | In Development (shows warning banner) |
| `>= 1.0.0` | Production |

### Placeholders

When content is not available:

| Section | Placeholder |
|---------|-------------|
| Technical Challenges | `🚧 Technical challenges and solutions will be documented as the project evolves.` |
| Screenshots | `🚧 Screenshots are being prepared...` |
| Key Features | `🚧 Features are being developed. Check back soon!` |
| API Endpoints | `🚧 API documentation coming soon.` |

## /readme:update

Updates existing README.md based on project changes.

### Workflow

```bash
/readme:update
# → "What changes to reflect?"
# → 2. PR changes (main..HEAD)
# → Analyzing changes...
# → "Update these sections?"
# → Features: "Add: User authentication"
# → Tech Stack: "Add: React Query"
# ✓ README.md updated successfully!
```

### Analysis Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| Full project scan | Scans entire project structure | Major refactoring |
| PR changes | Analyzes main..HEAD diff | After creating PR |
| Describe manually | User describes changes | Specific updates |

### Deep Analysis

Same depth as `/git:pr`:

| Step | Analysis |
|------|----------|
| Commit Analysis | `git log origin/main..HEAD` |
| Diff Analysis | `git diff origin/main..HEAD` |
| Changed Files | `git diff --name-only` |
| **File Content** | **Read tool for actual content** |
| Context | Conversation history |

### Change Detection

| Change | Action |
|--------|--------|
| Version >= 1.0.0 | Suggest removing "In Development" |
| Screenshots added | Suggest adding image links |
| New features | Suggest updating Features section |
| Tech changes | Suggest updating Tech Stack |

## git:pr Integration

After creating a PR with `/git:pr`:

```
✓ Pull Request created successfully!

📄 Update README?
1. Yes
2. No
```

Selecting "Yes":
- Runs `/readme:update`
- Auto-selects "PR changes" mode
- Reuses analysis context (saves tokens)

## Template Sections

### Frontend

- Overview
- Key Features
- Tech Stack
- Architecture
- Project Structure
- **Screenshots**
- Technical Challenges & Solutions
- Getting Started
- Roadmap
- Author

### Backend

- Overview
- Key Features
- Tech Stack
- Architecture (ASCII diagram)
- Project Structure
- **API Endpoints**
- **Authentication Flow**
- Technical Challenges & Solutions
- Getting Started
- Roadmap
- Author

## License

MIT
