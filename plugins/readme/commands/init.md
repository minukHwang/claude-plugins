---
description: Generate README.md based on project type and structure
---

# Initialize README

Analyze the project and generate a comprehensive README.md file in English.

## Step 1: Check Existing README

```bash
ls README.md 2>/dev/null
```

If README.md exists:
"README.md already exists. Use `/readme:update` to modify it, or delete it first to regenerate."
→ Stop

## Step 2: Project Analysis

### 2.1 Project Structure Scan

```bash
ls -la
```

### 2.2 Package/Build File Detection

```bash
# Node.js
cat package.json 2>/dev/null

# Java
cat pom.xml 2>/dev/null
cat build.gradle 2>/dev/null

# Go
cat go.mod 2>/dev/null

# Python
cat requirements.txt 2>/dev/null
cat pyproject.toml 2>/dev/null

# Rust
cat Cargo.toml 2>/dev/null
```

### 2.3 Key Files Reading

Use Read tool to examine:
- Main entry files
- Configuration files
- Source directory structure

## Step 3: Project Status Detection

### 3.1 Version Check

| Source | Check |
|--------|-------|
| package.json | `version` field |
| pom.xml | `<version>` tag |
| Cargo.toml | `version` field |

**Version Rules:**
- `< 1.0.0` → In Development
- `>= 1.0.0` → Production

### 3.2 Screenshot Folder Check

```bash
ls screenshots/ 2>/dev/null || ls docs/images/ 2>/dev/null
```

- Empty/Not found → Use placeholder: `🚧 Screenshots are being prepared...`
- Found → Generate image links

### 3.3 Status Confirmation

**Ask user (AskUserQuestion):**
"Project status: [In Development/Production]. Is this correct?"

| Option | Description |
|--------|-------------|
| Yes | Proceed with detected status |
| No | Toggle status |

## Step 4: Project Type Detection

### Detection Logic

| Indicator | Type |
|-----------|------|
| package.json with react/vue/next/angular | Frontend |
| pom.xml, build.gradle, go.mod | Backend |
| package.json with express/nestjs/fastify | Backend |
| Frontend + Backend indicators | Full-stack |
| None of above | Other |

### Type Confirmation

**Ask user (AskUserQuestion):**
"Project type: [Type]. Is this correct?"

| Option | Description |
|--------|-------------|
| Frontend | Frontend project |
| Backend | Backend project |
| Full-stack | Full-stack project |
| Other | Auto-generate based on analysis |

## Step 5: Generate README

### Frontend Template

```markdown
# Project Name

> Brief description

## ⚠️ Development Status (if applicable)

> **Note:** This project is currently in development. Features and documentation are being actively worked on.

## Table of Contents

1. [Overview](#overview)
2. [Key Features](#key-features)
3. [Tech Stack](#tech-stack)
4. [Architecture](#architecture)
5. [Project Structure](#project-structure)
6. [Screenshots](#screenshots)
7. [Technical Challenges & Solutions](#technical-challenges--solutions)
8. [Getting Started](#getting-started)
9. [Roadmap](#roadmap)
10. [Author](#author)

---

## Overview

[2-3 sentences about the project]

## Key Features

- **Feature 1**: Description
- **Feature 2**: Description
- **Feature 3**: Description

(or placeholder: `🚧 Features are being developed. Check back soon!`)

## Tech Stack

### Core
- Framework (version)
- Language (version)

### Libraries
- Library 1
- Library 2

## Architecture

[Architecture description or diagram]

## Project Structure

```
project/
├── src/
│   ├── components/
│   ├── pages/
│   └── ...
└── ...
```

## Screenshots

[Screenshots or placeholder: `🚧 Screenshots are being prepared...`]

## Technical Challenges & Solutions

### Challenge 1: [Title]
**Problem:** Description
**Solution:** How it was solved

(or placeholder: `🚧 Technical challenges and solutions will be documented as the project evolves.`)

## Getting Started

### Prerequisites
- Node.js (version)
- Package manager

### Installation
```bash
# Clone
git clone <repo-url>

# Install
npm install

# Run
npm run dev
```

## Roadmap

- [ ] Feature 1
- [ ] Feature 2

## Author

**Name** - [GitHub](link)
```

### Backend Template

```markdown
# Project Name

> Brief description

## ⚠️ Development Status (if applicable)

> **Note:** This project is currently in development.

## Table of Contents

1. [Overview](#overview)
2. [Key Features](#key-features)
3. [Tech Stack](#tech-stack)
4. [Architecture](#architecture)
5. [Project Structure](#project-structure)
6. [API Endpoints](#api-endpoints)
7. [Authentication Flow](#authentication-flow)
8. [Technical Challenges & Solutions](#technical-challenges--solutions)
9. [Getting Started](#getting-started)
10. [Roadmap](#roadmap)
11. [Author](#author)

---

## Overview

[2-3 sentences]

## Key Features

- **Feature 1**: Description
- **Feature 2**: Description

(or placeholder)

## Tech Stack

### Core
- Framework (version)
- Language (version)
- Database

### Infrastructure
- Docker
- Cloud services

## Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Client    │────▶│   Server    │────▶│  Database   │
└─────────────┘     └─────────────┘     └─────────────┘
```

## Project Structure

```
project/
├── src/
│   ├── controllers/
│   ├── services/
│   ├── models/
│   └── ...
└── ...
```

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /api/resource | Get resources |
| POST | /api/resource | Create resource |

(or placeholder: `🚧 API documentation coming soon.`)

## Authentication Flow

[Auth flow description]

(or placeholder if not implemented)

## Technical Challenges & Solutions

(same as Frontend)

## Getting Started

### Prerequisites
- Runtime (version)
- Database

### Installation
```bash
# Clone
git clone <repo-url>

# Install dependencies
./install.sh

# Run
./run.sh
```

### Environment Variables

| Variable | Description |
|----------|-------------|
| DATABASE_URL | Database connection string |
| JWT_SECRET | JWT signing secret |

## Roadmap

- [ ] Feature 1
- [ ] Feature 2

## Author

**Name** - [GitHub](link)
```

### Full-stack Template

Combine Frontend + Backend sections:
- Table of Contents with all applicable sections
- Screenshots section from Frontend
- API Endpoints section from Backend
- Both frontend and backend in Tech Stack

### Other Template

Auto-generate based on analysis:
- Table of Contents with generated sections
- Overview (from README patterns or package description)
- Tech Stack (from detected dependencies)
- Project Structure (from directory scan)
- Getting Started (from detected build system)
- All other sections with placeholders

## Step 6: Write File

Use Write tool to create README.md

## Output Format

### On Success:
```
✓ README.md created successfully!

Sections generated:
- Overview
- Key Features
- Tech Stack
- [other sections...]

Run `/readme:update` to modify content later.
```

### On Failure:
```
✗ Failed to create README: <error message>
```

## Constraints

1. **English only**: All content in English
2. **No section skipping**: Use placeholders for missing content
3. **Accurate detection**: Verify project type with user
4. **Template consistency**: Follow template structure strictly
5. **Git info**: Extract repo URL from git remote if available
