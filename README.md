# CI/CD Pipeline for Docker Image with GitHub Actions

This repository contains a CI/CD pipeline that builds a Docker image and pushes it to Docker Hub whenever there's a push to the main branch.

Let's break this down step-by-step, focusing on the CI/CD aspect of building a Docker image from a Dockerfile and then deploying it to a Kubernetes cluster using a manifest file, all while using GitHub and GitHub Desktop for version control.

Prerequisites:
- You have a GitHub repository with a Dockerfile and a Kubernetes deployment manifest.
- You're using GitHub Desktop to manage this repository.
- You have a Docker Hub account (or another container registry) to store your Docker images.
- You have a Kubernetes cluster set up (like AKS, EKS, GKE, or a local one like minikube).

## Step-by-Step Guide:

### Step 1: Version Control with GitHub Desktop:

- Clone Your Repository: If you haven't already, use GitHub Desktop to clone your repository to your local machine.
- Make Changes: Modify your Dockerfile or application code as needed.
- Commit and Push: Using GitHub Desktop, commit your changes and push them to GitHub.

### Step 2: Create GitHub Secrets

- Navigate to your GitHub repository and click on Settings.
- From the left sidebar, select Secrets and variables, then click on Actions.
- Click on New repository secret.
- Create a secret named DOCKERHUB_USERNAME and set its value to your Docker Hub username.
- Create another secret named DOCKERHUB_TOKEN and set its value to your Docker Hub Personal Access Token.


### Step 3: Create the GitHub Actions Workflow

Set Up CI/CD with GitHub Actions:

Create a GitHub Actions Workflow:
In your GitHub repository, navigate to the Actions tab.

Click on New workflow or set up a workflow yourself.
Define the CI/CD Workflow:
You'll be presented with a YAML editor. Here's a basic workflow to build a Docker image and push it to Docker Hub:

```
name: Build and Push Docker Image

on:
  push:
    branches:
      - main  # or your default branch

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      -
        name: Checkout
        uses: actions/checkout@v3
      -
        name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      -
        name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      -
        name: Build the Docker image
        run: |
          docker build . -t ${{ secrets.DOCKERHUB_USERNAME }}/nginx-agent-github:latest --no-cache -f ./Dockerfile --secret id=nginx-crt,src=./nginx-repo.crt --secret id=nginx-key,src=./nginx-repo.key --build-arg BASE_IMAGE=ubuntu --build-arg PACKAGES_REPO=pkgs.nginx.com
      -
        name: Push the Docker image
        run: docker push ${{ secrets.DOCKERHUB_USERNAME }}/nginx-agent-github:latest
```

### Step 4: Push the Changes
Commit the new workflow file and push it to your GitHub repository.

### Step 5: Monitor the Workflow
After pushing the changes, navigate to the Actions tab in your GitHub repository to monitor the workflow.




Automate Kubernetes Deployment:
After pushing the Docker image, you can add steps to deploy it to your Kubernetes cluster. This can be complex, as it requires setting up kubectl in the CI environment and securely handling kubeconfig or other credentials. For simplicity, here's a basic idea:

Copy code
    - name: Set up Kubeconfig
      run: echo "${{ secrets.KUBECONFIG }}" > ./kubeconfig
      env:
        KUBECONFIG: /path/to/kubeconfig  # Adjust this path

    - name: Deploy to Kubernetes
      run: |
        kubectl apply -f deployment.yaml


You'd need to add your kubeconfig file (without sensitive info) as a secret in GitHub, similar to the Docker Hub credentials.

Monitor the Workflow:
After setting up the workflow, every time you push changes to your repository using GitHub Desktop, GitHub Actions will automatically build the Docker image, push it to Docker Hub, and deploy it to your Kubernetes cluster.

You can monitor the progress of these actions in the Actions tab of your GitHub repository.

Note:
This is a basic setup and might need adjustments based on your specific requirements, especially around Kubernetes deployment. Handling Kubernetes credentials securely is crucial. Consider using specialized tools or services for more complex workflows or if you need advanced features and security.



