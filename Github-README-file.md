
---

# Jenkins Pipelines (Shared Library) — Asaf CI Hub

This repository contains a **central Jenkins Shared Library** used to run a **single, reusable CI pipeline** across **multiple GitHub repositories** (Node, Python, Java, Playwright, etc.) without rebuilding pipelines per project.

The goal:
✅ Any repo with a minimal `Jenkinsfile` can automatically run CI in Jenkins
✅ Centralized logic lives here (one place to update)
✅ Supports both public and private repos (via GitHub token)
✅ Runs inside Docker (easy to remove/reset)

---

## 1) What We Built

### ✅ Jenkins running in Docker (Windows + Docker Desktop / WSL2)

* Jenkins runs as a container: `jenkins-server`
* Jenkins UI is available at:

  * `http://localhost:8080`

### ✅ Organization Folder / Multi-Repo CI

Jenkins is configured to scan your GitHub account/organization repositories and build them as **multibranch pipelines**.

### ✅ Shared Library

This repo is registered in Jenkins as a **Global Pipeline Library** (name example):

* `jenkins-pipelines`

Any repo can call it via:

```groovy
@Library('jenkins-pipelines@main') _
runPipeline()
```

### ✅ Central CI logic (one file)

Main pipeline logic is in:

```
vars/runCI.groovy
```

This pipeline:

* Checks out the repo
* Detects the project type by files
* Runs CI steps per type (Node / Python / Maven / Gradle)
* Publishes:

  * ✅ JUnit results (if XML exists)
  * ✅ Playwright HTML report (if `playwright-report/` exists)

---

## 2) Folder Structure (This Repo)

```
jenkins-pipelines/
│
├── README.md
└── vars/
    ├── runPipeline.groovy   # wrapper: calls runCI()
    └── runCI.groovy         # main shared pipeline logic
```

Important Jenkins requirement:

* A shared library repo must contain at least one of:

  * `vars/` ✅
  * `src/`

---

## 3) Requirements / Installations Done

### On Windows Host

* Docker Desktop installed and running
* Docker Engine context: `desktop-linux`

### Jenkins Container

* Jenkins runs in Docker (Debian-based image: `jenkins/jenkins:lts`)
* Jenkins uses `/var/jenkins_home` to store:

  * jobs
  * credentials
  * plugins
  * config

> Note: Tools installed inside the container are available to builds that run on that container.

### Jenkins Plugins (Installed / Used)

* Git + GitHub integration plugins (for repo scanning + checkout)
* **AnsiColor** (installed to support `ansiColor('xterm')` if needed)
* **HTML Publisher** (recommended for Playwright HTML report)

---

## 4) How It Works (High Level)

**Per repo build flow:**

```
GitHub repo change
   ↓
Jenkins Organization Folder detects branch
   ↓
Repo contains Jenkinsfile
   ↓
Jenkinsfile loads Shared Library (this repo)
   ↓
Shared library runs runCI.groovy
   ↓
CI stages run + reports published
```

---

## 5) Minimal Setup Required Per Repo (IMPORTANT)

Each repository must contain **one file** at the repository root:

### `Jenkinsfile`

```groovy
@Library('jenkins-pipelines@main') _
runPipeline()
```

That’s it.

> Without a `Jenkinsfile`, Jenkins cannot run that repository automatically.

---

## 6) Project Type Detection Rules

The shared pipeline detects project type using common files:

### Node / JavaScript

* `package.json` OR `yarn.lock` OR `pnpm-lock.yaml`

### Python

* `requirements.txt` OR `pyproject.toml` OR `setup.py`

### Maven

* `pom.xml`

### Gradle

* `build.gradle` OR `build.gradle.kts`

---

## 7) Test Reports Support

### ✅ JUnit XML test results (Displayed in Jenkins "Tests")

Jenkins scans for test result XML files:

* `**/test-results/*.xml`
* `**/junit*.xml`
* `**/TEST-*.xml`
* `**/surefire-reports/*.xml`

If none exist → build still succeeds (allowEmptyResults).

### ✅ Playwright HTML report (Displayed as link)

If Playwright generates:

```
playwright-report/index.html
```

Jenkins:

* archives it as artifacts
* publishes it via HTML Publisher plugin

> If not present → no failure.

---

## 8) Triggers (How Builds Start)

You can trigger builds via:

### A) Manual

* `Build Now`

### B) Poll SCM (cron schedule)

Example:

* `H/1 * * * *`
  means: Jenkins checks for new commits every ~1 minute (hashed)

### C) GitHub Webhook (Recommended)

If configured, builds trigger instantly on push/merge.
✅ Webhooks do not cost money.

---

## 9) Memory / Performance Notes

Jenkins runs inside Docker, so you can limit resources via Docker Desktop:

* RAM limit for Docker engine (WSL2)
* CPU limit

Jenkins itself can be tuned later (executors, workspace cleanup, build retention).

---

## 10) Add a New Repository in the Future

1. Create or push a repo in GitHub
2. Add `Jenkinsfile` to repo root:

```groovy
@Library('jenkins-pipelines@main') _
runPipeline()
```

3. Make sure the repo is included in your Organization Folder scanning
4. Jenkins will detect branches and run CI automatically

---

## 11) Next Improvements (Roadmap)

### Recommended Next Steps

✅ **Dockerfile custom Jenkins image**

* Bake Node/Python/Java/Playwright into the Jenkins image so tools never disappear

✅ **Playwright JUnit integration**

* Configure Playwright to output JUnit XML so Jenkins Tests view shows Pass/Fail counts

✅ **Parallel builds**

* Run multiple repos / branches concurrently

✅ **Workspace cleanup**

* Reduce disk usage automatically

✅ **Artifacts / Reports**

* Store:

  * Playwright trace files
  * screenshots/videos
  * coverage reports

✅ Notifications

* Slack / Email notifications on failure

✅ GitHub PR checks

* Report status back to PRs as checks

---

## 12) Safety / Removal (High Level)

This entire Jenkins environment is isolated under Docker.
You can stop / remove Jenkins without affecting Windows system tools.

**Stop Jenkins container:**

```bash
docker stop jenkins-server
```

**Start again:**

```bash
docker start jenkins-server
```

**Remove the container (data may remain in volumes):**

```bash
docker rm -f jenkins-server
```

> Full cleanup guide can be added when needed (including volumes and Docker reset).

---

## Credits / Ownership

Built and maintained by **Asaf** as a personal CI hub for multi-repo automation projects.

---

Full Cleanup Guide (Reset Jenkins Completely)

If you ever want to reset the entire Jenkins environment, including the Docker container and all Jenkins data, follow the steps below.

⚠️ This will remove:

Jenkins jobs

pipeline history

credentials

plugins

build artifacts

Jenkins configuration

1) Stop Jenkins Container
docker stop jenkins-server
2) Remove Jenkins Container
docker rm jenkins-server

or force remove if needed:

docker rm -f jenkins-server
3) Remove Jenkins Volumes (Important)

Jenkins stores its data inside Docker volumes.

List volumes:

docker volume ls

Look for something like:

jenkins_home

Remove it:

docker volume rm jenkins_home

If you want to remove all unused volumes:

docker volume prune
4) Remove Jenkins Image (Optional)
docker rmi jenkins/jenkins:lts
5) Clean Unused Docker Resources (Optional Full Reset)

This removes unused containers, networks, and images.

docker system prune -a

⚠️ Only run this if you understand it may remove other unused Docker resources.

6) Start Jenkins Again From Scratch

Pull image again:

docker pull jenkins/jenkins:lts

Run Jenkins container again:

docker run -d ^
  --name jenkins-server ^
  -p 8080:8080 ^
  -p 50000:50000 ^
  jenkins/jenkins:lts

Then open:

http://localhost:8080

and repeat the Jenkins setup.

Tip

If you plan to run Jenkins long-term, consider creating a custom Docker image with tools preinstalled:

Node.js

Python

Java

Playwright

This prevents reinstalling tools after container resets.
