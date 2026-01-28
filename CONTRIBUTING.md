# Contributing to GrepAI Skills

Thank you for your interest in contributing to GrepAI Skills! This document provides guidelines for contributions.

## How to Contribute

### Reporting Issues

- Check if the issue already exists
- Use a clear and descriptive title
- Provide as much relevant information as possible
- Include steps to reproduce if reporting a bug

### Adding New Skills

1. **Fork** the repository
2. **Create a branch**: `git checkout -b feature/new-skill-name`
3. **Add your skill** following the structure below
4. **Update** `marketplace.json` to include your skill
5. **Submit** a pull request

## Skill Structure

Each skill follows this structure:

```
skills/
└── [category]/
    └── [skill-id]/
        └── SKILL.md
```

### SKILL.md Format

```markdown
---
name: grepai-skill-name
description: Brief description (60-150 chars). Use this skill when [use case].
---

# Skill Title

Introduction explaining what this skill covers.

## When to Use This Skill

- Use case 1
- Use case 2

## Main Content

Detailed content with examples...

## Examples

```bash
# Example commands
grepai search "query"
```

## Best Practices

- Practice 1
- Practice 2

## Common Mistakes

❌ **Mistake:** Description
✅ **Solution:** How to fix it

## Output Format

Template showing expected output.
```

## Naming Conventions

- **Skill IDs:** `grepai-[descriptive-name]` (lowercase, hyphens)
- **Categories:** lowercase (getting-started, configuration, search, etc.)
- **Files:** Always `SKILL.md` (uppercase)

## Categories

| Category | Purpose |
|----------|---------|
| `getting-started` | Installation and quickstart |
| `configuration` | Configuration and setup |
| `embeddings` | Embedding providers |
| `storage` | Storage backends |
| `indexing` | Indexing and watching |
| `search` | Semantic search |
| `trace` | Call graph analysis |
| `integration` | MCP and AI agent integration |
| `advanced` | Workspaces and advanced features |

## Quality Guidelines

### Content

- Be **specific** and **actionable**
- Include **working examples**
- Cover **common mistakes** and solutions
- Keep language **clear and concise**

### Format

- Use proper Markdown formatting
- Include code blocks with syntax highlighting
- Use tables for comparisons
- Add emoji sparingly (only for visual markers like ✅ ❌)

### Testing

- Verify all commands work with current GrepAI version
- Test examples before submitting
- Ensure YAML frontmatter is valid

## Pull Request Process

1. Ensure your skill follows the format guidelines
2. Update `marketplace.json` if adding a new skill
3. Update `CHANGELOG.md` with your changes
4. Submit a PR with a clear description
5. Address any review feedback

## Code of Conduct

Please read and follow our [Code of Conduct](CODE_OF_CONDUCT.md).

## Questions?

Feel free to open an issue for questions or discussions.

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
