# Secure RDP Access to GitHub Actions Runners via Tailscale

# ⚠️ Caution ⚠️
## Don't use your main github account 

This repository contains a GitHub Actions workflow that provides secure, on-demand Remote Desktop Protocol (RDP) access to a GitHub-hosted Windows runner.

Instead of exposing the RDP port to the public internet, this workflow uses [Tailscale](https://tailscale.com/) to create a secure, private overlay network. This allows you to connect directly to the runner from your local machine as if it were on the same local network, without complex firewall configurations or the security risks of a public IP.

## Features

  - **Secure by Default**: RDP access is restricted to your private Tailscale network (Tailnet), never exposed to the public internet.
  - **On-Demand Access**: Trigger the workflow manually whenever you need interactive access.
  - **Dynamic User Creation**: Automatically creates a dedicated RDP user with credentials you provide via GitHub Secrets.
  - **Robust Configuration**: The workflow includes checks for credentials, sets up firewall rules, and verifies connectivity.
  - **Long-Running Sessions**: The job is configured with a long timeout and a final step that keeps the runner alive so you can stay connected.

-----

## Prerequisites

Before you can use this workflow, you need the following:

1.  A **GitHub Repository** where you will add this workflow file.
2.  A **Tailscale Account**. You can sign up for a free personal account at [tailscale.com](https://tailscale.com/).
3.  The **Tailscale Client** installed and running on your local machine (the one you'll be connecting *from*).

-----

## Setup Instructions

Follow these steps to configure the workflow in your repository.

### Step 1: Fork This Repository by clicking [here](https://github.com/HARAJIT05/rdp/fork)

### Step 2: Generate a Tailscale Auth Key

An auth key allows the GitHub runner to join your private Tailnet without needing to log in interactively.

1.  Go to your Tailscale Admin Console: [https://login.tailscale.com/admin/settings/keys](https://login.tailscale.com/admin/settings/keys)
2.  Click **Generate auth key...**.
3.  Click **Generate key and check on the reusable toggle**.
4.  **Copy the key immediately**. You will not be able to see it again. It will look like `tskey-auth-k...`.

### Step 3: Configure GitHub Secrets

You must store sensitive information like passwords and auth keys as encrypted secrets in your repository.

1.  In your GitHub repository, go to **Settings** \> **Secrets and variables** \> **Actions**.

2.  Click **New repository secret** for each of the following secrets:

      * `TAILSCALE_AUTH_KEY`: Paste the Tailscale auth key you generated in the previous step.
      * `RDP_USERNAME`: Choose a username for the RDP account (e.g., `admin`, `dev`, `runner`).
      * `RDP_PASSWORD`: Create a **strong password** for the RDP account. Windows runners have password complexity requirements, so use a password with at least 8 characters, including uppercase letters, lowercase letters, and numbers.

-----

## How to Use

### Step 1: Run the Workflow

Because this workflow is triggered by `workflow_dispatch`, you must start it manually.

1.  Go to the **Actions** tab in your GitHub repository.
2.  In the left sidebar, click on the **RDP** workflow.
3.  You will see a message: "This workflow has a `workflow_dispatch` event trigger." Click the **Run workflow** button.
4.  Keep the default branch selected and click the green **Run workflow** button again.

### Step 2: Get Connection Details

1.  Click on the newly started workflow run. You will see the `secure-rdp` job running.

2.  Click on the `secure-rdp` job to view the live logs.

3.  Wait for the job to run through the setup steps. The final step, **Maintain Connection**, will print the necessary credentials.

4.  Look for the following output in the logs:

    ```
    === RDP ACCESS ===
    Address: 100.XX.XX.XX
    ==================
    ```

      * **Address**: This is the Tailscale IP address of the GitHub runner.
      * **Username**: The username you set in the `RDP_USERNAME` secret.
      * **Password**: The password you set in the `RDP_PASSWORD` secret (it will be masked as `***` in the logs for security, but the workflow uses the real value).

### Step 3: Connect via RDP

1.  Ensure the **Tailscale client is running** and you are logged in on your local machine.
2.  Open your preferred RDP client (e.g., "Remote Desktop Connection" on Windows or "Microsoft Remote Desktop" on macOS).
3.  In the "Computer" or "PC name" field, enter the **Tailscale IP Address** from the logs.
4.  When prompted, enter the **Username** and **Password** you configured in the GitHub secrets.
5.  Connect\! You should now have a full desktop session on the GitHub Actions runner.

### Step 4: Terminate the Session

The runner will stay online for up to 6 hours (the `timeout-minutes` value). When you are finished with your session:

  - Go back to the GitHub Actions workflow run page and click **Cancel workflow**. This is important as it stops the runner and prevents you from using up your free Actions minutes.

-----
### Feel free to star this repository ⭐
