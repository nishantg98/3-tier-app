# Yelp Camp Web Application

This web application allows users to add, view, access, and rate campgrounds by location. It is based on "The Web Developer Bootcamp" by Colt Steele, but includes several modifications and bug fixes. The application leverages a variety of technologies and packages, such as:

- **Node.js with Express**: Used for the web server.
- **Bootstrap**: For front-end design.
- **Mapbox**: Provides a fancy cluster map.
- **MongoDB Atlas**: Serves as the database.
- **Passport package with local strategy**: For authentication and authorization.
- **Cloudinary**: Used for cloud-based image storage.
- **Helmet**: Enhances application security.
- ...

## Application Screenshots
![](./images/home.jpg)
![](./images/campgrounds.jpg)
![](./images/register.jpg)

# 3-Tier Full Stack Project

## Project Overview
This project demonstrates a comprehensive 3-tier full stack application deployment using Azure DevOps for CI/CD, Docker for containerization, and Amazon EKS for orchestration. The application is built with Node.js, and it leverages Cloudinary for image storage, Mapbox for mapping, and MongoDB Atlas for database management. The project includes a robust deployment pipeline, static code analysis, and security scanning.

![Screenshot from 2024-07-30 17-09-15](https://github.com/user-attachments/assets/d4492e78-5573-42d5-8075-23dcf9a3c37c)

## Installation
To get started with the project, clone the repository and follow the steps below:

```sh
git clone <repository_url>
cd <repository_directory>
```

## Setting Up Azure DevOps

### Create a New Project
1. Go to Azure DevOps.
2. Create a new project.
3. Set up a new pipeline and connect it to your GitHub repository.

### Pipeline Configuration
1. Navigate to Pipelines > Create Pipeline.
2. Select your repository and configure the pipeline using the classic editor

## Installing Docker
Docker Installation Script

```sh
#!/bin/bash
# Update package manager repositories
sudo apt-get update
# Install necessary dependencies
sudo apt-get install -y ca-certificates curl
# Create directory for Docker GPG key
sudo install -m 0755 -d /etc/apt/keyrings
# Download Docker's GPG key
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
# Ensure proper permissions for the key
sudo chmod a+r /etc/apt/keyrings/docker.asc
# Add Docker repository to Apt sources
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# Update package manager repositories
sudo apt-get update
# Install Docker
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
# Allow Docker to be run without sudo
sudo chmod 666 /var/run/docker.sock
```

## Setting Up SonarQube
1. Ensure Docker is installed using the script above.
2. Run the following command to start SonarQube:
```sh
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```
![Screenshot from 2024-07-30 13-35-24](https://github.com/user-attachments/assets/45d8c116-fc91-428d-af2e-4393aa9a95e6)

## Setting Up EKS

AWS CLI Installation
```sh
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws configure
```

kubectl Installation
```sh
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```
eksctl Installation
```sh
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

## Creating EKS Cluster
```sh
eksctl create cluster --name=my-eks22 --region=ap-south-1 --zones=ap-south-1a,ap-south-1b --without-nodegroup
eksctl utils associate-iam-oidc-provider --region ap-south-1 --cluster my-eks22 --approve
eksctl create nodegroup --cluster=my-eks22 --region=ap-south-1 --name=node2 --node-type=t3.medium --nodes=3 --nodes-min=2 --nodes-max=4 --node-volume-size=20 --ssh-access --ssh-public-key=Key --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access
```
![Screenshot from 2024-07-24 12-55-11](https://github.com/user-attachments/assets/b5267b00-ca32-47e9-b6a2-c47bed975033)

## Local Execution
Connect to the machine using ssh-key in Mobaxterm:
1. Clone the repository:
```sh
git clone <repository_url>
cd <repository_directory>
```

2. Install Node.js using NVM:
```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
nvm install 20
node -v # should print `v20.12.2`
npm -v # should print `10.5.0`
```

3. Obtain API keys from Cloudinary, Mapbox, and MongoDB Atlas.
4. Create a .env file and add the following lines with your actual values:
```sh
CLOUDINARY_CLOUD_NAME=[Your Cloudinary Cloud Name]
CLOUDINARY_KEY=[Your Cloudinary Key]
CLOUDINARY_SECRET=[Your Cloudinary Secret]
MAPBOX_TOKEN=[Your Mapbox Token]
DB_URL=[Your MongoDB Atlas Connection URL]
SECRET=[Your Chosen Secret Key]
```

5. Install project dependencies:
```sh
npm install
```

6. Start the application:
```sh
npm start
```

7. Access the app at http://VM_IP:3000 (replace VM_IP with the IP address of your Ubuntu machine).

![Screenshot from 2024-07-30 11-40-43](https://github.com/user-attachments/assets/887f980a-8471-4c03-a953-e695d32f2e84)

![Screenshot from 2024-07-30 11-42-55](https://github.com/user-attachments/assets/4abd2d7e-1658-45ae-8706-7cf5b9790e82)

![Screenshot from 2024-07-30 11-44-54](https://github.com/user-attachments/assets/d7173164-2b28-48e3-8dd6-b5f337908d05)


## Dev Deployment Pipeline

### Azure DevOps Pipeline Configuration
```sh
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  SCANNER_HOME: '/usr/share/sonar-scanner'

steps:
- task: NodeTool@0
  inputs:
    versionSpec: '20.x'
  displayName: 'Install Node.js'

- script: npm install
  displayName: 'Install Package Dependencies'

- script: npm test
  displayName: 'Run Unit Tests'

- script: trivy fs --format table -o fs-report.html .
  displayName: 'Run Trivy FS Scan'

- task: SonarQubePrepare@5
  inputs:
    SonarQube: 'sonar'
    scannerMode: 'CLI'
    configMode: 'manual'
    cliProjectKey: 'Campground'
    cliProjectName: 'Campground'

- task: SonarQubeAnalyze@5
  displayName: 'Run SonarQube Analysis'

- script: |
    docker build -t adijaiswal/camp:latest .
    docker push adijaiswal/camp:latest
  displayName: 'Build and Push Docker Image'

- script: trivy image --format table -o fs-report.html adijaiswal/camp:latest
  displayName: 'Run Trivy Image Scan'

- script: |
    docker run -d -p 3000:3000 adijaiswal/camp:latest
  displayName: 'Deploy Docker Image Locally'
```
![Screenshot from 2024-07-30 14-05-41](https://github.com/user-attachments/assets/039884b5-10db-4a73-8bb8-10c67338dc2d)

## Production Deployment Pipeline
### Azure DevOps Pipeline Configuration
```sh
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  SCANNER_HOME: '/usr/share/sonar-scanner'

steps:
- task: NodeTool@0
  inputs:
    versionSpec: '20.x'
  displayName: 'Install Node.js'

- script: npm install
  displayName: 'Install Package Dependencies'

- script: npm test
  displayName: 'Run Unit Tests'

- script: trivy fs --format table -o fs-report.html .
  displayName: 'Run Trivy FS Scan'

- task: SonarQubePrepare@5
  inputs:
    SonarQube: 'sonar'
    scannerMode: 'CLI'
    configMode: 'manual'
    cliProjectKey: 'Campground'
    cliProjectName: 'Campground'

- task: SonarQubeAnalyze@5
  displayName: 'Run SonarQube Analysis'

- script: |
    docker build -t adijaiswal/campa:latest .
    docker push adijaiswal/campa:latest
  displayName: 'Build and Push Docker Image'

- script: trivy image --format table -o fs-report.html adijaiswal/campa:latest
  displayName: 'Run Trivy Image Scan'

- script: |
    kubectl apply -f Manifests/
    sleep 60
  displayName: 'Deploy to EKS'

- script: |
    kubectl get pods -n webapps
    kubectl get svc -n webapps
  displayName: 'Verify Deployment'
```
![Screenshot from 2024-07-30 16-33-32](https://github.com/user-attachments/assets/38c66754-bfc9-4602-a5f2-ab3c3e5228a1)

## Environment Variables
Ensure you have the following environment variables configured in your .env file:

```sh
CLOUDINARY_CLOUD_NAME=[Your Cloudinary Cloud Name]
CLOUDINARY_KEY=[Your Cloudinary Key]
CLOUDINARY_SECRET=[Your Cloudinary Secret]
MAPBOX_TOKEN=[Your Mapbox Token]
DB_URL=[Your MongoDB Atlas Connection URL]
SECRET=[Your Chosen Secret Key]
```
License
This project is licensed under the MIT License. See the LICENSE file for details.

Feel free to contribute to the project by forking the repository and submitting pull requests. For any issues or feature requests, please open an issue in the repository.

