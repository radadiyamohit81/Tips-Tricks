================================================================================
  GUIDE: Push a Container Image to GitHub Container Registry (ghcr.io) 
  using Podman
================================================================================

TABLE OF CONTENTS
  1. Generate a GitHub Personal Access Token (PAT)
  2. Log in to ghcr.io with Podman
  3. Build the Container Image
  4. Tag the Image for ghcr.io
  5. Push the Image to ghcr.io
  6. Make the Package Public
  7. Verify the Public Image
  8. Pull & Run the Public Image
  9. Revoke / Rotate Token
  10. Troubleshooting

================================================================================
1. GENERATE A GITHUB PERSONAL ACCESS TOKEN (PAT)
================================================================================

A PAT is required to authenticate with ghcr.io. GitHub does NOT accept your
regular password for container registry operations.

Steps:
  a) Go to: https://github.com/settings/tokens
  b) Click "Generate new token" → "Generate new token (classic)"
  c) Fill in:
       - Note: "podman-ghcr-push" (or any descriptive name)
       - Expiration: 30 days (or as needed)
       - Scopes: Check these boxes:
           ✅ write:packages   (push images)
           ✅ read:packages    (pull images)
           ✅ delete:packages  (optional, to delete old images)
  d) Click "Generate token"
  e) COPY the token immediately (starts with "ghp_...")
     ⚠️  You will NOT be able to see it again after leaving the page!

Save it temporarily:
  export GITHUB_TOKEN="ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

================================================================================
2. LOG IN TO ghcr.io WITH PODMAN
================================================================================

Option A: Using --password-stdin (RECOMMENDED — safer, no token in shell history)

  echo "$GITHUB_TOKEN" | podman login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin

Option B: Interactive prompt

  podman login ghcr.io -u YOUR_GITHUB_USERNAME
  # When prompted for "Password:", paste your PAT token

Expected output:
  Login Succeeded!

Verify login:
  podman login ghcr.io --get-login
  # Should print: YOUR_GITHUB_USERNAME

================================================================================
3. BUILD THE CONTAINER IMAGE
================================================================================

Navigate to the project directory (must contain a Dockerfile):

  cd /path/to/your/project
  podman build -t my-app-name .

Example:
  cd /Users/mohit.radadiya/Documents/blackrock/blk-hacking-ind-mohit-radadiya
  podman build -t blk-hacking-ind-mohit-radadiya .

Verify the image was built:
  podman images | grep my-app-name

================================================================================
4. TAG THE IMAGE FOR ghcr.io
================================================================================

Format:
  podman tag LOCAL_IMAGE ghcr.io/GITHUB_USERNAME/IMAGE_NAME:TAG

Example:
  podman tag localhost/blk-hacking-ind-mohit-radadiya:latest \
    ghcr.io/radadiyamohit81/blk-hacking-ind-mohit-radadiya:latest

You can also add version tags:
  podman tag localhost/blk-hacking-ind-mohit-radadiya:latest \
    ghcr.io/radadiyamohit81/blk-hacking-ind-mohit-radadiya:1.0.0

================================================================================
5. PUSH THE IMAGE TO ghcr.io
================================================================================

  podman push ghcr.io/GITHUB_USERNAME/IMAGE_NAME:TAG

Example:
  podman push ghcr.io/radadiyamohit81/blk-hacking-ind-mohit-radadiya:latest

Expected output:
  Getting image source signatures
  Copying blob sha256:...
  Copying blob sha256:...
  Copying config sha256:...
  Writing manifest to image destination

Push multiple tags:
  podman push ghcr.io/radadiyamohit81/blk-hacking-ind-mohit-radadiya:latest
  podman push ghcr.io/radadiyamohit81/blk-hacking-ind-mohit-radadiya:1.0.0

================================================================================
6. MAKE THE PACKAGE PUBLIC
================================================================================

⚠️  By default, GitHub packages are PRIVATE. You MUST change visibility
    manually for the image to be publicly pullable.

Steps:
  a) Go to: https://github.com/YOUR_USERNAME?tab=packages
  b) Click on the package name (e.g., blk-hacking-ind-mohit-radadiya)
  c) Click "Package settings" (right sidebar)
  d) Scroll down to "Danger Zone"
  e) Click "Change visibility"
  f) Select "Public"
  g) Type the package name to confirm
  h) Click "I understand the consequences, change package visibility"

================================================================================
7. VERIFY THE PUBLIC IMAGE
================================================================================

Log out first to test as an anonymous user:
  podman logout ghcr.io

Pull the image (should work without authentication):
  podman pull ghcr.io/GITHUB_USERNAME/IMAGE_NAME:latest

Example:
  podman pull ghcr.io/radadiyamohit81/blk-hacking-ind-mohit-radadiya:latest

If this works, your image is public! ✅

Log back in:
  echo "$GITHUB_TOKEN" | podman login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin

================================================================================
8. PULL & RUN THE PUBLIC IMAGE
================================================================================

Anyone can now run your container:

  podman pull ghcr.io/radadiyamohit81/blk-hacking-ind-mohit-radadiya:latest
  podman run -p 5477:5477 ghcr.io/radadiyamohit81/blk-hacking-ind-mohit-radadiya:latest

Verify it's running:
  curl http://localhost:5477/blackrock/challenge/v1/performance

================================================================================
9. REVOKE / ROTATE TOKEN
================================================================================

⚠️  IMPORTANT: After you're done pushing, revoke the token for security.

Steps:
  a) Go to: https://github.com/settings/tokens
  b) Find your token (e.g., "podman-ghcr-push")
  c) Click "Delete" → Confirm

Also clear it from your shell:
  unset GITHUB_TOKEN
  history -c   # Clear shell history (optional)

================================================================================
10. TROUBLESHOOTING
================================================================================

ERROR: "403 Forbidden" during login
  → Your PAT token doesn't have write:packages scope
  → Generate a new token with the correct scopes (see Step 1)

ERROR: "denied" or "unauthorized" during push
  → You're not logged in: run podman login ghcr.io
  → Token expired: generate a new one
  → Wrong username in the image tag

ERROR: "name unknown" during pull
  → Package doesn't exist yet (push it first)
  → Package is still private (make it public — see Step 6)

ERROR: "exit code 125" during pull
  → Package is private and you're not authenticated
  → Make the package public OR log in first

IMAGE TOO LARGE?
  → Use multi-stage Dockerfile (build stage + slim runtime stage)
  → Use Alpine-based images (e.g., eclipse-temurin:21-jre-alpine)

================================================================================
  QUICK REFERENCE (COPY-PASTE COMMANDS)
================================================================================

# One-time setup
export GITHUB_TOKEN="ghp_your_token_here"
echo "$GITHUB_TOKEN" | podman login ghcr.io -u radadiyamohit81 --password-stdin

# Build, tag, push
podman build -t blk-hacking-ind-mohit-radadiya .
podman tag localhost/blk-hacking-ind-mohit-radadiya:latest ghcr.io/radadiyamohit81/blk-hacking-ind-mohit-radadiya:latest
podman push ghcr.io/radadiyamohit81/blk-hacking-ind-mohit-radadiya:latest

# Then go to GitHub → Packages → Make it Public

================================================================================
