# Nullstone Skill for Claude Code

A Claude Code skill for deploying and managing applications on AWS via [Nullstone](https://nullstone.io) infrastructure orchestration platform.

## Features

- **Deploy Applications** - Push Docker images and deploy to AWS ECS Fargate
- **View Logs** - Stream live application logs or view historical logs
- **Check Status** - Monitor application health and deployment status
- **Execute Commands** - SSH into running containers or run one-off commands
- **Manage Environments** - Work with staging, production, and preview environments
- **GitOps Support** - Understand and help configure `.nullstone/` configuration files

## Installation

This skill can be installed via the Claude Code plugin system or manually.

### Option 1: Via Plugin System (Recommended)

```bash
# Add this repository as a marketplace
/plugin marketplace add iou-financial/nullstone-skill

# Install the plugin
/plugin install nullstone-skill@nullstone-skill
```

Verify installation by running `/skills` to confirm the skill is available.

### Option 2: Manual Git Clone

Install directly from GitHub to your skills directory:

**Global Installation (Available Everywhere):**
```bash
# Navigate to your Claude skills directory
cd ~/.claude/skills

# Clone the skill
git clone https://github.com/iou-financial/nullstone-skill.git

# The skill is now available in Claude Code
```

**Project-Specific Installation:**
```bash
# Install in a specific project
cd /path/to/your/project
mkdir -p .claude/skills
cd .claude/skills
git clone https://github.com/iou-financial/nullstone-skill.git
```

### Option 3: Download Release

1. Download the latest release from [GitHub Releases](https://github.com/iou-financial/nullstone-skill/releases)
2. Extract to:
   - Global: `~/.claude/skills/nullstone-skill`
   - Project: `/path/to/your/project/.claude/skills/nullstone-skill`

### Verify Installation

Run `/skills` to confirm the skill is loaded, then ask Claude to check a Nullstone app status.

## Prerequisites

### 1. Install Nullstone CLI

```bash
# macOS
brew install nullstone-io/tap/nullstone

# Linux
curl -sSL https://nullstone.io/install.sh | bash
```

### 2. Configure Nullstone CLI

```bash
nullstone configure
```

This will prompt you to enter your Nullstone API key. Get your API key from your [Nullstone Profile](https://app.nullstone.io/profile).

### 3. Set Organization (Optional)

```bash
nullstone set-org <your-org-name>
```

## Usage

Once installed, ask Claude to help with Nullstone operations:

- "Deploy proton to staging"
- "Show me the logs for nucleus in production"
- "Check the status of the zoltar app"
- "SSH into the proton-worker container"
- "What apps are deployed in the primary stack?"

## Configuration

### Environment Variables

For CI/CD and automation, set these environment variables:

| Variable | Description |
|----------|-------------|
| `NULLSTONE_ORG` | Your Nullstone organization name |
| `NULLSTONE_API_KEY` | API key from your Nullstone profile |
| `NULLSTONE_STACK` | Default stack name |
| `NULLSTONE_ENV` | Default environment name |

### GitOps Configuration

Place configuration files in your project's `.nullstone/` directory:

```
.nullstone/
  config.yml      # Base configuration
  staging.yml     # Staging overrides
  production.yml  # Production overrides
  previews.yml    # Preview environment config
```

See the [SKILL.md](skills/nullstone-skill/SKILL.md) for detailed configuration examples.

## Example Workflows

### Deploy to Staging

```bash
# Build Docker image
docker build -t myapp:latest .

# Push and deploy
nullstone push --app=myapp --source=myapp:latest --stack=primary
nullstone deploy --app=myapp --env=staging --stack=primary -w
```

### View Live Logs

```bash
nullstone logs --app=myapp --env=staging --stack=primary -t
```

### Run Database Migrations

```bash
nullstone exec --app=myapp --env=staging --stack=primary -- rails db:migrate
```

## GitHub Actions Integration

```yaml
name: Deploy

on:
  push:
    branches: [main]

env:
  NULLSTONE_ORG: your-org
  NULLSTONE_API_KEY: ${{ secrets.NULLSTONE_API_KEY }}

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: nullstone-io/setup-nullstone-action@v0

      - name: Build and Push
        run: |
          docker build -t myapp:latest .
          nullstone push --app=myapp --source=myapp:latest
          nullstone deploy --app=myapp --env=production --stack=primary -w
```

## Security

**This skill contains NO secrets, API keys, or credentials.**

- All authentication is handled via the Nullstone CLI which stores credentials securely in `~/.nullstone/`
- API keys should ONLY be stored in:
  - GitHub Secrets (for CI/CD)
  - Environment variables (for local use)
  - AWS Secrets Manager (for application runtime)
- NEVER commit API keys, tokens, or credentials to this repository
- The `nullstone configure` command handles secure credential storage locally

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

MIT License - see [LICENSE](LICENSE) for details.

## Links

- [Nullstone Documentation](https://docs.nullstone.io/)
- [Nullstone CLI Reference](https://docs.nullstone.io/cli/)
- [Claude Code Documentation](https://docs.anthropic.com/claude-code)
