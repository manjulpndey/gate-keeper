# GATEKEEPER

GATEKEEPER is a small Flask service and a practical DevSecOps reference
implementation. Its GitHub Actions workflow makes security and quality checks
mandatory before a container image is published to GitHub Container Registry
(GHCR).

## What the pipeline enforces

Every pull request to `main` runs:

- unit tests with pytest;
- style checks with flake8;
- static application security testing with Bandit; and
- dependency vulnerability scanning with pip-audit.

On a successful push to `main`, it additionally builds and publishes a Docker
image, scans that image with Trivy, and uploads a deployment-package artifact.
The workflow stops if a required check fails or Trivy finds a HIGH or CRITICAL
vulnerability.

## Architecture

```text
Pull request / push
        |
        v
Tests + lint + SAST + dependency audit
        |
        v
Docker build and GHCR publish (main only)
        |
        v
Trivy image scan --> deployment artifact
```

## Run locally

Prerequisite: Python 3.13 or later.

```bash
python -m venv .venv
# Windows: .venv\\Scripts\\activate
# macOS/Linux: source .venv/bin/activate
python -m pip install -r requirements.txt
python app/app.py
```

The service listens on `http://127.0.0.1:5000` by default.

```bash
curl http://127.0.0.1:5000/
curl http://127.0.0.1:5000/health
```

Environment variables:

| Variable | Default | Purpose |
| --- | --- | --- |
| `FLASK_HOST` | `127.0.0.1` | Interface used by the service |
| `FLASK_PORT` | `5000` | HTTP port |
| `FLASK_DEBUG` | `false` | Enables Flask debug mode when `true` |

## Run with Docker

```bash
docker build -t gatekeeper .
docker run --rm -p 5000:5000 gatekeeper
```

The container sets `FLASK_HOST=0.0.0.0`, while the application remains
loopback-only by default when run directly.

## Quality and security checks

```bash
python -m pip install -r requirements.txt -r requirements-dev.txt
python -m pytest -v
python -m flake8 .
python -m bandit -r app
python -m pip_audit
```

## Container publishing

The workflow publishes to `ghcr.io/<repository-owner>/GATEKEEPER` using the
repository's built-in `GITHUB_TOKEN`. No registry username is embedded in the
workflow; the target is derived from the repository that runs it. If you want
the published package to be public, change its visibility from the package
settings on GitHub after the first successful workflow run.

## Repository layout

```text
app/                    Flask application
tests/                  Unit tests
.github/workflows/      CI/CD definition
Dockerfile              Container build
requirements*.txt       Runtime and development dependencies
```

## Security notes

Do not add credentials, API keys, or private configuration files to this
repository. Use GitHub Actions secrets for deployment credentials and keep
dependency updates current. The included scanners are quality gates, not a
substitute for code review or runtime security controls.

## License

No license has been selected yet. Add one before distributing or accepting
external contributions.
