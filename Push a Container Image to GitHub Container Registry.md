# Guide: Push a Container Image to GitHub Container Registry (ghcr.io) using Podman

---

## Table of Contents

1. [Generate a GitHub Personal Access Token (PAT)](#1-generate-a-github-personal-access-token-pat)
2. [Log in to ghcr.io with Podman](#2-log-in-to-ghcrio-with-podman)
3. [Build the Container Image](#3-build-the-container-image)
4. [Tag the Image for ghcr.io](#4-tag-the-image-for-ghcrio)
5. [Push the Image to ghcr.io](#5-push-the-image-to-ghcrio)
6. [Make the Package Public](#6-make-the-package-public)
7. [Verify the Public Image](#7-verify-the-public-image)
8. [Pull & Run the Public Image](#8-pull--run-the-public-image)
9. [Revoke / Rotate Token](#9-revoke--rotate-token)
10. [Troubleshooting](#10-troubleshooting)

---

## 1. Generate a GitHub Personal Access Token (PAT)

A PAT is required to authenticate with ghcr.io. GitHub does **NOT** accept your regular password for container registry operations.

### Steps

1. Go to: **https://github.com/settings/tokens**
2. Click **"Generate new token"** → **"Generate new token (classic)"**
3. Fill in:
   - **Note:** `podman-ghcr-push` (or any descriptive name)
   - **Expiration:** 30 days (or as needed)
   - **Scopes:** Check these boxes:
     - ✅ `write:packages` — push images
     - ✅ `read:packages` — pull images
     - ✅ `delete:packages` — optional, to delete old images
4. Click **"Generate token"**
5. **COPY the token immediately** (starts with `ghp_...`)

> ⚠️ You will **NOT** be able to see it again after leaving the page!

Save it temporarily in your terminal:

```bash
export GITHUB_TOKEN="ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

---

## 2. Log in to ghcr.io with Podman

### Option A: Using `--password-stdin` (Recommended — safer, no token in shell history)

```bash
echo "$GITHUB_TOKEN" | podman login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin
```

### Option B: Interactive prompt

```bash
podman login ghcr.io -u YOUR_GITHUB_USERNAME
# When prompted for "Password:", paste your PAT token
```

**Expected output:**

```
Login Succeeded!
```

**Verify login:**

```bash
podman login ghcr.io --get-login
# Should print: YOUR_GITHUB_USERNAME
```

---

## 3. Build the Container Image

Navigate to the project directory (must contain a `Dockerfile`):

```bash
cd /path/to/your/project
podman build -t my-app-name .
```

**Example:**

```bash
cd /Users/mohit.radadiya/Documents/blackrock/blk-hacking-ind-mohit-radadiya
podman build -t blk-hacking-ind-mohit-radadiya .
```

**Verify the image was built:**

```bash
podman images | grep my-app-name
```

---

## 4. Tag the Image for ghcr.io

**Format:**

```bash
podman tag LOCAL_IMAGE ghcr.io/GITHUB_USERNAME/IMAGE_NAME:TAG
```

**Example:**

```bash
podman tag localhost/blk-hacking-ind-mohit-radadiya:latest \
  ghcr.io/radadiyamohit81/blk-hacking-ind-mohit-radadiya:latest
```

You can also add version tags:

```bash
podman tag localhost/blk-hacking-ind-mohit-radadiya:latest \
  ghcr.io/radadiyamohit81/blk-hacking-ind-mohit-radadiya:1.0.0
```

---

## 5. Push the Image to ghcr.io

```bash
podman push ghcr.io/GITHUB_USERNAME/IMAGE_NAME:TAG
```

**Example:**

```bash
podman push ghcr.io/radadiyamohit81/blk-hacking-ind-mohit-radadiya:latest
```

**Expected output:**

```
Getting image source signatures
Copying blob sha256:...
Copying blob sha256:...
Copying config sha256:...
Writing manifest to image destination
```

**Push multiple tags:**

```bash
podman push ghcr.io/radadiyamohit81/blk-hacking-ind-mohit-radadiya:latest
podman push ghcr.io/radadiyamohit81/blk-hacking-ind-mohit-radadiya:1.0.0
```

---

## 6. Make the Package Public

> ⚠️ By default, GitHub packages are **PRIVATE**. You **MUST** change visibility manually for the image to be publicly pullable.

### Steps

1. Go to: **https://github.com/YOUR_USERNAME?tab=packages**
2. Click on the package name (e.g., `blk-hacking-ind-mohit-radadiya`)
3. Click **"Package settings"** (right sidebar)
4. Scroll down to **"Danger Zone"**
5. Click **"Change visibility"**
6. Select **"Public"**
7. Type the package name to confirm
8. Click **"I understand the consequences, change package visibility"**

---

## 7. Verify the Public Image

Log out first to test as an anonymous user:

```bash
podman logout ghcr.io
```

Pull the image (should work **without** authentication):

```bash
podman pull ghcr.io/radadiyamohit81/blk-hacking-ind-mohit-radadiya:latest
```

If this works, your image is public! ✅

Log back in for future pushes:

```bash
echo "$GITHUB_TOKEN" | podman login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin
```

---

## 8. Pull & Run the Public Image

Anyone can now run your container:

```bash
podman pull ghcr.io/radadiyamohit81/blk-hacking-ind-mohit-radadiya:latest
podman run -p 5477:5477 ghcr.io/radadiyamohit81/blk-hacking-ind-mohit-radadiya:latest
```

**Verify it's running:**

```bash
curl http://localhost:5477/blackrock/challenge/v1/performance
```

---

## 9. Revoke / Rotate Token

> ⚠️ **IMPORTANT:** After you're done pushing, revoke the token for security.

### Steps

1. Go to: **https://github.com/settings/tokens**
2. Find your token (e.g., `podman-ghcr-push`)
3. Click **"Delete"** → Confirm

Also clear it from your shell:

```bash
unset GITHUB_TOKEN
history -c   # Clear shell history (optional)
```

---

## 10. Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| `403 Forbidden` during login | PAT token doesn't have `write:packages` scope | Generate a new token with correct scopes (see Step 1) |
| `denied` or `unauthorized` during push | Not logged in or token expired | Run `podman login ghcr.io` or generate a new token |
| `name unknown` during pull | Package doesn't exist or is private | Push it first, or make it public (see Step 6) |
| `exit code 125` during pull | Package is private and you're not authenticated | Make the package public OR log in first |
| Image too large | Not using multi-stage build | Use multi-stage Dockerfile + Alpine-based images (e.g., `eclipse-temurin:21-jre-alpine`) |

---

## Quick Reference (Copy-Paste Commands)

```bash
# One-time setup
export GITHUB_TOKEN="ghp_your_token_here"
echo "$GITHUB_TOKEN" | podman login ghcr.io -u radadiyamohit81 --password-stdin

# Build, tag, push
podman build -t blk-hacking-ind-mohit-radadiya .
podman tag localhost/blk-hacking-ind-mohit-radadiya:latest \
  ghcr.io/radadiyamohit81/blk-hacking-ind-mohit-radadiya:latest
podman push ghcr.io/radadiyamohit81/blk-hacking-ind-mohit-radadiya:latest

# Then go to GitHub → Packages → Make it Public
```
