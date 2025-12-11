+++
title = "CI/CD"
weight = 3
chapter = false
pre = " <b> 5.3.3.  </b> "
+++

### OVERVIEW WORKFLOW

1.  **Security Configuration:** Provide AWS credentials to GitHub.

2.  **Project Structure:** Create `.yml` workflow files.

3.  **Workflow Content:** Copy standard code for 6 services.

4.  **Activation:** Push code to GitHub to trigger the system.

* * * * *

### STEP 1: SECURITY CONFIGURATION (SECRETS)

GitHub needs AWS access rights to deploy on your behalf.

1.  Go to your Repository on GitHub.

2.  Select **Settings** > **Secrets and variables** > **Actions**.

3.  Click **New repository secret** and add the following 3 variables (Get these from your `.aws/credentials` file):

| **Secret Name** | **Value** | **Note** |
| --- | --- | --- |
| **`AWS_ACCESS_KEY_ID`** | `AKIA...` (or `ASIA...`) | Required |
| **`AWS_SECRET_ACCESS_KEY`** | `...` (Long secret string) | Required |
| **`AWS_SESSION_TOKEN`** | `...` (Very long string) | **Required** if using AWS Academy (Learner Lab). Update every 4 hours. |

* * * * *

### STEP 2: PREPARE CONFIGURATION FILES

In your project's root directory on your local machine, create (or verify) the folder structure as follows:

Plaintext

```
MY-PROJECT/
├── .gitignore               <-- File to ignore junk (create new if not exists)
├── .github/
│   └── workflows/           <-- Folder containing CI/CD files
│       ├── deploy-auth.yml
│       ├── deploy-gateway.yml
│       ├── deploy-user.yml
│       ├── deploy-task.yml
│       ├── deploy-noti.yml
│       └── deploy-model.yml

```

**Tip:** Create a `.gitignore` file and add the line `*.zip` to avoid pushing large junk files to GitHub.

* * * * *

### STEP 3: WORKFLOW CONTENT (Copy & Paste)

#### 1\. File: `.github/workflows/deploy-apigateway.yml`



```YAML
# ========================================================
# 1. WORKFLOW IDENTITY
# ========================================================
name: Deploy API Gateway  # Display name in the "Actions" tab on GitHub.

# ========================================================
# 2. TRIGGER
# ========================================================
on:
  push:  # Trigger event: When "Push code" action occurs...
    branches: [ "main", "pre-production" ]  # ...to one of these 2 branches (pushes to other branches won't run).

    paths:  # SMART FILTER (Important):
      - 'api-gateway/**'                    # Only run when files in 'api-gateway' folder change.
      - '.github/workflows/deploy-gateway.yml'  # Or when this config file itself is modified.
      # (Saves resources: Modifying auth-service won't trigger api-gateway deploy).

# ========================================================
# 3. ENVIRONMENT VARIABLES (GLOBAL)
# ========================================================
env:
  AWS_REGION: ap-southeast-1          # AWS Server Region (Singapore).
  ECR_REPOSITORY: api-gateway         # Repository Name on AWS ECR.
  ECS_SERVICE: api-gateway            # Service Name running on AWS ECS Fargate.
  ECS_CLUSTER: SGUTodolist-Cluster    # Cluster Name containing services.
  WORKING_DIRECTORY: ./api-gateway    # Path to source code folder of this service.

# ========================================================
# 4. JOBS (TASKS TO PERFORM)
# ========================================================
jobs:
  deploy:              # Job ID.
    name: Build & Deploy  # Job Display Name.
    runs-on: ubuntu-latest  # Environment: GitHub provides a fresh Ubuntu VM.
    environment: production # Environment tag (for better Secrets management on GitHub).

    # ====================================================
    # 5. STEPS (SEQUENTIAL ACTIONS)
    # ====================================================
    steps:

    # STEP 1: FETCH CODE
    - name: Checkout Code
      id: checkout             # Set ID for error handling step to check status.
      uses: actions/checkout@v3 # Use existing GitHub action to download repo code to Ubuntu VM.

    # STEP 2: INSTALL JAVA
    - name: Set up JDK 17
      id: setup-jdk            # Set ID.
      uses: actions/setup-java@v3 # Action to install Java.
      with:
        java-version: '17'     # Select Java version 17.
        distribution: 'temurin' # Select JDK provider (Eclipse Temurin).
        # Purpose: Prepare environment if running tests or Maven build outside Docker is needed.

    # STEP 3: LOGIN TO AWS
    - name: Configure AWS Credentials
      id: aws-creds            # Set ID.
      uses: aws-actions/configure-aws-credentials@v1 # Action to login AWS CLI.
      with:
        # Get sensitive info from "Vault" (Settings > Secrets) of GitHub Repo.
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }} # Temporary Token (Required for AWS Academy/Learner Lab).
        aws-region: ${{ env.AWS_REGION }} # Get region from env variable defined above.

    # STEP 4: LOGIN TO ECR (Docker Registry)
    - name: Login to Amazon ECR
      id: login-ecr            # Set ID (Important: Next step needs output from this step).
      uses: aws-actions/amazon-ecr-login@v1 # Action helps VM Docker login to AWS ECR.

    # STEP 5: BUILD & PUSH DOCKER IMAGE (Core)
    - name: Build, Tag, and Push Image
      id: build-image          # Set ID.
      env:
        # Get Registry URL from login-ecr step output (e.g., 0311...dkr.ecr...).
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        # Use current Commit Hash as Tag (e.g., a1b2c3d). Helps version code.
        IMAGE_TAG: ${{ github.sha }}
      run: |  # Start running Linux commands (Bash shell)
        cd ${{ env.WORKING_DIRECTORY }}  # Move to code directory (./api-gateway).

        # Command 1: Build Image. Assign 2 tags simultaneously:
        # - Tag by commit hash (for history tracking).
        # - Tag 'latest' (for ECS to always pull this newest version).
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .

        # Command 2: Push Image with commit tag to ECR.
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG

        # Command 3: Push Image with latest tag to ECR.
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

    # STEP 6: UPDATE ECS (Deploy)
    - name: Force ECS Update
      id: ecs-update           # Set ID.
      run: |
        # AWS CLI command to update Service.
        # --force-new-deployment: Force ECS to stop old containers and start new ones (pulling the latest image).
        aws ecs update-service --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --force-new-deployment

    # STEP 7: SUCCESS NOTIFICATION
    - name: Success Notification
      if: success()            # Condition: Only run this line if ALL steps above are OK (Green).
      run: echo "🎉 DEPLOY API GATEWAY SUCCESSFUL!"

    # STEP 8: ERROR HANDLER
    - name: ⛔ ERROR HANDLER
      if: failure()            # Condition: Only run this line if ANY step above fails (Red).
      run: |
        echo "🚨 DEPLOY GATEWAY FAILED!"
        # Check each step to see which one died to print easy-to-understand notification.
        if [[ "${{ steps.checkout.outcome }}" == 'failure' ]]; then echo "❌ Error: Checkout Code"; fi
        if [[ "${{ steps.aws-creds.outcome }}" == 'failure' ]]; then echo "❌ Error: AWS Credentials (Wrong Key/Expired)"; fi
        if [[ "${{ steps.build-image.outcome }}" == 'failure' ]]; then echo "❌ Error: Docker Build/Push (Code error or wrong Dockerfile)"; fi
        if [[ "${{ steps.ecs-update.outcome }}" == 'failure' ]]; then echo "❌ Error: ECS Service Not Found (Wrong Cluster or Service Name)"; fi
        exit 1 # Report failure to GitHub (show red X on dashboard).

```

#### 2\. File: `.github/workflows/deploy-auth.yml`



```YAML
name: Deploy Auth Service 

on:
  push:
    branches: [ "main", "pre-production" ]
    paths:
      - 'auth-service/**'
      - '.github/workflows/deploy-auth.yml'

env:
  AWS_REGION: ap-southeast-1
  ECR_REPOSITORY: auth-service
  ECS_SERVICE: auth-service
  ECS_CLUSTER: SGUTodolist-Cluster
  WORKING_DIRECTORY: ./auth-service

jobs:
  deploy:
    name: Build & Deploy
    runs-on: ubuntu-latest
    environment: production

    steps:
    - name: Checkout Code
      id: checkout
      uses: actions/checkout@v3

    - name: Set up JDK 17
      id: setup-jdk
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: Configure AWS Credentials
      id: aws-creds
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, Tag, and Push Image
      id: build-image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        IMAGE_TAG: ${{ github.sha }}
      run: |
        cd ${{ env.WORKING_DIRECTORY }}
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

    - name: Force ECS Update
      id: ecs-update
      run: |
        aws ecs update-service --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --force-new-deployment

    - name: Success Notification
      if: success()
      run: echo "🎉 DEPLOY AUTH SERVICE SUCCESSFUL!"

    - name: ⛔ ERROR HANDLER
      if: failure()
      run: |
        echo "🚨 DEPLOY AUTH FAILED!"
        if [[ "${{ steps.aws-creds.outcome }}" == 'failure' ]]; then echo "❌ Error: AWS Credentials"; fi
        if [[ "${{ steps.build-image.outcome }}" == 'failure' ]]; then echo "❌ Error: Docker Build/Push"; fi
        exit 1

```

#### 3\. File: `.github/workflows/deploy-user.yml`



```YAML
name: Deploy User Service 

on:
  push:
    branches: [ "main", "pre-production" ]
    paths:
      - 'user-service/**'
      - '.github/workflows/deploy-user.yml'

env:
  AWS_REGION: ap-southeast-1
  ECR_REPOSITORY: user-service
  ECS_SERVICE: user-service
  ECS_CLUSTER: SGUTodolist-Cluster
  WORKING_DIRECTORY: ./user-service

jobs:
  deploy:
    name: Build & Deploy
    runs-on: ubuntu-latest
    environment: production

    steps:
    - name: Checkout Code
      id: checkout
      uses: actions/checkout@v3

    - name: Set up JDK 17
      id: setup-jdk
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: Configure AWS Credentials
      id: aws-creds
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, Tag, and Push Image
      id: build-image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        IMAGE_TAG: ${{ github.sha }}
      run: |
        cd ${{ env.WORKING_DIRECTORY }}
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

    - name: Force ECS Update
      id: ecs-update
      run: |
        aws ecs update-service --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --force-new-deployment

    - name: Success Notification
      if: success()
      run: echo "🎉 DEPLOY USER SERVICE SUCCESSFUL!"

    - name: ⛔ ERROR HANDLER
      if: failure()
      run: |
        echo "🚨 DEPLOY USER FAILED!"
        if [[ "${{ steps.aws-creds.outcome }}" == 'failure' ]]; then echo "❌ Error: AWS Credentials"; fi
        if [[ "${{ steps.build-image.outcome }}" == 'failure' ]]; then echo "❌ Error: Docker Build/Push"; fi
        exit 1

```

#### 4\. File: `.github/workflows/deploy-task.yml`



```YAML
name: Deploy Taskflow Service 

on:
  push:
    branches: [ "main", "pre-production" ]
    paths:
      - 'taskflow-service/**'
      - '.github/workflows/deploy-task.yml'

env:
  AWS_REGION: ap-southeast-1
  ECR_REPOSITORY: taskflow-service
  ECS_SERVICE: taskflow-service
  ECS_CLUSTER: SGUTodolist-Cluster
  WORKING_DIRECTORY: ./taskflow-service

jobs:
  deploy:
    name: Build & Deploy
    runs-on: ubuntu-latest
    environment: production

    steps:
    - name: Checkout Code
      id: checkout
      uses: actions/checkout@v3

    - name: Set up JDK 17
      id: setup-jdk
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: Configure AWS Credentials
      id: aws-creds
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, Tag, and Push Image
      id: build-image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        IMAGE_TAG: ${{ github.sha }}
      run: |
        cd ${{ env.WORKING_DIRECTORY }}
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

    - name: Force ECS Update
      id: ecs-update
      run: |
        aws ecs update-service --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --force-new-deployment

    - name: Success Notification
      if: success()
      run: echo "🎉 DEPLOY TASKFLOW SERVICE SUCCESSFUL!"

    - name: ⛔ ERROR HANDLER
      if: failure()
      run: |
        echo "🚨 DEPLOY TASKFLOW FAILED!"
        if [[ "${{ steps.aws-creds.outcome }}" == 'failure' ]]; then echo "❌ Error: AWS Credentials"; fi
        if [[ "${{ steps.build-image.outcome }}" == 'failure' ]]; then echo "❌ Error: Docker Build/Push"; fi
        exit 1

```

#### 5\. File: `.github/workflows/deploy-noti.yml`



```YAML
name: Deploy Notification Service 

on:
  push:
    branches: [ "main", "pre-production" ]
    paths:
      - 'notification-service/**'
      - '.github/workflows/deploy-noti.yml'

env:
  AWS_REGION: ap-southeast-1
  ECR_REPOSITORY: notification-service
  ECS_SERVICE: notification-service
  ECS_CLUSTER: SGUTodolist-Cluster
  WORKING_DIRECTORY: ./notification-service

jobs:
  deploy:
    name: Build & Deploy
    runs-on: ubuntu-latest
    environment: production

    steps:
    - name: Checkout Code
      id: checkout
      uses: actions/checkout@v3

    - name: Set up JDK 17
      id: setup-jdk
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: Configure AWS Credentials
      id: aws-creds
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, Tag, and Push Image
      id: build-image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        IMAGE_TAG: ${{ github.sha }}
      run: |
        cd ${{ env.WORKING_DIRECTORY }}
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

    - name: Force ECS Update
      id: ecs-update
      run: |
        aws ecs update-service --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --force-new-deployment

    - name: Success Notification
      if: success()
      run: echo "🎉 DEPLOY NOTIFICATION SERVICE SUCCESSFUL!"

    - name: ⛔ ERROR HANDLER
      if: failure()
      run: |
        echo "🚨 DEPLOY NOTIFICATION FAILED!"
        if [[ "${{ steps.aws-creds.outcome }}" == 'failure' ]]; then echo "❌ Error: AWS Credentials"; fi
        if [[ "${{ steps.build-image.outcome }}" == 'failure' ]]; then echo "❌ Error: Docker Build/Push"; fi
        exit 1

```

#### 6\. File: `.github/workflows/deploy-model.yml`



```YAML
name: Deploy AI Model 

on:
  push:
    branches: [ "main", "pre-production" ]
    paths:
      - 'model/**'
      - '.github/workflows/deploy-model.yml'

env:
  AWS_REGION: ap-southeast-1
  ECR_REPOSITORY: ai-model-service
  ECS_SERVICE: ai-model-service
  ECS_CLUSTER: SGUTodolist-Cluster
  WORKING_DIRECTORY: ./model    # Note: Folder name is 'model'

jobs:
  deploy:
    name: Build & Deploy
    runs-on: ubuntu-latest
    environment: production

    steps:
    - name: Checkout Code
      id: checkout
      uses: actions/checkout@v3

    # Python does not need JDK setup

    - name: Configure AWS Credentials
      id: aws-creds
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, Tag, and Push Image
      id: build-image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        IMAGE_TAG: ${{ github.sha }}
      run: |
        cd ${{ env.WORKING_DIRECTORY }}
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

    - name: Force ECS Update
      id: ecs-update
      run: |
        aws ecs update-service --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --force-new-deployment

    - name: Success Notification
      if: success()
      run: echo "🎉 DEPLOY AI MODEL SUCCESSFUL!"

    - name: ⛔ ERROR HANDLER
      if: failure()
      run: |
        echo "🚨 DEPLOY AI MODEL FAILED!"
        if [[ "${{ steps.aws-creds.outcome }}" == 'failure' ]]; then echo "❌ Error: AWS Credentials"; fi
        if [[ "${{ steps.build-image.outcome }}" == 'failure' ]]; then echo "❌ Error: Docker Build/Push"; fi
        exit 1

```

* * * * *

### STEP 4: TRIGGER

Open Terminal at project folder and run:

Bash

```
git add .
git commit -m "Official CI/CD Setup: 6 Services"
git push origin pre-production

```

**Immediately after:**

1.  Go to GitHub > **Actions** Tab.

2.  You will see 6 workflows starting to run.

3.  Wait for them to turn Green ✅.

* * * * *

### OVERVIEW WORKFLOW (CI/CD WORKFLOW)

This is the journey of a line of code from Developer's machine to Server (AWS ECS), fully automated:

1.  **Code Change:** Developer modifies code (e.g., changes button color, fixes logic error) on local machine.

2.  **Git Push:** Developer pushes code to GitHub.

    -   *If pushing to personal branch (`minh`, `dat`...):* System does **NOTHING** (Saves resources).

    -   *If merging into `main` or `pre-production`:* System **AUTOMATICALLY ACTIVATES**.

3.  **GitHub Actions (The Robot):**

    -   Automatically spins up a fresh Ubuntu VM.

    -   Downloads latest code.

    -   Logs into AWS using "Secret Keys".

    -   Packages code into **Docker Image**.

    -   Pushes Image to **AWS ECR** repository.

4.  **Deploy (AWS ECS):**

    -   GitHub Actions commands **AWS ECS**: *"Hey, there's a new update, replace the old containers!"*.

    -   AWS ECS automatically stops old container, pulls new image, and starts new container.

5.  **Result:** Users accessing the website see the new feature immediately without anyone SSHing into server to type commands.

* * * * *

### USER GUIDE (FOR DEVELOPERS)

To ensure CI/CD system runs smoothly and avoid conflicts, each team member needs to follow this process every time they want to update code:

#### 1\. Branching Strategy

-   **`main`**: OFFICIAL Branch. Code here is live version (Production). **DIRECT PUSH FORBIDDEN**, only Merge from other branches allowed.

-   **`pre-production`**: TRIAL RUN Branch. This is for integration testing. Merging here automatically triggers test server deploy.

-   **`developer`, `minh`, `dat`...**: PERSONAL Branch. Freely code, commit, push without fear of breaking the server.

#### 2\. Code Update Steps (Standard Process)

**Step 1: Code and Test Locally**

-   Code new feature on personal branch (e.g., `dat`).

-   Run locally to ensure no errors.

**Step 2: Push code to GitHub**

Bash

```
git add .
git commit -m "Add Login with Google feature"
git push origin dat  # Push to your own branch

```

*(At this point CI/CD hasn't run, server remains peaceful).*

**Step 3: Create Pull Request (PR) or Merge**

-   When you want to push feature to server for team testing, create **Pull Request** from `dat` branch to `pre-production` branch.

-   Or if team is small, direct merge is possible:

    Bash

    ```
    git checkout pre-production
    git pull origin pre-production  # Update latest code from server first
    git merge dat                   # Merge your code in
    git push origin pre-production  # Push up -> 🚀 BOOM! CI/CD ACTIVATED

    ```

**Step 4: Monitor and Enjoy**

-   Go to **Actions** tab on GitHub Repo.

-   You will see **"Deploy ... Service"** workflow running (yellow spinning circle).

-   Wait about 3-5 minutes until green check ✅ appears.

-   Go to web to check new feature.

#### 3\. Handling Errors (Red ❌ Mark)

If Actions reports Red (Failed):

1.  Click on the failed workflow.

2.  Click on **Build & Deploy** block.

3.  Scroll to bottom, find **⛔ ERROR HANDLER** section.

4.  Read the notification line (e.g., *"❌ Error at step [Docker Build]: Code error..."*) to know cause and fix code.

* * * * *

**Summary:**

> **"Code on sub-branch -> Merge to `pre-production` -> Auto to Test Server -> Test Good -> Merge to `main` -> Auto to Real Server."**