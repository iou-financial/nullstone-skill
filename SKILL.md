# Nullstone IaC Skills - IOU Repos (masley-ns 2026-01-31)

Analyzed .nullstone/ in 10 repos + iou-admin-tasks notes.

## Repo .nullstone Summaries

### fireball (.nullstone/config.yaml)
```yaml
workspaces:
  - name: partners
    modules:
      - s3-buckets
    capabilities:
      - subdomain
```

### iou-dev
Complex Nucleus + efs/DB/redis/LB (full yaml: vpc/eks/nucleus-app...)

[Truncated - full in file]

## Commands (from iou-admin-tasks)
nullstone iac test/plan --stack iou-infrastructure --env dev
nullstone ssh --app nucleus-app --env staging

## Notes
Stack setup: domains/clusters/DBs/CI/rails key
Custom: zoltar-replication/DMS
Zoltar: mysql cutover resque→solid_queue
