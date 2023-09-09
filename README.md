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



# Enhancement #1 Run the Docker image you just pushed on an EC2 instance


To automatically run the Docker image you just pushed on an EC2 instance, you can use GitHub Actions to SSH into the EC2 instance and execute the necessary Docker commands. Here's a step-by-step guide:


Prerequisites:
Docker Installed on EC2: Ensure that Docker is installed and running on your EC2 instance.
Open SSH Port: Ensure that the EC2 instance's security group allows SSH traffic (port 22) from the GitHub Actions runner IP addresses.
Steps:

## Set Up Secrets:

In your GitHub repository, add the following secrets:
EC2_HOST: The public IP address or DNS name of your EC2 instance.
EC2_PEM: The content of your PEM file (private key) for the EC2 instance.

## Update Your Workflow:
Add the following steps to your GitHub Actions workflow:

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

      - name: Deploy Docker image to EC2
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_PEM }}
          script: |
            docker pull ausente/nginx-agent-github:latest
            docker stop $(docker ps -q) || true
            docker run -d -p 8083:80 ausente/nginx-agent-github:latest
```


This workflow does the following:

- SSH into the EC2 instance.
- Pull the latest version of your Docker image.
- Stop any running containers (you can modify this as needed).
- Run the Docker image.
- Run the Workflow:
After updating your workflow with the above steps, commit and push your changes. When the workflow runs, it will SSH into the EC2 instance and deploy the Docker image.

Notes:
The action appleboy/ssh-action@master is a popular GitHub Action for executing remote SSH commands. Ensure you trust any third-party action you use.
The example assumes you're running a web service on port 80 in your Docker container. Adjust the port mapping (-p 8083:80) as needed.
Ensure the EC2 instance has enough resources (CPU, memory) to run the Docker container. Also ensure that you have allowed the necessary ports to be accessed from your Security Group. 
Always handle secrets like the PEM file with care. Using GitHub secrets is a secure way to provide your workflow with necessary credentials without exposing them.



# Enhancement # 2: Container vulnerability checks using Trivy prior to pushing to Dockerhub

Incorporating container vulnerability checks into your CI/CD pipeline is a great practice to ensure the security of your applications. One of the popular tools for this purpose is Trivy by Aqua Security. Here's how you can integrate Trivy with your GitHub Actions workflow to scan Docker images for vulnerabilities:

1. Set Up Trivy in GitHub Actions:
Add Trivy to Your Workflow:
After building your Docker image, you can add a step to scan it using Trivy. Here's a sample workflow snippet:



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
     
      - name: Scan image with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: '${{ secrets.DOCKERHUB_USERNAME }}/nginx-agent-github:latest'
          exit-code: '1'  # This will fail the build if vulnerabilities are found
          ignore-unfixed: 'true'  # This will ignore vulnerabilities without fixes. Set to 'false' if you want to consider them.
          format: 'table'  # The output format of the vulnerabilities. 'table' or 'json'.
          severity: 'CRITICAL,HIGH'  # Only consider vulnerabilities of specified severity.
          
      - name: Push the Docker image
        run: docker push ${{ secrets.DOCKERHUB_USERNAME }}/nginx-agent-github:latest

      - name: Deploy Docker image to EC2
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_PEM }}
          script: |
            docker pull ausente/nginx-agent-github:latest
            docker stop $(docker ps -q) || true
            docker run -d -p 8083:80 ausente/nginx-agent-github:latest
```



# PENDING ENHANCEMENT: Automating Kubernetes Deployment with GitHub Actions
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
