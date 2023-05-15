---
title: Database Change Management with Redis and GitHub
author: Ningjing
published_at: 2023/04/14 11:45
feature_image: /docs/tutorials/intermediate/database-change-management-with-redis-and-github/feature-image.webp
tags: Tutorial
integrations: Redis, GitHub
level: Intermediate
description: This tutorial will bring your Redis data change to the next level by introducing the GitOps workflow, where you commit data change script to the GitHub repository, which will in turn trigger the data change pipeline in Bytebase.
---

This is a series of articles about Database Change Management with Redis

- [Database Change Management with Redis](/docs/tutorials/beginner/database-change-management-with-redis)
- Database Change Management with Redis and GitHub (this one)

---

In the last article [Database Change Management with Redis](/docs/tutorials/beginner/database-change-management-with-redis), you have tried UI workflow in Bytebase.

This tutorial will bring you to the next level by introducing the GitOps workflow, where you commit the schema change script to the GitHub repository, which will in turn trigger the schema deployment pipeline in Bytebase.

You can use Bytebase free version to finish the tutorial.

## Features included

- GitOps Workflow

## Prerequisites

Before you start this tutorial, make sure:

- You have followed our previous UI-based change tutorial [Database Change Management with Redis](/docs/tutorials/beginner/database-change-management-with-redis).
- You have [a local running Redis](https://redis.io/docs/getting-started/installation/).
- You have a [GitHub](https://github.com/) account.
- You have a public GitHub repository, e.g  `redis-test-bb-local`.
- You have [Docker](https://www.docker.com/) installed locally.
- You have a [ngrok](http://ngrok.com) account. ngrok is a reverse proxy tunnel, and in our case, we need it for a public network address in order to receive webhooks from GitHub.com. We use ngrok here for demonstration purposes. For production use, we recommend using [Caddy](https://caddyserver.com/).
  ![ngrok](/docs/tutorials/intermediate/database-change-management-with-redis-and-github/ngrok.webp)

## Step 1 - Run Bytebase in Docker with URL generated by ngrok

To make local-running Bytebase visible to GitHub, we’ll pass ngrok generated URL to [--external-url](https://www.bytebase.com/docs/get-started/install/external-url).

1. Login to [ngrok Dashboard](https://dashboard.ngrok.com/) and follow its [Getting Started](https://dashboard.ngrok.com/get-started/setup) steps to install and configure.

2. Run

```bash
ngrok http 5678
```

and obtain the public URL:
![terminal-ngrok](/docs/tutorials/intermediate/database-change-management-with-redis-and-github/terminal-ngrok.webp)

3. Make sure your Docker daemon is running, if it’s running Bytebase container for [the previous tutorial](/docs/tutorials/beginner/database-change-management-with-redis), there're two ways:

- 1a. Go to **Settings** > **Workspace** > **General**, fill the **External URL** in and click **Update**. Using this method, you can skip 4.
- 1b. Stop and remove it. The data created in the last tutorial is stored under `~/.bytebase/data` by default and will be restored if the system restarts.

4. Start the Bytebase Docker container by typing the following command in the terminal. Pay attention to the last parameter `--external-url https://9825-210-129-49-45.ngrok-free.app`, which is generated by ngrok.

```bash
docker run --init \
--name bytebase \
--platform linux/amd64 \
--restart always \
--publish 5678:8080 \
--health-cmd "curl --fail http://localhost:5678/healthz || exit 1" \
--health-interval 5m \
--health-timeout 60s \
--volume ~/.bytebase/data:/var/opt/bytebase \
bytebase/bytebase:%%bb_version%% \
--data /var/opt/bytebase \
--port 8080 \
--external-url https://9825-210-129-49-45.ngrok-free.app
```

5. Bytebase is running successfully in Docker, and you can visit it via `https://9825-210-129-49-45.ngrok-free.app`.
   ![docker](/docs/tutorials/intermediate/database-change-management-with-redis-and-github/docker.webp)

## Step 2 - Find your Redis instance in Bytebase

1. Visit `https://9825-210-129-49-45.ngrok-free.app` in your browser, and log in using your admin account created from the previous article.
   ![bb-login](/docs/tutorials/intermediate/database-change-management-with-redis-and-github/bb-login.webp)

2. If you have followed the last article, you should have a project `TestRedis` and a database `0`.

## Step 3 - Connect Bytebase with GitHub.com

1. Click **Settings** on the top bar, and then click **Workspace** > **GitOps**. Choose **GitHub.com** and click **Next**.
   ![bb-gitops-github-step1](/docs/tutorials/intermediate/database-change-management-with-redis-and-github/bb-gitops-github-step1.webp)

2. Follow the instructions within **STEP 2**, and in this tutorial, we will use a personal account instead of an organization account. The configuration is similar.

3. Go to your GitHub account. Click **Settings** on the dropdown menu.
   ![gh-settings-dropdown](/docs/tutorials/intermediate/database-change-management-with-redis-and-github/gh-settings-dropdown.webp)

4. Click **Developer Settings** at the bottom of the left side bar. Click **OAuth Apps**, and click **New OAuth App**.
   ![gh-oauth-apps](/docs/tutorials/intermediate/database-change-management-with-redis-and-github/gh-oauth-apps.webp)

5. Fill **Application name** and then copy the **Homepage** and **Authorization callback URL** in Bytebase and fill them. Click **Register application**.

6. After the OAuth application is created successfully. Click **Generate a new client secret**. Copy **Client ID** and this newly generated client secret and paste them back in Bytebase.
   ![bb-gitops-github-step2](/docs/tutorials/intermediate/database-change-management-with-redis-and-github/bb-gitops-github-step2.webp)
   ![gh-gitops-provider-redis](/docs/tutorials/intermediate/database-change-management-with-redis-and-github/gh-gitops-provider-redis.webp)

7. Click **Next**. You will be redirected to the confirmation page. Click **Confirm and add**, and the Git provider is successfully added.
   ![bb-gitops-github-step3](/docs/tutorials/intermediate/database-change-management-with-redis-and-github/bb-gitops-github-step3.webp)

## Step 4 - Enable GitOps workflow with Redis

1. Go to project `TestRedis`, click **GitOps**, and choose **GitOps Workflow**. Click **Configure GitOps**.
   ![bb-project-gitops-gitops-workflow](/docs/tutorials/intermediate/database-change-management-with-redis-and-github/bb-project-gitops-gitops-workflow.webp)

2. Choose `GitHub.com` - the provider you just added. It will display all the repositories you can manipulate. Choose `redis-test-bb-local`.
   ![bb-project-gitops-github-repo](/docs/tutorials/intermediate/database-change-management-with-redis-and-github/bb-project-gitops-github-repo.webp)

3. Keep the default setting, and click **Finish**.

## Step 5 - Change data for Redis by pushing SQL data change files to GitHub

1. In your GitHub repository `redis-test-bb-local`, create a folder `bytebase`, then create a subfolder `test`, and create an sql file following the pattern `{{ENV_ID}}/{{DB_NAME}}##{{VERSION}}##{{TYPE}}##{{DESCRIPTION}}.sql`. It is the default configuration for file path template setting under project GitOps.

   `test/0##2023041410400000##dml##add_country.sql`

   - `test` corresponds to `{{ENV_ID}}`
   - `0` corresponds to `{{DB_NAME}}`
   - `2023041410400000` corresponds to `{{VERSION}}`
   - `dml` corresponds to `{{TYPE}}`
   - `add_country` corresponds to `{{DESCRIPTION}}`

   Paste the sql script in it.

```sql
set country China
```

2. Commit and push this file.
   ![gh-add-sql](/docs/tutorials/intermediate/database-change-management-with-redis-and-github/gh-add-sql.webp)

3. Go to Bytebase, and go into project `TestRedis`. You’ll find there is a new `Push Event` and a new `issue 106` created.
   ![bb-project-push](/docs/tutorials/intermediate/database-change-management-with-redis-and-github/bb-project-push.webp)

4. Click `issue/106` and go the issue page. Click **Resolve issue**, and the issue will be `Done`. You’ll see
   - The issue is created via GitHub.com
   - The issue is executed without approval because it’s on `Test` environment where manual approval is skipped by default. The Assignee is `Bytebase`, because the execution is automatic, and requires no manual approval.
   - The SQL is exactly the one we have committed to the GitHub repository.
   - The Creator is `A`, because the GitHub user you use to commit the change has the same email address found in the Bytebase member list.

![bb-issue-gitops-change-data-done](/docs/tutorials/intermediate/database-change-management-with-redis-and-github/bb-issue-gitops-change-data-done.webp)

## Summary and Next

Now you have tried out GitOps workflow, which will store your Redis data in GitHub and trigger the change upon committing the change to the repository, to bring your Redis change workflow to the next level of Database DevOps - [Database as Code](/blog/database-as-code).

In real world scenario, you might have separate features and main branches corresponding to your dev and production environment, you can check out [GitOps with Feature Branch Workflow](/docs/how-to/workflow/gitops-feature-branch) to learn the setup. Have a try and look forward to your feedback!