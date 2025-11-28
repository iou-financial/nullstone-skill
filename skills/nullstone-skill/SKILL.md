# Nullstone Infrastructure Orchestration Skill

Deploy, manage, and monitor applications on AWS via Nullstone CLI. This skill provides commands for deploying Docker containers to AWS ECS Fargate, viewing application logs, checking deployment status, and managing environments.

## Prerequisites

**Nullstone CLI must be installed and configured:**
```bash
# Install via Homebrew
brew install nullstone-io/tap/nullstone

# Configure (one-time setup)
nullstone configure
```

**Required Environment Variables (for CI/CD or automation):**
- `NULLSTONE_ORG` - Your Nullstone organization name
- `NULLSTONE_API_KEY` - API key from your Nullstone profile

## Quick Reference

| Command | Description |
|---------|-------------|
| `nullstone status --app=<app> --env=<env>` | Check app status |
| `nullstone logs --app=<app> --env=<env> -t` | Tail live logs |
| `nullstone deploy --app=<app> --env=<env>` | Deploy latest pushed artifact |
| `nullstone push --app=<app> --source=<image>` | Push Docker image |
| `nullstone launch --app=<app> --env=<env> --source=<image>` | Push + Deploy + Wait |
| `nullstone exec --app=<app> --env=<env>` | SSH into running container |

## Common Operations

### 1. Check Application Status

View the status of your application including running tasks and load balancer health:

```bash
# Check status for specific app/env
nullstone status --app=proton --env=staging --stack=primary

# Watch status in real-time
nullstone status --app=proton --env=staging --stack=primary -w
```

### 2. View Application Logs

Stream logs from your running application:

```bash
# Tail live logs
nullstone logs --app=proton --env=staging --stack=primary -t

# View logs from last hour
nullstone logs --app=proton --env=staging --stack=primary -s 1h

# View logs from last 24 hours
nullstone logs --app=proton --env=staging --stack=primary -s 24h
```

### 3. Deploy Application

Deploy a previously pushed artifact:

```bash
# Deploy latest version
nullstone deploy --app=proton --env=staging --stack=primary

# Deploy and wait for completion
nullstone deploy --app=proton --env=staging --stack=primary -w

# Deploy specific version
nullstone deploy --app=proton --env=staging --stack=primary --version=abc123
```

### 4. Push Docker Image

Push a Docker image to Nullstone's registry:

```bash
# Push local Docker image
nullstone push --app=proton --source=proton:latest --stack=primary

# Push with specific version tag
nullstone push --app=proton --source=proton:latest --version=v1.2.3 --stack=primary
```

### 5. Launch (Push + Deploy + Wait)

Combined operation that pushes, deploys, and waits for healthy status:

```bash
nullstone launch --app=proton --env=staging --source=proton:latest --stack=primary
```

### 6. Execute Commands in Container

SSH or run commands inside a running container:

```bash
# Open shell
nullstone exec --app=proton --env=staging --stack=primary

# Run specific command
nullstone exec --app=proton --env=staging --stack=primary -- rails console

# Run database migrations
nullstone exec --app=proton --env=staging --stack=primary -- rails db:migrate
```

### 7. List Resources

```bash
# List all apps in organization
nullstone apps list

# List all stacks
nullstone stacks list

# List environments in a stack
nullstone envs list --stack=primary
```

## Deployment Workflows

### Standard Deployment (CI/CD)

The typical CI/CD workflow used with GitHub Actions:

```yaml
env:
  NULLSTONE_ORG: iou-financial
  NULLSTONE_API_KEY: ${{ secrets.NULLSTONE_API_KEY }}
  NULLSTONE_STACK: primary

steps:
  - uses: nullstone-io/setup-nullstone-action@v0

  - name: Build Docker Image
    run: docker build -t myapp:latest .

  - name: Push and Deploy
    run: |
      nullstone push --app=myapp --source=myapp:latest
      nullstone deploy --app=myapp --env=${{ env.NULLSTONE_ENV }}
```

### Manual Deployment

For manual deployments from local machine:

```bash
# 1. Build your Docker image
docker build -t myapp:latest .

# 2. Push to Nullstone
nullstone push --app=myapp --source=myapp:latest --stack=primary

# 3. Deploy to staging
nullstone deploy --app=myapp --env=staging --stack=primary -w

# 4. Verify deployment
nullstone status --app=myapp --env=staging --stack=primary

# 5. Check logs
nullstone logs --app=myapp --env=staging --stack=primary -t
```

## GitOps Configuration

Nullstone uses `.nullstone/` directory for GitOps configuration:

```
.nullstone/
  config.yml      # Base configuration (apps, capabilities, connections)
  staging.yml     # Staging environment overrides
  production.yml  # Production environment overrides
  previews.yml    # Preview environment configuration
```

### Example config.yml

```yaml
version: "0.1"

apps:
  myapp:
    module: nullstone/aws-fargate-service
    vars:
      num_tasks: 1
      cpu: 512
      memory: 1024
      port: 3000
    connections:
      cluster-namespace: namespace0
    capabilities:
      load-balancer:
        module: nullstone/aws-load-balancer
        connections:
          subdomain: app-subdomain
      postgres:
        module: nullstone/aws-postgres-access
        connections:
          postgres: postgres0
      redis:
        module: nullstone/aws-redis-access
        connections:
          redis: redis0
    environment:
      RAILS_ENV: "{{ NULLSTONE_ENV }}"
      RAILS_MASTER_KEY: "{{ secret(arn:aws:secretsmanager:...) }}"
```

### Environment-specific overrides (staging.yml)

```yaml
version: "0.1"

apps:
  myapp:
    vars:
      num_tasks: 1
    environment:
      DEBUG: "true"
```

### Production overrides (production.yml)

```yaml
version: "0.1"

apps:
  myapp:
    vars:
      num_tasks: 3
      cpu: 1024
      memory: 2048
    environment:
      DEBUG: "false"
```

## Environment Variables in Nullstone

Nullstone supports several variable interpolation patterns:

| Pattern | Description |
|---------|-------------|
| `{{ NULLSTONE_ENV }}` | Current environment name (staging, production) |
| `{{ NULLSTONE_APP }}` | Current application name |
| `{{ NULLSTONE_STACK }}` | Current stack name |
| `{{ secret(arn:...) }}` | AWS Secrets Manager secret |
| `{{ output(block.output_name) }}` | Output from another block |

## Common Capabilities

| Capability | Module | Purpose |
|------------|--------|---------|
| Load Balancer | `nullstone/aws-load-balancer` | Public HTTP/HTTPS access |
| PostgreSQL | `nullstone/aws-postgres-access` | PostgreSQL database connection |
| Redis | `nullstone/aws-redis-access` | Redis cache connection |
| MySQL | `nullstone/aws-mysql-access` | MySQL database connection |
| S3 | `nullstone/aws-s3-access` | S3 bucket access |
| Autoscaling | `nullstone/aws-ecs-autoscaling-cpu` | CPU-based autoscaling |
| Rails Cookies | `nullstone/rails-cookies` | Rails session configuration |

## Troubleshooting

### Check if CLI is configured
```bash
nullstone profile
```

### View detailed deployment logs
```bash
nullstone logs --app=myapp --env=staging --stack=primary -s 30m -t
```

### Check infrastructure status
```bash
nullstone outputs --app=myapp --env=staging --stack=primary
```

### Force re-deploy
```bash
nullstone deploy --app=myapp --env=staging --stack=primary --version=$(git rev-parse HEAD) -w
```

## Links

- [Nullstone Documentation](https://docs.nullstone.io/)
- [GitHub Actions Integration](https://docs.nullstone.io/getting-started/deployment/ci-cd/github.html)
- [Nullstone Modules Registry](https://registry.nullstone.io/)
