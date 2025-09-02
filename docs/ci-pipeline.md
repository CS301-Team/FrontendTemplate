# CI Pipeline Documentation

This document provides detailed information about the Continuous Integration (CI) pipeline implemented for this project.

## Pipeline Overview

The CI pipeline is triggered on:

- All push events to any branch
- Pull requests to the main branch

The pipeline consists of three main jobs that run in parallel where possible:

1. `security-checks` - Runs security audits and code quality checks
2. `build` - Builds and packages the Docker image
3. `docker-security` - Scans the built Docker image for vulnerabilities

## Environment Variables

The pipeline uses the following environment variables:

- `IMAGE_NAME`: Name of the Docker image (default: `my-app`)
- `IMAGE_VERSION`: Version tag for the Docker image (default: `latest`)

## Jobs

### 1. Security Checks (`security-checks`)

This job runs various security and code quality checks.

**Runner**: `ubuntu-latest`
**Continue on Error**: Yes (failures won't stop the pipeline)

**Steps**:

1. **Checkout repository** - Uses `actions/checkout@v4` to checkout the code
2. **Run security audit** - Executes `npm audit --audit-level high` to check for high or critical vulnerabilities in dependencies
3. **Setup Node.js** - Uses `actions/setup-node@v4` with LTS Node.js version and npm cache
4. **Install dependencies** - Runs `npm ci` for clean installation of dependencies
5. **Run ESLint security checks** - Executes `npm run lint` to check code quality and potential security issues

### 2. Build (`build`)

This job builds the Docker image for the application.

**Runner**: `ubuntu-latest`

**Steps**:

1. **Checkout repository** - Uses `actions/checkout@v4` to checkout the code
2. **Set up Docker Buildx** - Uses `docker/setup-buildx-action@v3` to set up Docker Buildx
3. **Build Docker image** - Uses `docker/build-push-action@v5` to build the Docker image with:
   - Context: Current directory
   - Dockerfile: `./deploy/Dockerfile`
   - Push: Disabled (build only)
   - Load: Enabled (loads image into Docker daemon)
   - Tags: Uses `IMAGE_NAME` and `IMAGE_VERSION` environment variables
   - Caching: Uses GitHub Actions cache for faster builds
4. **Export Docker image** - Exports the built image as a gzipped tar file to `/tmp/docker-image.tar.gz`
5. **Upload Docker image as artifact** - Uses `actions/upload-artifact@v4` to upload the Docker image artifact for potential use in later jobs

### 3. Docker Security (`docker-security`)

This job scans the built Docker image for vulnerabilities using Trivy.

**Runner**: `ubuntu-latest`
**Needs**: `build` job completion
**Continue on Error**: Yes (failures won't stop the pipeline)
**Permissions**: Requires `security-events: write` to upload scan results

**Steps**:

1. **Checkout repository** - Uses `actions/checkout@v4` to checkout the code
2. **Run Trivy vulnerability scanner** - Uses `aquasecurity/trivy-action@master` to scan for vulnerabilities with:
   - Scan type: Filesystem scan
   - Ignore unfixed vulnerabilities: Enabled
   - Output format: SARIF
   - Output file: `trivy-results.sarif`
   - Severity levels: CRITICAL and HIGH only
3. **Upload Trivy scan results** - Uses `github/codeql-action/upload-sarif@v3` to upload results to GitHub Security tab

## Tools Used

- **npm audit**: Checks for security vulnerabilities in npm dependencies
- **ESLint**: JavaScript/TypeScript linting tool for code quality
- **Docker**: Containerization platform for building and packaging the application
- **Trivy**: Comprehensive security scanner for containers and dependencies
- **GitHub Actions**: CI/CD platform for automating the pipeline

## Caching Strategy

The pipeline implements caching to improve build performance:

- Docker layer caching using GitHub Actions cache
- npm dependency caching using the setup-node action

## Artifacts

The pipeline produces the following artifacts:

- Docker image tarball (`docker-image.tar.gz`) that can be downloaded from the workflow run

## Security Features

The pipeline includes multiple security layers:

- Dependency vulnerability scanning with npm audit
- Code quality and security checks with ESLint
- Container image scanning with Trivy
- Results integration with GitHub Security tab
