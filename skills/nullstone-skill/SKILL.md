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

---

# IOU Financial — Real-World Nullstone Configurations

Everything below is from the actual IOU Financial repos and admin runbooks. Use these as reference when setting up new stacks, debugging, or onboarding.

## Organization & Stack Info

- **Org:** `iou-financial`
- **Nullstone UI:** https://app.nullstone.io/orgs/iou-financial
- **Global Stack (domains):** https://app.nullstone.io/orgs/iou-financial/stacks/2574/overview
- **Primary Stack:** https://app.nullstone.io/orgs/iou-financial/stacks/3048/overview
- **Domain registrar:** name.com (Bryce Beyler manages)
- **Confluence:** https://ioufinancial.atlassian.net/wiki/spaces/IE/pages/4196040707/Nullstone+Stack+Setup+Documentation

## New Stack Setup (Step-by-Step)

### 1. Register a New Domain
Add to the global stack:
```bash
name: ioufinancial.dev
module: nullstone/aws-domain
```
Nullstone generates NS records. Share with Bryce to add to name.com registrar (remove old ns1-4.nam.com). Click "Verify" in Nullstone UI once propagated.

### 2. Create Network
```bash
name: network0
module: nullstone/aws-network
```
**Critical:** CIDR ranges must not overlap with existing stacks.

Existing CIDRs:
- Staging: `10.10.0.0/16` (private: 10.10.1-3.0/24, public: 10.10.101-103.0/24)
- Production: `10.11.0.0/16` (private: 10.11.1-3.0/24, public: 10.11.101-103.0/24)

### 3. Create ECS Cluster
```bash
name: cluster0
module: nullstone/aws-fargate
connection: network0
```

### 4. Create Namespace
```bash
name: namespace0
module: nullstone/aws-fargate-namespace
connection: cluster0
```

### 5. Create Datastores
```bash
# PostgreSQL
name: postgres0
module: nullstone/aws-rds-postgres
connection: network0

# Aurora MySQL
name: mysql0
module: nullstone/aws-aurora-mysql
connection: network0

# Redis
name: redis0
module: nullstone/aws-elasticache
connection: network0

# S3
name: s30
module: nullstone/aws-s3-bucket

# SendGrid
name: sendgrid0
module: nullstone/aws-sendgrid-domain
```

### 6. GitHub CI/CD Connection
```bash
# Create API token at https://app.nullstone.io/profile
nullstone configure --api-key=<TOKEN>

# Add to GitHub repo secrets: NULLSTONE_API_KEY
# Ensure .github/workflows references this secret
```

### 7. Rails Master Key
```bash
# Add as env var or secret in Nullstone config
RAILS_MASTER_KEY: "{{ secret(arn:aws:secretsmanager:...) }}"
```

### 8. Test & Plan
```bash
nullstone iac test --stack=<stack> --env=staging
nullstone iac test --stack=<stack> --env=production
nullstone plan --stack=<stack> --block=<app> --env=staging --wait
```

### 9. Subdomain DNS Flow
When creating a new subdomain, add NS records to the root account hosted zone:
```
Client → ioufinancial.com
  → NS delegation for nexus.staging.ioufinancial.com
    → Route53 hosted zone → ALB → ECS

Record name: nexus.staging.ioufinancial.com
Record type: NS
TTL: 300
Values:
  ns-380.awsdns-47.com.
  ns-941.awsdns-53.net.
  ns-1638.awsdns-12.co.uk.
  ns-1380.awsdns-44.org.
```

---

## IOU Repo .nullstone Configurations (Verbatim)

### iou-infrastructure (Core Infra)

**config.yml:**
```yaml
version: "0.1"

apps:
  bastion:
    module: nullstone/aws-bastion
    connections:
      network: network0
    vars:
      instance_type: "t3.nano"
    capabilities:
      postgres:
        module: nullstone/aws-postgres-access
        connections:
          postgres: postgres0
      mysql:
        module: nullstone/aws-mysql-access
        connections:
          mysql: mysql0
      zoltar0:
        module: nullstone/aws-mysql-access
        connections:
          mysql: zoltar0

datastores:
  sendgrid0:
    module: nullstone/aws-sendgrid-domain
    vars:
      domain: "ioucentral.com"
  redis0:
    module: nullstone/aws-elasticache
    vars:
      redis_version: "6.x"
      node_type: "cache.t3.small"
      enforce_ssl: true
    connections:
      network: network0
  coverband0:
    module: nullstone/aws-elasticache
    vars:
      redis_version: "6.x"
      node_type: "cache.t3.small"
      enforce_ssl: true
    connections:
      network: network0
  s30:
    module: nullstone/aws-s3-bucket
    vars:
      public_read_only: false
      server_side_encryption: true
      versioning: true
  postgres0:
    module: nullstone/aws-rds-postgres
    vars:
      postgres_version: "14"
      backup_retention_period: 1
      instance_class: "db.t3.medium"
      high_availability: false
      enable_public_access: false
      enforce_ssl: true
      allocated_storage: 500
    connections:
      network: network0
  mysql0:
    module: nullstone/aws-aurora-mysql
    vars:
      mysql_version: "8.0"
      backup_retention_period: 1
      instance_class: "db.r8g.large"
      instance_count: 1
      enable_performance_insights: false
      custom_mysql_params:
        binlog_checksum: "NONE"
        binlog_row_image: "full"
        binlog_backup: "1"
        binlog_format: "ROW"
        binlog_replication_globaldb: "1"
    connections:
      network: network0
  zoltar0:
    module: nullstone/aws-aurora-mysql
    vars:
      mysql_version: "8.0"
      backup_retention_period: 1
      instance_class: "db.r8g.2xlarge"
      instance_count: 1
      enable_performance_insights: false
      custom_mysql_params:
        binlog_checksum: "NONE"
        binlog_row_image: "full"
        binlog_backup: "1"
        binlog_format: "ROW"
        binlog_replication_globaldb: "1"
    connections:
      network: network0
  appsignal0:
    module: nullstone/aws-appsignal
  newrelic0:
    module: nullstone/aws-newrelic

cluster_namespaces:
  namespace0:
    module: nullstone/aws-fargate-namespace
    connections:
      cluster: cluster0

clusters:
  cluster0:
    module: nullstone/aws-fargate
    connections:
      network: network0

networks:
  network0:
    module: nullstone/aws-network
```

**production.yml (infra overrides):**
```yaml
version: "0.1"
datastores:
  redis0:
    vars:
      node_type: "cache.t3.small"
  postgres0:
    vars:
      backup_retention_period: 14
      instance_class: "db.r8g.large"
      high_availability: true
  mysql0:
    vars:
      backup_retention_period: 14
      enable_performance_insights: true
      instance_class: "db.r8g.2xlarge"
      instance_count: 1
  zoltar0:
    vars:
      backup_retention_period: 14
      enable_performance_insights: true
      instance_class: "db.r8g.2xlarge"
      instance_count: 1
networks:
  network0:
    vars:
      cidr: "10.11.0.0/16"
      private_subnets: ["10.11.1.0/24", "10.11.2.0/24", "10.11.3.0/24"]
      public_subnets: ["10.11.101.0/24", "10.11.102.0/24", "10.11.103.0/24"]
```

**staging.yml (infra overrides):**
```yaml
version: "0.1"
networks:
  network0:
    vars:
      cidr: "10.10.0.0/16"
      private_subnets: ["10.10.1.0/24", "10.10.2.0/24", "10.10.3.0/24"]
      public_subnets: ["10.10.101.0/24", "10.10.102.0/24", "10.10.103.0/24"]
```

#### Custom Modules (iou-infrastructure)

**cross-account-efs-attachment** (capability):
```yaml
org_name: iou-financial
name: cross-account-efs-attachment
description: Attaches EFS in another AWS account to an ECS/Fargate app
category: capability
subcategory: datastores
provider_types: [aws]
platform: efs
```
Publish: `cd tf/old-efs-attachment && nullstone modules publish --version=next-patch`

**zoltar-replication** (block):
```yaml
org_name: iou-financial
name: zoltar-replication
description: Configures replication of nucleus database into zoltar database
category: block
provider_types: [aws]
platform: dms
```
Publish: `cd zoltar-replication && nullstone modules publish --version=next-patch`

---

### fireball

**config.yml:**
```yaml
version: "0.1"

subdomains:
  partners-subdomain:
    dns_name: "partners"
    module: nullstone/aws-subdomain-manual
    vars:
      create_ssl_certificate: true
    connections:
      domain: global.global.ioufinancial-com-domain

apps:
  fireball:
    module: nullstone/aws-fargate-service
    connections:
      cluster-namespace: namespace0
    vars:
      num_tasks: 1
      cpu: 1024
      ephemeral_storage: 40
      health_check_grace_period: 60
      memory: 2048
      port: 9000
    capabilities:
      load-balancer:
        module: nullstone/aws-load-balancer
        vars:
          health_check_path: /healthcheck
        connections:
          subdomain: partners-subdomain
      nginx-sidecar:
        module: nullstone/aws-fargate-nginx-sidecar
      aws-ecs-autoscaling-cpu:
        module: nullstone/aws-ecs-autoscaling-cpu
      rails-cookies:
        module: nullstone/rails-cookies
      postgres:
        module: nullstone/aws-postgres-access
        connections:
          postgres: postgres0
      s3:
        module: nullstone/aws-s3-access
        connections:
          s3_bucket: old-fireball-s3
      sendgrid:
        module: nullstone/aws-sendgrid-access
        connections:
          sendgrid: sendgrid0
      appsignal:
        module: nullstone/aws-ecs-appsignal
        connections:
          appsignal: appsignal0
    environment:
      RAILS_ENV: "{{ NULLSTONE_ENV }}"
      RAILS_MASTER_KEY: "{{ secret(...) }}"
      S3_BUCKET: "{{ S3_BUCKET_NAME }}"
      NUCLEUS_URL: "https://app.{{ NULLSTONE_ENV }}.ioufinancial.com"
      ZOLTAR_URL: "https://reporting.{{ NULLSTONE_ENV }}.ioufinancial.com"

  fireball-worker:
    module: nullstone/aws-fargate-service
    vars:
      num_tasks: 1
      cpu: 1024
      memory: 2048
      port: 0
    capabilities:
      postgres, s3, sendgrid, appsignal, autoscaling (same connections)

datastores:
  old-fireball-s3:
    module: nullstone/aws-existing-s3-bucket
    vars:
      bucket_name: iou-fireball-development
  fireball-s3:
    module: nullstone/aws-s3-bucket
```

**production.yml:** num_tasks: 2, vanity subdomain, production secrets ARNs, BLAZER_DATABASE_URL
**staging.yml:** ZOLTAR_URL override (reporting-staging), BLAZER_DATABASE_URL
**previews.yml:** database_name: "fireball-{{ NULLSTONE_ENV }}", s3 → fireball-s3

---

### iou-dev (Nucleus)

**config.yml:**
```yaml
version: "0.1"

subdomains:
  app-subdomain:
    dns_name: "app"
    module: nullstone/aws-subdomain-manual
    connections:
      domain: global.global.ioufinancial-com-domain

apps:
  nucleus:
    module: nullstone/aws-fargate-service
    vars:
      num_tasks: 1
      cpu: 2048
      memory: 4096
      ephemeral_storage: 40
      port: 9000
      health_check_grace_period: 180
    capabilities:
      nginx-sidecar, rails-cookies, sendgrid, redis, redis-coverband (COVERBAND namespace),
      nucleus-load-balancer (health_check_path: /healthcheck, interval: 30, timeout: 29),
      mysql (database_name: nucleus, connection: mysql0),
      efs (mount_path: /mnt/efs/fs-backups),
      old-efs (iou-financial/cross-account-efs-attachment — cross-account EFS from old infra),
      appsignal, newrelic, s3, autoscaling (min_capacity: 2)
    environment:
      # YAML anchor &nucleus-environment shared with worker
      APP_DOMAIN: "app.{{ NULLSTONE_ENV }}.ioufinancial.com"
      RAILS_ENV, RAILS_MASTER_KEY, SLACK_API_TOKEN
      Experian (API + CrossCore), Dropbox Sign, Mailchimp, Paraxial, etc.

  nucleus-worker:
    module: nullstone/aws-fargate-service
    vars:
      num_tasks: 3
      cpu: 2048
      memory: 4096
      ephemeral_storage: 50
      port: 0
    capabilities:
      efs, old-efs, mysql, redis, redis-coverband, sendgrid, appsignal, newrelic, s3,
      autoscaling (min_capacity: 3)
    environment:
      <<: *nucleus-environment  # inherits all nucleus env vars
      COVERBAND_DISABLE_AUTO_START: "1"
```

**production.yml:** num_tasks: 2 (app) / 3 (worker), cpu: 4096, memory: 8192, old-efs enabled with create_resolver_rule, mysql → noop-capability (manually connecting to old MySQL in root account), OLD_MYSQL_* env vars, USE_OLD_DATASTORES: "true"
**previews.yml:** database_name: "nucleus-{{ NULLSTONE_ENV }}", s3-nightly-dump for DB_DUMP_FILE

---

### proton

**config.yml:**
```yaml
subdomains:
  staff-subdomain:
    dns_name: "staff"
    module: nullstone/aws-subdomain-manual

apps:
  proton:
    module: nullstone/aws-fargate-service
    vars: {num_tasks: 1, cpu: 512, memory: 1024, port: 9000}
    capabilities: load-balancer (staff-subdomain), nginx-sidecar, autoscaling,
      rails-cookies, postgres, redis, mysql (nucleus db), appsignal, s3
    environment:
      NUCLEUS_URL: "https://app.{{ NULLSTONE_ENV }}.ioufinancial.com"
      ZOLTAR_URL: "https://reporting.{{ NULLSTONE_ENV }}.ioufinancial.com"

  proton-worker:
    vars: {num_tasks: 1, cpu: 256, memory: 512, port: 0}
    capabilities: postgres (database_name: proton), redis, mysql (nucleus), appsignal, s3, autoscaling
```

**production.yml:** OLD_MYSQL_* env vars (connects to root account MySQL), worker cpu: 512/memory: 1024
**staging.yml:** ZOLTAR_URL override (reporting-staging)

---

### nexus

**config.yml:**
```yaml
subdomains:
  nexus-subdomain:
    dns_name: "nexus"
    module: nullstone/aws-subdomain-manual

apps:
  nexus-app:
    module: nullstone/aws-fargate-service
    vars: {num_tasks: 2, cpu: 2048, memory: 4096, port: 3000}
    capabilities: load-balancer (/up), nginx-sidecar,
      postgres (database_name: "nexus_{{ NULLSTONE_ENV }}"),
      mysql (database_name: nucleus — reads from nucleus DB)
    environment:
      Active Record encryption keys, JWT_SECRET, NEXUS_API_KEY
      WEB_CONCURRENCY: "2", MALLOC_ARENA_MAX: "2"

  nexus-worker:
    vars: {num_tasks: 1, cpu: 1024, memory: 2048, port: 0}
    command: ["bundle", "exec", "rails", "solid_queue:start"]
    environment: SOLID_QUEUE_PROCESSES: "3", SOLID_QUEUE_THREADS: "5"
    capabilities: postgres (nexus_{{ NULLSTONE_ENV }})
```

---

### iou-velocity-metrics

**config.yml:**
```yaml
apps:
  velocity-metrics:
    module: nullstone/aws-fargate-service
    vars: {num_tasks: 1, cpu: 512, memory: 1024, port: 3000}
    capabilities: load-balancer (subdomain: velocity, /up), nginx-sidecar,
      postgres (database_name: "velocity_metrics_{{ NULLSTONE_ENV }}"), rails-cookies
    environment: TZ: America/New_York

  velocity-metrics-worker:
    vars: {num_tasks: 1, cpu: 512, memory: 1024, port: 0}
    command: ["bundle", "exec", "rails", "solid_queue:start"]
    environment: SOLID_QUEUE_PROCESSES: "2", SOLID_QUEUE_THREADS: "3"
```

**staging.yml:** Nexus SSO config (NEXUS_URL, NEXUS_CLIENT_ID/SECRET)

---

### retail_loan_app

**config.yml:**
```yaml
subdomains:
  direct:
    dns_name: "direct"
    module: nullstone/aws-subdomain

apps:
  retail-loan-app:
    module: nullstone/aws-fargate-service
    vars: {num_tasks: 1, cpu: 2048, memory: 4096, port: 3000}
    capabilities: load-balancer (/healthcheck), nginx-sidecar, rails-cookies,
      postgres (retail-loan-app), sendgrid, s3 (retail-loan-s3)
    environment:
      # 3rd party integrations:
      Salesforce (CLIENT_ID/SECRET, USERNAME/PASSWORD/SECURITY_TOKEN)
      BaseLayer (API_KEY, API_URL)
      SmartyStreets (AUTH_ID/TOKEN)
      Plaid (CLIENT_ID, SECRET, ENVIRONMENT, WEBHOOK_URL)
      Lendflow (WEBHOOK_SECRET)
      Google OAuth (CLIENT_ID/SECRET)
      Turnstile (SITE_KEY/SECRET_KEY)
      SendGrid (API_KEY, DOMAIN)
      Nucleus API (URL + API_KEY)

  retail-loan-app-worker:
    vars: {num_tasks: 1, cpu: 1024, memory: 2048, port: 0}
    command: ["bundle", "exec", "rails", "solid_queue:start"]
    environment: SOLID_QUEUE_PROCESSES: "2", SOLID_QUEUE_THREADS: "3"
    # Inherits all secrets from main app

datastores:
  retail-loan-s3:
    module: nullstone/aws-existing-s3-bucket
    vars:
      bucket_name: retail-loan-app
```

---

### zoltar

**config.yml:**
```yaml
subdomains:
  reporting-subdomain:
    dns_name: "reporting"
    module: nullstone/aws-subdomain-manual

apps:
  zoltar:
    module: nullstone/aws-fargate-service
    vars: {num_tasks: 1, cpu: 512, memory: 1024, port: 9000}
    capabilities:
      zoltar-load-balancer (health_check_interval: 125, timeout: 120, idle_timeout: 120),
      nginx-sidecar, rails-cookies,
      mysql: nullstone/noop-capability (manually connects to old MySQL),
      redis, appsignal, autoscaling
    environment:
      # Dual MySQL connections (old infra):
      ZOLTAR_HOSTNAME, ZOLTAR_DATABASE: zoltar_prod
      IOU_DB_DATABASE: iou_prod_us
      MYSQL_HOST/PORT/DB/USER/PASSWORD
      # ETL job intervals:
      Q0-Q5_ETL_JOB_INTERVAL (10m-180m staging, 13m-83m production)
      AWS_S3_BUCKET: iou-zoltar-staging/production

  zoltar-etl:
    module: nullstone/aws-fargate-service
    vars: {num_tasks: 1, cpu: 2048, memory: 4096, port: 0}
    capabilities: mysql (noop), redis, appsignal, autoscaling
    # Same environment as zoltar app
```

**production.yml:** num_tasks: 2 (zoltar), 0 (etl — disabled), vanity subdomain, production MySQL hosts

---

## SSH Quick Reference (All Apps)

```bash
# Staging
nullstone ssh --app=nucleus --env=staging --stack=primary
nullstone ssh --app=nucleus-worker --env=staging --stack=primary
nullstone ssh --app=fireball --env=staging --stack=primary
nullstone ssh --app=fireball-worker --env=staging --stack=primary
nullstone ssh --app=proton --env=staging --stack=primary
nullstone ssh --app=proton-worker --env=staging --stack=primary
nullstone ssh --app=zoltar --env=staging --stack=primary
nullstone ssh --app=zoltar-etl --env=staging --stack=primary
nullstone ssh --app=nexus-app --env=staging --stack=primary
nullstone ssh --app=nexus-worker --env=staging --stack=primary
nullstone ssh --app=velocity-metrics --env=staging --stack=primary
nullstone ssh --app=retail-loan-app --env=staging --stack=primary

# Production
nullstone ssh --app=nucleus --env=production --stack=primary
nullstone ssh --app=nucleus-worker --env=production --stack=primary
nullstone ssh --app=fireball --env=production --stack=primary
nullstone ssh --app=fireball-worker --env=production --stack=primary
nullstone ssh --app=proton --env=production --stack=primary
nullstone ssh --app=proton-worker --env=production --stack=primary
nullstone ssh --app=zoltar --env=production --stack=primary
nullstone ssh --app=zoltar-etl --env=production --stack=primary
```

## Logs Quick Reference

```bash
nullstone logs -t -s '125m' --app=nucleus --env=staging --stack=primary
nullstone logs -t -s '125m' --app=nucleus-worker --env=staging --stack=primary
nullstone logs -t -s '125m' --app=fireball --env=staging --stack=primary
nullstone logs -t -s '125m' --app=proton --env=staging --stack=primary
nullstone logs -t -s '125m' --app=zoltar --env=staging --stack=primary
```

## Manual Deploy (Nucleus Example)

```bash
# Nucleus app — staging
docker build --build-arg COMMIT=$(git rev-parse HEAD) -t iou-dev -f .docker/Dockerfile . --platform=linux/amd64
nullstone push --app=nucleus --source=iou-dev --env=staging
nullstone deploy --app=nucleus --stack=primary --env=staging --version=<commit>

# Nucleus worker — staging
docker build --build-arg COMMIT=$(git rev-parse HEAD) -t iou-worker -f .docker/worker.Dockerfile . --platform=linux/amd64
nullstone push --app=nucleus-worker --source=iou-worker --env=staging
nullstone deploy --app=nucleus-worker --stack=primary --env=staging --version=<commit>
```

## Terraform State Management

```bash
# Select workspace
nullstone workspaces select --stack=primary --env=production --block=nucleus

# Pull state
terraform state pull > terraform_nuke_state_file.json

# Edit serial (+1) then push
terraform state push terraform_nuke_state_file.json

# Plan with var file
terraform plan -var-file=TF_VARS.json -target='aws_secretsmanager_secret_version.app_secret["SECRET_KEY_BASE"]'

# Console
terraform console
local.all_secrets["SECRET_KEY_BASE"]
```

## DMS Replication (Zoltar)

Zoltar uses AWS DMS to replicate nucleus tables. Custom module: `iou-financial/zoltar-replication`.

```bash
# Start replication task
aws dms start-replication-task \
  --replication-task-arn <ARN> \
  --start-replication-task-type resume-processing

# Check status
aws dms describe-replication-tasks \
  --filters Name=replication-task-id,Values=zoltar-replication-gveid \
  --query "ReplicationTasks[0].Status"

# Reload specific table
aws dms reload-tables \
  --replication-task-arn <ARN> \
  --tables-to-reload '[{"SchemaName":"nucleus","TableName":"collection_credit_reportings"}]'

# Verify
aws dms describe-table-statistics \
  --replication-task-arn <ARN> \
  --filters Name=schema-name,Values=nucleus Name=table-name,Values=collection_credit_reportings
```

Task IDs:
- Staging: `zoltar-replication-gveid` (account 566883448644)
- Production: `zoltar-replication-hueip` (account 956024584686)

## New Relic Integration

```yaml
# Staging
account_id: 2324150
api_key: <stored in AWS Secrets Manager>

# Production
account_id: 2952
api_key: <stored in AWS Secrets Manager>
```

Connector: `nullstone-production-connector`

## GitHub Actions — Manual Workflow Trigger

```bash
# Install GitHub CLI
brew install gh
gh auth login

# List workflows
gh workflow list

# Trigger manually
gh workflow run "Rails Test and Deploy" \
  --ref DEVOPS-229 \
  -f approver="Mahpara test"

# Check status
gh run list --workflow "Rails Test and Deploy"
```

To deploy from a specific branch, add to the workflow:
```yaml
on:
  push:
    branches: [main, develop, DEVOPS-231]
```

## Debugging Inside Containers

```bash
# Install tools (Alpine-based containers)
apk add --no-cache ca-certificates busybox-extras bind-tools netcat-openbsd openssh-client redis

# Test connectivity
telnet <host> <port>
nslookup <hostname>

# Rails console
RAILS_ENV=production bundle exec rails c

# Check env vars
ENV['OLD_MYSQL_HOST']
ENV['EFS_MOUNT_PATH']
ENV['USE_OLD_DATASTORES']

# Check mounts
mount -t nfs4
ls -la /mnt/efs/fs-backups/
ls -la /mnt/old-efs/fs-backups/
```

## AWS Account IDs

| Environment | Account ID |
|-------------|------------|
| Staging | 566883448644 |
| Production | 956024584686 |

## Patterns & Conventions

- **App + Worker pattern:** Every Rails app has a companion `-worker` service (port: 0, no LB)
- **Solid Queue workers:** `command: ["bundle", "exec", "rails", "solid_queue:start"]` (nexus, velocity-metrics, retail_loan_app)
- **Legacy workers:** Resque-based (zoltar ETL, cutover from EC2)
- **Secrets:** Always use `{{ secret(arn:aws:secretsmanager:...) }}` — never hardcode
- **YAML anchors:** Nucleus uses `&nucleus-environment` / `<<: *nucleus-environment` to share env between app and worker
- **noop-capability:** Use `nullstone/noop-capability` when manually managing a connection (e.g., old MySQL in root account)
- **Cross-account EFS:** Custom module `iou-financial/cross-account-efs-attachment` for mounting EFS from old infra
- **Subdomain types:** `aws-subdomain-manual` (most apps) vs `aws-subdomain` (retail_loan_app)
- **Health checks:** `/healthcheck` (nucleus, fireball, retail) or `/up` (nexus, velocity-metrics)
- **Production vanity:** `create_vanity: true` on subdomain for clean URLs (no .staging. prefix)
