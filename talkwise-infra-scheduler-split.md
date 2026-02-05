# talkwise-infra / talkwise-scheduler Split

## Executive Summary

We've extracted scheduler-specific code from `talkwise-infra` into a new dedicated repository `talkwise-scheduler`. This separates concerns and follows evolutionary architecture principles - the longer these stay coupled, the harder they become to split.

## What Changed

### talkwise-infra (generic infrastructure modules)

**Removed:**
- `modules/scheduler-api/` - Runtime API for dynamic schedules
- `projects/ruk-scheduler/` - Ruk's scheduled messaging
- `projects/practice-interviews/` - PI scheduled jobs

**Kept:**
- All generic modules: `lambda/`, `scheduler/`, `iam/`, `ssm/`, `networking/`, `ecs/`, `alb/`, `ecr/`, `secrets/`, `codedeploy/`, `waf/`
- `projects/agent-factory/` - ECS-based service (uses only generic modules)

### talkwise-scheduler (new repo)

**Added:**
- `modules/scheduler-api/` - Scheduler-specific runtime API
- `projects/ruk-scheduler/` - Ruk scheduled tasks
- `projects/practice-interviews/` - PI scheduled tasks

## Architecture Decision: Module Sourcing

### Decision: Git source with tags

```hcl
module "lambda" {
  source = "git::https://github.com/FractalLabsDev/talkwise-infra.git//modules/lambda?ref=v1.0.0"
}
```

### Alternatives Considered

| Approach | Pros | Cons |
|----------|------|------|
| **Git with tags** (chosen) | Simple, no extra infra, version pinning, works with private repos | Requires creating tags manually |
| **Terraform Registry** | Semantic versioning, discovery, docs | Requires registry setup, publishing pipeline, overkill for internal modules |
| **Git submodules** | Single checkout | Complex workflows, version conflicts, git expertise required |
| **Monorepo** | Everything together | Defeats purpose of separation |
| **Copy modules locally** | No external deps | Drift, no updates, duplication |

**Why git+tags?** It's the simplest approach that provides version pinning without requiring additional infrastructure. When you want to update, change `ref=v1.0.0` to `ref=v1.1.0` and run `terraform init -upgrade`.

## Architecture Decision: State Management

### Decision: Same S3 bucket, different key paths

```
Bucket: infra-platform-tfstate-427479925030

Old keys (in talkwise-infra):
├── infra-platform/ruk-scheduler/production/terraform.tfstate
├── infra-platform/practice-interviews/production/terraform.tfstate

New keys (in talkwise-scheduler):
├── talkwise-scheduler/ruk-scheduler/production/terraform.tfstate
├── talkwise-scheduler/practice-interviews/production/terraform.tfstate
```

### Alternatives Considered

| Approach | Pros | Cons |
|----------|------|------|
| **Same bucket, new keys** (chosen) | Zero infra changes, Terraform handles migration | Keys need updating |
| **New S3 bucket** | Complete isolation | Extra bucket to manage, IAM updates needed |
| **Keep same keys** | No migration | Confusing, both repos point to same state |

**Why same bucket?** Terraform can migrate state automatically with `terraform init -migrate-state`. No manual state surgery or new infrastructure required.

## Migration Steps (Post-Merge)

### Order of operations:

1. **Merge talkwise-scheduler PR first** (adds the code)
2. **Merge talkwise-infra PR** (removes the code)
3. **Migrate state** in talkwise-scheduler:

```bash
# In talkwise-scheduler repo
cd projects/ruk-scheduler
terraform init -backend-config=backend-production.hcl -migrate-state
# Terraform will prompt: "Do you want to copy existing state?" → yes

cd ../practice-interviews
terraform init -backend-config=backend-production.hcl -migrate-state
# Same prompt → yes

# Verify no changes
terraform plan -var-file=production.tfvars
# Should show: "No changes. Your infrastructure matches the configuration."
```

### What happens during migration:

1. Terraform detects state exists at old key path
2. Asks if you want to copy to new key path
3. Copies state (doesn't delete old)
4. Future applies use new key

### Rollback (if needed):

Old state remains in S3 at original key. To rollback:
1. Revert PRs
2. State still exists at old paths

## Pull Requests

| Repo | PR | Description |
|------|-----|-------------|
| talkwise-scheduler | [PR #1](https://github.com/FractalLabsDev/talkwise-scheduler/pull/1) | Add scheduler code from infra |
| talkwise-infra | [PR #10](https://github.com/FractalLabsDev/talkwise-infra/pull/10) | Remove scheduler code |

## Version Tag

Created `v1.0.0` tag on talkwise-infra for module references. All scheduler projects now reference this tag.

## Questions for Discussion

1. **Module versioning strategy**: Should we automate tag creation on merge to main?
2. **Terraform Registry**: Worth setting up for discoverability, or git+tags sufficient?
3. **agent-factory**: Should this eventually move to its own repo too? (It's different from scheduler pattern - ECS vs Lambda)

## Contact

Questions? Reach out to @austin or @ruk on Slack.
