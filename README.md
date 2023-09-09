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

      - name: Build the Docker image
        run: |
          docker build . -t ${{ secrets.DOCKERHUB_USERNAME }}/nginx-agent-github:latest --no-cache -f ./Dockerfile --secret id=nginx-crt,src=./nginx-repo.crt --secret id=nginx-key,src=./nginx-repo.key --build-arg BASE_IMAGE=ubuntu --build-arg PACKAGES_REPO=pkgs.nginx.com
      - name: Push the Docker image
        run: docker push ${{ secrets.DOCKERHUB_USERNAME }}/nginx-agent-github:latest
```

### Step 4: Push the Changes
Commit the new workflow file and push it to your GitHub repository.

### Step 5: Monitor the Workflow
After pushing the changes, navigate to the Actions tab in your GitHub repository to monitor the workflow.



# Automating Kubernetes Deployment with GitHub Actions
After successfully building and pushing your Docker image using GitHub Actions, you can further automate the deployment of this image to a Kubernetes cluster. This section provides a step-by-step guide on how to set this up.

## Prerequisites:
- A running Kubernetes cluster.
- kubectl command-line tool configured to communicate with your cluster.
- A deployment.yaml file in your repository that defines how your application should run in the Kubernetes cluster.

Steps:

## Prepare Your kubeconfig File:

Your kubeconfig file contains the configuration required for kubectl to communicate with your Kubernetes cluster.
For security reasons, remove any sensitive information, especially credentials, from the kubeconfig file.

Add kubeconfig as a GitHub Secret:
- Navigate to your GitHub repository and click on Settings.
- From the left sidebar, select Secrets and variables, then click on Actions.
- Click on New repository secret.
- Name the secret KUBECONFIG and paste the content of your sanitized kubeconfig file as its value.
- Update Your GitHub Actions Workflow:
- Add the following steps to your existing GitHub Actions workflow to set up kubectl and deploy to your Kubernetes cluster:

```
- name: Set up Kubeconfig
  run: echo "${{ secrets.KUBECONFIG }}" > ./kubeconfig
  env:
    KUBECONFIG: ./kubeconfig

- name: Deploy to Kubernetes
  run: kubectl apply -f deployment.yaml
```

## Commit and Push Your Changes:
After updating the workflow, commit the changes and push them to your GitHub repository.

## Monitor the Workflow:
Navigate to the Actions tab in your GitHub repository to monitor the progress of the workflow. You should see the steps for building the Docker image, pushing it to Docker Hub, and deploying it to your Kubernetes cluster.

Important Note:
This setup provides a basic integration of Kubernetes deployment with GitHub Actions. Depending on your specific requirements and the complexity of your application, you might need to make further adjustments.

Handling Kubernetes credentials securely is of utmost importance. Always ensure that sensitive information is not exposed. For more advanced deployment scenarios or enhanced security features, consider using specialized CI/CD tools or Kubernetes operators.

Pending:

```
- name: Set up AWS CLI
      run: |
        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
        unzip awscliv2.zip
        sudo ./aws/install
        aws --version

    - name: Configure AWS credentials
      run: |
        aws configure set aws_access_key_id ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws configure set aws_secret_access_key ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws configure set region your-region-name # e.g., us-west-2

    - name: Update kubeconfig for EKS cluster
      run: |
        aws eks update-kubeconfig --name your-eks-cluster-name

    - name: Deploy to EKS
      run: kubectl apply -f deployment.yaml
```
