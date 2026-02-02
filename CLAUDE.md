# CLAUDE.md - AI Assistant Guide

This document provides guidance for AI assistants working with this repository.

## Project Overview

This is a **library of reusable GitHub Actions** - a growing collection of composite actions designed to be shared and used across projects. The goal is to centralize useful, well-tested actions that solve common CI/CD challenges.

**Repository:** `bricefotzo/composite-actions`
**License:** GNU General Public License v3

## Repository Structure

```
composite-actions/
├── .github/
│   ├── actions/                     # Collection of reusable composite actions
│   │   └── build-and-push-images/   # Docker build & push action
│   │       └── action.yml
│   └── workflows/
│       └── main.yml                 # Workflow for testing actions
├── app/                             # Sample app for testing (not the main focus)
└── CLAUDE.md                        # This file
```

## Actions Catalog

### `build-and-push-images`
**Path:** `.github/actions/build-and-push-images/action.yml`
**Purpose:** Build and push Docker images to any container registry

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `context` | No | `.` | Docker build context path |
| `image-tag` | Yes | - | Full image name and tag |
| `path` | No | `Dockerfile` | Path to Dockerfile |
| `registry` | No | `europe-west1-docker.pkg.dev` | Container registry URL |
| `registry-username` | No | `_json_key` | Registry auth username |
| `registry-password` | Yes | - | Registry auth password/token |

**Output:** `image_tags` - Tags of the built image

---

*More actions coming soon...*

## How to Use These Actions

### From Another Repository

Reference actions directly from this repo:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: bricefotzo/composite-actions/.github/actions/build-and-push-images@main
        with:
          image-tag: ghcr.io/myorg/myapp:latest
          registry: ghcr.io
          registry-username: ${{ github.actor }}
          registry-password: ${{ secrets.GITHUB_TOKEN }}
```

### Pin to a Specific Version

For stability, pin to a commit SHA or tag:

```yaml
- uses: bricefotzo/composite-actions/.github/actions/build-and-push-images@v1.0.0
```

## Adding New Actions

### Directory Structure

Each action lives in its own directory under `.github/actions/`:

```
.github/actions/
├── build-and-push-images/
│   └── action.yml
├── deploy-to-kubernetes/      # Example future action
│   └── action.yml
├── run-tests-with-coverage/   # Example future action
│   └── action.yml
└── notify-slack/              # Example future action
    └── action.yml
```

### Action Template

Create a new action at `.github/actions/<action-name>/action.yml`:

```yaml
name: 'Action Name'
description: 'What this action does'

inputs:
  required-input:
    description: 'Description of this input'
    required: true
  optional-input:
    description: 'Description with default'
    required: false
    default: 'default-value'

outputs:
  result:
    description: 'What this output contains'
    value: ${{ steps.step-id.outputs.result }}

runs:
  using: 'composite'
  steps:
    - name: Step description
      shell: bash
      run: |
        echo "Action logic here"
        echo "result=value" >> $GITHUB_OUTPUT
```

### Best Practices for New Actions

1. **Clear naming:** Use descriptive kebab-case names (`deploy-to-s3`, `run-tests`)
2. **Document inputs/outputs:** Every input and output needs a description
3. **Sensible defaults:** Provide defaults where possible to reduce required config
4. **Idempotent:** Actions should be safe to run multiple times
5. **Minimal permissions:** Request only necessary permissions
6. **Error handling:** Fail fast with clear error messages
7. **Caching:** Use GitHub Actions cache when beneficial

### Testing New Actions

1. Create a test workflow in `.github/workflows/` that exercises the action
2. Use the `/app` sample application or create minimal test fixtures
3. Test on a feature branch before merging

## Common Tasks for AI Assistants

### When adding a new action:
1. Create directory: `.github/actions/<action-name>/`
2. Create `action.yml` following the template above
3. Add comprehensive input/output documentation
4. Create or update a test workflow
5. Update this CLAUDE.md to add the action to the catalog

### When modifying an existing action:
1. Read the current `action.yml` first
2. Maintain backward compatibility when possible
3. Update input/output documentation if signatures change
4. Test changes don't break existing workflows

### When a user wants to add a specific action:
1. Understand what problem the action solves
2. Research existing GitHub Actions that could be composed
3. Design clear inputs/outputs
4. Implement with error handling
5. Add to the catalog in this file

## Ideas for Future Actions

Common CI/CD tasks that could become actions:
- Kubernetes deployment
- Slack/Discord notifications
- Test coverage reporting
- Security scanning
- Release automation
- Environment provisioning
- Database migrations
- Cache management
- Artifact publishing

## Resources

- [Creating composite actions](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action)
- [GitHub Actions syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Actions marketplace](https://github.com/marketplace?type=actions) for inspiration
