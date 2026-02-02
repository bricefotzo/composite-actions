# CLAUDE.md - AI Assistant Guide

This document provides comprehensive guidance for AI assistants working with this repository.

## Project Overview

This is a **GitHub Actions Composite Actions** repository that demonstrates building, pushing, and testing Docker container images to GitHub Container Registry (ghcr.io). It includes a sample Python FastAPI application used for testing the CI/CD pipeline.

**Repository:** `bricefotzo/composite-actions`
**License:** GNU General Public License v3

## Repository Structure

```
composite-actions/
├── .github/
│   ├── actions/
│   │   └── build-and-push-images/
│   │       └── action.yml           # Reusable composite action for Docker builds
│   └── workflows/
│       └── main.yml                 # Main CI/CD workflow
├── app/
│   ├── __init__.py                  # Python package marker
│   ├── main.py                      # FastAPI application with embedded tests
│   ├── requirements.txt             # Python dependencies
│   ├── Dockerfile                   # Multi-stage Docker build
│   └── .dockerignore                # Docker build exclusions
├── .gitignore                       # Git exclusions
├── LICENSE                          # GNU GPL v3
└── CLAUDE.md                        # This file
```

## Key Components

### 1. Composite Action (`/.github/actions/build-and-push-images/action.yml`)

A reusable GitHub Action for building and pushing Docker images to container registries.

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `context` | No | `.` | Docker build context path |
| `image-tag` | Yes | - | Full image name and tag |
| `path` | No | `Dockerfile` | Path to Dockerfile |
| `registry` | No | `europe-west1-docker.pkg.dev` | Container registry URL |
| `registry-username` | No | `_json_key` | Registry authentication username |
| `registry-password` | Yes | - | Registry authentication password/token |

**Outputs:**
| Output | Description |
|--------|-------------|
| `image_tags` | Tags of the built image |

**Internal Steps:**
1. Setup Docker Buildx
2. Login to container registry
3. Extract metadata and tags
4. Build and push with GitHub Actions cache
5. Export image tags output

### 2. Main Workflow (`/.github/workflows/main.yml`)

Two-stage CI/CD pipeline triggered on push events and manual dispatch.

**Jobs:**
- `build` - Builds and pushes Docker image to ghcr.io
- `test` - Pulls the image and runs pytest tests (depends on build)

**Required Secrets:**
- `GH_TOKEN` - GitHub token for ghcr.io authentication

### 3. Sample Application (`/app/`)

A minimal FastAPI application for demonstration purposes.

**Tech Stack:**
- Python 3.10.10
- FastAPI (web framework)
- Uvicorn (ASGI server)
- pytest + httpx (testing)

**API Endpoint:**
- `GET /` - Returns `{"msg": "Hello World"}`

## Development Commands

### Local Development

```bash
# Install dependencies
pip install -r app/requirements.txt

# Run the application locally
cd app && uvicorn main:app --host=0.0.0.0 --port=8000

# Run tests locally
pytest app/main.py
```

### Docker Development

```bash
# Build Docker image
docker build -t composite-actions-app ./app

# Run container
docker run -p 8000:8000 composite-actions-app

# Run tests in container
docker run -v ./app:/app composite-actions-app pytest main.py
```

### Container Registry Operations

```bash
# Login to GitHub Container Registry
echo "$GH_TOKEN" | docker login ghcr.io -u USERNAME --password-stdin

# Push image
docker push ghcr.io/bricefotzo/composite-actions:tag
```

## Code Conventions

### Python

- **Framework:** FastAPI for web applications
- **Testing:** pytest with inline tests in the same file for simple apps
- **Test pattern:** Use `TestClient` from FastAPI for HTTP testing
- **Dependencies:** Listed in `requirements.txt`

### Docker

- **Base image:** `python:3.10.10-slim` for Python applications
- **Security:** Always use non-root users (see `appuser` in Dockerfile)
- **Optimization:** Use cache mounts for pip and bind mounts for requirements
- **Port:** Applications expose port 8000 by default

### GitHub Actions

- **Composite actions:** Store in `.github/actions/<action-name>/action.yml`
- **Workflows:** Store in `.github/workflows/`
- **Caching:** Use GitHub Actions cache for Docker layers
- **Authentication:** Use `--password-stdin` for secure registry login

## Workflow Patterns

### Adding a New Composite Action

1. Create directory: `.github/actions/<action-name>/`
2. Create `action.yml` with:
   - `name` and `description`
   - `inputs` with required/optional parameters
   - `outputs` for return values
   - `runs.using: composite` with steps

### Modifying the Application

1. Update code in `app/main.py`
2. Add tests in the same file using `test_*` naming
3. Update `requirements.txt` if adding dependencies
4. Test locally with `pytest app/main.py`
5. Commit and push to trigger CI/CD

### Testing Workflow Changes

1. Make changes to workflow files
2. Push to a branch to trigger the workflow
3. Check GitHub Actions tab for results
4. Debug using workflow run logs

## Important File Paths

| Purpose | Path |
|---------|------|
| Main workflow | `.github/workflows/main.yml` |
| Build composite action | `.github/actions/build-and-push-images/action.yml` |
| Application code | `app/main.py` |
| Application Dockerfile | `app/Dockerfile` |
| Python dependencies | `app/requirements.txt` |

## Common Tasks for AI Assistants

### When modifying the FastAPI application:
1. Read `app/main.py` first
2. Update the endpoint logic
3. Add/update corresponding test functions
4. Verify with `pytest app/main.py`

### When modifying the composite action:
1. Read `.github/actions/build-and-push-images/action.yml`
2. Update inputs/outputs as needed
3. Modify steps while maintaining Docker action versions
4. Update the main workflow if input/output signatures change

### When modifying the CI/CD workflow:
1. Read `.github/workflows/main.yml`
2. Understand job dependencies (`needs` field)
3. Use proper GitHub Actions context variables
4. Test by pushing to a feature branch

### When adding new dependencies:
1. Add to `app/requirements.txt`
2. Rebuild Docker image to verify installation
3. Update Dockerfile if special installation steps needed

## Troubleshooting

### Common Issues

1. **Docker build context errors:** Ensure `context` points to directory containing Dockerfile
2. **Registry authentication failures:** Use `--password-stdin` for secure token handling
3. **Volume mount issues:** Use `-v ./local:/container` syntax with proper paths
4. **Test failures in CI:** Ensure working directory is correct (`/app` in container)

### Debugging Tips

- Add `ls` commands in workflows to verify file locations
- Use `docker run ... ls /app` to check container file structure
- Check GitHub Actions logs for detailed error messages
- Verify secrets are properly configured in repository settings
