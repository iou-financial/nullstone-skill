# Contributing to Nullstone Skill

Thank you for your interest in contributing to the Nullstone Skill for Claude Code!

## Getting Started

1. Fork the repository
2. Clone your fork locally
3. Create a feature branch

## Development Setup

1. Install the skill locally for testing:
   ```bash
   # Link to your Claude Code plugins directory
   ln -s /path/to/nullstone-skill ~/.claude/plugins/nullstone-skill
   ```

2. Make your changes to the skill files

3. Test your changes in Claude Code

## File Structure

```
nullstone-skill/
  .claude-plugin/
    plugin.json        # Plugin metadata
    marketplace.json   # Marketplace listing info
  skills/
    nullstone-skill/
      SKILL.md         # Main skill documentation (loaded by Claude)
      lib/             # Helper scripts (if needed)
  README.md            # Project readme
  LICENSE              # MIT license
  CONTRIBUTING.md      # This file
```

## Making Changes

### Updating the Skill

The main skill content is in `skills/nullstone-skill/SKILL.md`. This file is loaded by Claude Code when the skill is invoked.

Key sections:
- **Quick Reference** - Command cheat sheet
- **Common Operations** - Step-by-step instructions
- **Deployment Workflows** - CI/CD patterns
- **GitOps Configuration** - Config file examples
- **Troubleshooting** - Common issues and solutions

### Guidelines

1. **Keep it practical** - Include real-world examples that work
2. **Be concise** - Claude has context limits, so be efficient with words
3. **Test commands** - Verify all CLI commands actually work
4. **Update versions** - Bump version in plugin.json when releasing

## Submitting Changes

1. Commit your changes with clear messages
2. Push to your fork
3. Open a Pull Request with:
   - Description of changes
   - Any testing you've done
   - Screenshots if UI-related

## Code of Conduct

Be respectful and constructive. We're all here to make great tools.

## Questions?

Open an issue for questions or discussions about the skill.
