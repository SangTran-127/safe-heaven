# Guide to Running Dev Containers on VS Code via Remote-SSH (Optimized for Weak VPS)

This guide provides detailed instructions on how to set up and optimize a **Go 1.26** development environment using **VS Code** connected to a **Remote VPS** via **Remote-SSH**. The environment is configured to be minimal, keeping RAM and CPU usage on your VPS as low as possible.

---

## 1. Prerequisites

### On your Local Machine (Host macOS):
1.  Install **VS Code**.
2.  Install two official extensions from Microsoft:
    *   **Remote - SSH** (to connect VS Code to your VPS).
    *   **Dev Containers** (to build and run Docker containers on your VPS).
3.  **Configure SSH Key-based Authentication (Highly Recommended):**
    Ensure you have copied your local SSH public key to your VPS to allow passwordless login:
    ```bash
    ssh-copy-id root@<YOUR_VPS_IP>
    ```

### On the Remote VPS:
1.  Install **Docker Engine** and ensure it is running.
2.  Make sure the project directory (`test-devcontainer`) is cloned or copied onto the VPS (e.g., at `/root/test-devcontainer` or any other location).
    *   *Tip:* You can use `git clone` or SFTP client tools to upload your project directory to the VPS beforehand.

---

## 2. Step-by-Step Setup Guide

Since the VPS has limited resources, the best method is to connect VS Code to the VPS via SSH first, then run the Dev Container locally on the VPS.

### Step 1: Connect VS Code to the VPS via SSH
1.  Open VS Code on your local machine.
2.  Click on the **Remote Explorer** icon in the Activity Bar (or press `Cmd + Shift + P` / `Ctrl + Shift + P` and search for `Remote-SSH: Connect to Host...`).
3.  Enter the connection string: `root@<YOUR_VPS_IP>`.
4.  VS Code will open a new window and install the lightweight VS Code Server on the VPS (this process is fast and efficient).

### Step 2: Open the Project Folder on the VPS
1.  In the VS Code window connected via SSH, select **File** -> **Open Folder...**
2.  Select the path containing the project folder on the VPS (e.g., `/root/test-devcontainer`). Click **OK**.
3.  Enter your SSH password if prompted (if SSH keys are not set up).

### Step 3: Reopen in Container
1.  Once the folder is opened, VS Code will automatically detect the `.devcontainer/devcontainer.json` file and show a notification popup in the bottom right corner: *"Folder contains a Dev Container configuration..."*.
2.  Click **Reopen in Container** (or press `Cmd + Shift + P` / `Ctrl + Shift + P` and search for `Dev Containers: Reopen in Container`).
3.  VS Code will pull the pre-built image from GitHub Container Registry (GHCR) and launch the container in about **5 seconds** without running heavy CPU-intensive `docker build` processes on your VPS.

---

## 3. GitHub Actions CI/CD Pipeline

To protect your weak VPS, we have set up a GitHub Actions workflow to build the Docker image natively on GitHub's servers and host it on GitHub Container Registry (GHCR) at `ghcr.io/sangtran-127/safe-heaven:latest`.

### How it works:
*   Every time you push a change to `.devcontainer/` (like modifying your `Dockerfile` or packages) to the `main` or `master` branch, GitHub Actions will automatically rebuild the image.
*   You can also manually trigger a build from the **Actions** tab on your GitHub repository page by selecting the **Build and Publish Dev Container** workflow and clicking **Run workflow**.

### First-Time Run / Changing Configurations:
1.  Add a remote to your local git repo pointing to GitHub:
    ```bash
    git remote add origin https://github.com/SangTran-127/test-devcontainer.git
    ```
2.  Commit and push your changes to GitHub to trigger the build workflow.
3.  Wait for the build to finish (about 2-3 minutes) on the GitHub Actions page.
4.  Once finished, open VS Code Remote-SSH on your VPS and select **Reopen in Container** / **Rebuild Container**. VS Code will download the new pre-built image instantly.

### 🧪 Local Override / Testing Dockerfile changes:
If you want to edit your `Dockerfile` and test the build *locally* without pushing to GitHub first:
1.  Open `.devcontainer/devcontainer.json`.
2.  Temporarily replace `"image": "ghcr.io/sangtran-127/safe-heaven:latest"` with:
    ```json
    "build": {
      "dockerfile": "Dockerfile"
    }
    ```
3.  Rebuild your container. VS Code will build the image locally.
4.  Once you are happy with the changes, restore `"image": "ghcr.io/sangtran-127/safe-heaven:latest"`, commit and push to Git to let GitHub Actions build the final image for the VPS.

---

## 4. Resource Optimization Tips for Weak VPS

To keep your VPS running as efficiently as possible, apply the following settings in your VS Code `settings.json` (User/Remote Settings):

### 🛠️ Optimized VS Code Settings

Open Settings (`Cmd + ,` / `Ctrl + ,`), select the **Remote [SSH: <YOUR_VPS_IP>]** tab or **Application** and configure the following:

1.  **Disable Git Autofetch (Configured automatically in devcontainer.json):**
    *   `"git.autofetch": false` - Prevents VS Code from executing periodic background git scans, which saves CPU cycles.
2.  **Limit File Watcher (Saves significant CPU/RAM):**
    Exclude large dependency and VCS directories from being watched:
    ```json
    "files.watcherExclude": {
      "**/.git/objects/**": true,
      "**/.git/subtree-cache/**": true,
      "**/node_modules/**": true,
      "**/vendor/**": true,
      "**/.venv/**": true
    }
    ```
3.  **Disable Telemetry:**
    Reduce background diagnostics and data reporting overhead:
    ```json
    "telemetry.telemetryLevel": "off"
    ```
4.  **Keep Extensions to a Minimum:**
    Only install essential extensions. The dev container config has already been pruned to keep only the Go extension (`golang.go`) and the TOML parser (`tamasfe.even-better-toml`).
