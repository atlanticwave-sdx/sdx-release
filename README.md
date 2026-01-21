# AtlanticWave-SDX Release Repository

## Purpose

This repository is the system-level orchestration and release repository for the AtlanticWave-SDX platform.

It does not contain application code.

---

## Repository Structure

```
atlanticwave-sdx-root/
├── services/           # Git submodules
├── docker-compose.yml  # Platform orchestration
├── .gitmodules
└── README.md
```

---

## What This Repository Does

- Aggregates SDX repositories using Git submodules
- Defines system-wide deployment via Docker Compose
- Provides a single authoritative SDX release
- Enables reproducible deployments and rollbacks

---

## Cloning the Repository

Clone with submodules:

```bash
git clone --recurse-submodules https://github.com/atlanticwave-sdx/atlanticwave-sdx-root.git
```

If already cloned incorrectly:

```bash
git submodule update --init --recursive
```

---

## Adding a New SDX Component

If a new SDX repository is created:

```bash
git submodule add https://github.com/atlanticwave-sdx/<new-repo>.git services/<new-repo>
git add .gitmodules services/<new-repo>
git commit -m "Add <new-repo> as SDX component"
git push
```

Optionally update `docker-compose.yml`.

---

## Daily Development Workflow

### Work on a Component

```bash
cd services/sdx-controller
git checkout main
git pull
# work on component
git commit -am "Change description"
git push
```

### Update the Root Repository Pointer

```bash
cd ../../
git add services/sdx-controller
git commit -m "Update sdx-controller submodule"
git push
```

This step is REQUIRED.

---

## Creating a Unified SDX Release

From the root repository:

```bash
git tag v<RELEASE>
git push origin v<RELEASE>
```

This tag defines:

- One SDX platform version  
- Exact commits of all components  
- A reproducible system state  

---

## GHCR Release Workflow

To build and publish Docker images:

```bash
# Step 1: Identify workflow commit
git log -1 --pretty=oneline -- .github/workflows/publish-images.yml
# Record commit hash as WORKFLOW_COMMIT

# Step 2: Delete old temporary tag
git tag -d temp-release
git push origin :refs/tags/temp-release

# Step 3: Create temporary tag to trigger workflow
git tag temp-release <WORKFLOW_COMMIT>
git push origin temp-release

# Step 4: Monitor workflow in GitHub Actions
# Wait for all 4 jobs to succeed:
# publish-sdx-controller, publish-sdx-lc, publish-sdx-oxp-integrator, publish-kytos-sdx

# Step 5: Pull built images locally
docker pull ghcr.io/atlanticwave-sdx/sdx-controller:temp-release
docker pull ghcr.io/atlanticwave-sdx/sdx-lc:temp-release
docker pull ghcr.io/atlanticwave-sdx/sdx-oxp-integrator:temp-release
docker pull ghcr.io/atlanticwave-sdx/kytos-sdx:temp-release

# Step 6: Retag images for official release
docker tag ghcr.io/atlanticwave-sdx/sdx-controller:temp-release ghcr.io/atlanticwave-sdx/sdx-controller:v<RELEASE>
docker tag ghcr.io/atlanticwave-sdx/sdx-lc:temp-release ghcr.io/atlanticwave-sdx/sdx-lc:v<RELEASE>
docker tag ghcr.io/atlanticwave-sdx/sdx-oxp-integrator:temp-release ghcr.io/atlanticwave-sdx/sdx-oxp-integrator:v<RELEASE>
docker tag ghcr.io/atlanticwave-sdx/kytos-sdx:temp-release ghcr.io/atlanticwave-sdx/kytos-sdx:v<RELEASE>

# Step 7: Push official release images to GHCR
docker push ghcr.io/atlanticwave-sdx/sdx-controller:v<RELEASE>
docker push ghcr.io/atlanticwave-sdx/sdx-lc:v<RELEASE>
docker push ghcr.io/atlanticwave-sdx/sdx-oxp-integrator:v<RELEASE>
docker push ghcr.io/atlanticwave-sdx/kytos-sdx:v<RELEASE>

# Step 8: Optional cleanup
docker rmi ghcr.io/atlanticwave-sdx/sdx-controller:temp-release
docker rmi ghcr.io/atlanticwave-sdx/sdx-lc:temp-release
docker rmi ghcr.io/atlanticwave-sdx/sdx-oxp-integrator:temp-release
docker rmi ghcr.io/atlanticwave-sdx/kytos-sdx:temp-release

git tag -d temp-release
git push origin :refs/tags/temp-release
```

- Temporary tag is only for triggering the workflow, not for production use  
- Ensure the temporary tag points to a commit containing the workflow  
- Official release images must always be retagged with the proper release version

---

## Operator Instructions — Pulling Images

```bash
docker pull ghcr.io/atlanticwave-sdx/sdx-controller:v<RELEASE>
docker pull ghcr.io/atlanticwave-sdx/sdx-lc:v<RELEASE>
docker pull ghcr.io/atlanticwave-sdx/sdx-oxp-integrator:v<RELEASE>
docker pull ghcr.io/atlanticwave-sdx/kytos-sdx:v<RELEASE>
```

- Replace `<RELEASE>` with the release version, e.g., `2026.1.1`  
- Authentication may be required:

```bash
echo "<PAT>" | docker login ghcr.io --username <GITHUB_USERNAME> --password-stdin
```

---

## Verify Deployment (Optional)

```bash
docker compose build
docker compose up -d
docker compose ps
```

---

## Reversibility

- Uses Git submodules → nothing is overwritten  
- Existing repositories remain unchanged  
- Tags or branches are not modified automatically  

---

### Reversal Scenarios

#### Delete Everything
- Remove the root repository → full rollback

#### Remove a Component
```bash
git submodule deinit -f services/<repo>
git rm -f services/<repo>
rm -rf .git/modules/services/<repo>
git commit -m "Remove <repo>"
git push
```

#### Undo a Tag
```bash
git tag -d vX.Y.Z
git push origin :refs/tags/vX.Y.Z
```

---

## Final Note

The repository stores only:

- Repository URLs  
- Pointers to exact commits  

Everything else is preserved. Official releases and operator images remain accessible.

