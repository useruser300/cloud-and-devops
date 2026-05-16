# Key Concepts of Continuous Integration (CI) with GitHub Actions

## 1. What Is GitHub Actions?

GitHub Actions is the automation platform built into GitHub.

It uses YAML files to define:

- What should be executed.
- When it should be executed.
- Which environment should execute it.

In other words:

```text
GitHub Actions = Automation Engine inside GitHub
```

---

## 2. What Is Continuous Integration (CI)?

Continuous Integration (CI) is an automation process that automatically builds an application and produces new artifacts (such as Docker images) whenever the source code changes.

The core idea:

```text
Developer updates code
→ Push to GitHub
→ Workflow runs automatically
→ Build Docker image
→ Push image to registry
→ Artifact is ready for deployment
```

---

## 3. The Problem CI Solves

Without CI, every time the code changes, you would have to manually:

1. Rebuild the Docker image.
2. Log in to the Docker registry.
3. Push the new image.
4. Trigger deployment to update the runtime environment (such as Kubernetes or ECS). This step is usually part of Continuous Deployment (CD); see cd.md for more details.

This process is repetitive and error-prone.

CI automates all of these steps.

---

## 4. Practical Scenario

You have an application that contains:

- Source Code
- Dockerfile

### First Time

1. Build the Docker image using `docker build`.
2. Push the image to Docker Hub using `docker push`.

### After Modifying the Code

You must:

1. Rebuild a new Docker image.
2. Push the updated image to Docker Hub.

CI makes this process happen automatically after every Push.

---

## 5. Relationship Between Docker Hub and Kubernetes

After a Docker image is pushed to Docker Hub, any runtime platform such as Kubernetes can pull the image and run it.

```text
Kubernetes Deployment
→ Pull image from Docker Hub
→ Run containers
```

Therefore, the goal of CI is to ensure that the latest version of the application is always available in a Docker registry.

---

## 6. Events That Can Trigger a Workflow

In the `on:` section, you define when a workflow should run.

Common events include:

- `push`
- `pull_request`
- `workflow_dispatch`
- `schedule` (Cron)

---

## 7. Trigger Example

```yaml
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
```

This means the workflow runs when:

- Any Push is made to the `main` branch.
- Any Pull Request targets the `main` branch.

---

## 8. Manual Execution with `workflow_dispatch`

You can add:

```yaml
on:
  workflow_dispatch:
```

This allows you to run the workflow manually from the GitHub UI.

---

## 9. Scheduled Execution with `schedule`

```yaml
on:
  schedule:
    - cron: '0 0 * * *'
```

This example runs the workflow every day at midnight.

---

## 10. Location of GitHub Actions Workflow Files

Workflow files must be placed in:

```text
.github/workflows/
```

Example:

```text
.github/workflows/docker-build.yml
```

---

## 11. Workflow File Used in This Example

```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Log in to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push Docker image
        env:
          REPOSITORY: nginx
          IMAGE_TAG: latest
        run: |
          docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/$REPOSITORY:$IMAGE_TAG .
          docker push ${{ secrets.DOCKERHUB_USERNAME }}/$REPOSITORY:$IMAGE_TAG
```

---

## 12. Workflow Components

### `name`

The name of the workflow.

### `on`

The events that trigger the workflow.

### `jobs`

A collection of jobs to execute.

### `runs-on`

The operating system used to run the job.

### `steps`

The individual actions and commands executed within the job.

---

## 13. `name`

```yaml
name: Build and Push Docker Image
```

The workflow name shown in the Actions tab.

---

## 14. `on`

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
```

Defines when the workflow runs.

---

## 15. What Is a Job?

A Job is a group of steps that run on the same Runner.

Example:

```yaml
jobs:
  build-and-push:
```

---

## 16. What Is a Step?

A Step is a single action or command within a Job.

Example:

```yaml
- name: Checkout code
```

---

## 17. `runs-on`

```yaml
runs-on: ubuntu-latest
```

Specifies the environment where the job will run.

---

## 18. What Is a Runner?

A Runner is a temporary virtual machine that GitHub creates to execute the workflow.

Characteristics:

- Created when the workflow starts.
- Executes all steps.
- Deleted after the workflow completes.

---

## 19. `steps`

```yaml
steps:
```

The ordered list of steps inside the Job.

In this example:

1. Checkout code
2. Docker login
3. Build and push Docker image

---

## 20. `uses` vs `run`

### `uses`

Used to call a prebuilt Action.

```yaml
- uses: actions/checkout@v2
```

### `run`

Used to execute shell commands directly.

```yaml
- run: docker build -t myimage .
```

---

## 21. What Is `actions/checkout`?

```yaml
- name: Checkout code
  uses: actions/checkout@v2
```

Downloads the repository files from GitHub to the Runner.

Equivalent to:

```bash
git clone <repository-url>
```

Without this step, the Runner would not have access to:

- Dockerfile
- Source code
- Project files

### What Does Checkout Mean?

Checkout means:

Copying (downloading) the project files from GitHub to the Runner that executes the workflow.

Checkout is the process of delivering the project files to the Runner.

### Summary

`actions/checkout` is a ready-made Action that downloads the repository contents to the Runner so that the remaining workflow steps can use them.

---

## 22. Docker Login

Before executing `docker push`, you must log in to Docker Hub.

You can use a prebuilt Action such as:

```yaml
- uses: docker/login-action@v1
```

---

## 23. `with`

```yaml
with:
  username: ...
  password: ...
```

Used to pass values and settings to an Action.

In this example, the Docker username and token are passed to the Docker Login Action.

---

## 24. Why Not Store Passwords in YAML?

Because the workflow file is stored in the GitHub repository, and anyone with access to the repository may be able to read it.

Sensitive information should never be stored directly in the workflow file.

---

## 25. GitHub Secrets

A secure place to store sensitive information such as:

- Docker Hub Username
- Docker Hub Token
- AWS Credentials
- Azure Credentials

---

## 26. Using Secrets in a Workflow

```yaml
${{ secrets.DOCKERHUB_USERNAME }}
${{ secrets.DOCKERHUB_TOKEN }}
```

This means:

Retrieve these values from GitHub Secrets.

---

## 27. Creating Secrets

In GitHub:

```text
Repository
→ Settings
→ Secrets and variables
→ Actions
→ New repository secret
```

---

## 28. Docker Hub Access Token

It is recommended to use an Access Token instead of your password because Access Tokens are more secure.

They allow you to:

- Limit permissions.
- Revoke them easily.
- Use them without exposing your account password.

You can create one from your Docker Hub account settings.

---

## 29. `env`

```yaml
env:
  REPOSITORY: nginx
  IMAGE_TAG: latest
```

Defines environment variables used in `run` commands.

---

## 30. Build Docker Image

```bash
docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/$REPOSITORY:$IMAGE_TAG .
```

Builds a Docker image based on the Dockerfile.

---

## 31. Meaning of `-t`

Short for Tag.

Used to assign a full name to the image.

---

## 32. Docker Image Tag Format

General format:

```text
username/repository:tag
```

Example:

```text
ali/nginx:latest
```

---

## 33. Why Use This Name?

```bash
${{ secrets.DOCKERHUB_USERNAME }}/$REPOSITORY:$IMAGE_TAG
```

Because Docker Hub expects image names in the following format:

```text
username/repository:tag
```

---

## 34. Meaning of the Dot `.` in `docker build`

```bash
docker build -t image-name .
```

The dot means:

Use the current directory as the Build Context.

Docker will read from this directory:

- Dockerfile
- Source code
- Any files required during the build

---

## 35. Push Docker Image

```bash
docker push ${{ secrets.DOCKERHUB_USERNAME }}/$REPOSITORY:$IMAGE_TAG
```

Uploads the image to Docker Hub.

---

## 36. What Happens When You Run `git push`?

```text
git push
→ GitHub detects push event
→ Workflow starts
→ Runner is created
→ Checkout code
→ Docker login
→ Docker build
→ Docker push
→ Workflow completes
```

---

## 37. Monitoring Execution

In GitHub:

```text
Repository → Actions
```

You can view:

- Running workflows
- Logs for each step
- Success or failure status

---

## 38. Re-running a Workflow

From the workflow page, select:

```text
Re-run all jobs
```

---

## 39. Working with Another Branch Such as `dev`

If the workflow contains:

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
```

Then:

- `git push origin main` → Workflow runs.
- `git push origin dev` → Workflow does not run.
- Pull Request from `dev` to `main` → Workflow runs.

---

## 40. Practical Example with `dev`

```text
Create dev branch
→ Modify code
→ git push origin dev
→ Open Pull Request (dev → main)
→ Workflow runs
→ Review and merge
```

---

## 41. How to Run the Workflow on `dev` Too

```yaml
on:
  push:
    branches: [main, dev]
  pull_request:
    branches: [main]
```

---

## 42. Running a Workflow on Any Branch

If the workflow contains:

```yaml
workflow_dispatch:
```

You can run it manually on any branch.

---

## 43. Examples of Prebuilt Actions

- `actions/checkout`
- `docker/login-action`
- `docker/build-push-action`
- `aws-actions/configure-aws-credentials`

You can find them in:

- GitHub Marketplace
- GitHub Actions Documentation

---

## 44. Other Docker Registries

The same approach can be used with:

- Docker Hub
- Amazon Web Services ECR
- Microsoft ACR
- Google Cloud GCR

---

## 45. Cleanup

After the Job completes, GitHub automatically cleans up the temporary environment and removes the Runner.

---

## 46. The Big Picture

```text
GitHub Repository = Source Code
GitHub Actions = Automation Engine
Dockerfile = Build Instructions
Docker Registry = Image Storage
Kubernetes/ECS = Runtime Environment
```

---

## 47. Complete Flow

```text
Developer changes code
        ↓
git push
        ↓
GitHub Actions workflow starts
        ↓
Checkout source code
        ↓
Docker login
        ↓
Docker build
        ↓
Docker push
        ↓
New image stored in Docker Hub
        ↓
Deployment platform pulls new image
        ↓
Updated application runs
```
