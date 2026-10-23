
# CI/CD Pipeline for a Web Application

## Project Overview

This project demonstrates an end-to-end CI/CD pipeline using Jenkins, GitHub, Docker, Docker Hub, and AWS EC2.

Whenever application code is pushed to GitHub, the CI/CD pipeline can check out the latest code, install dependencies, run unit tests, build a Docker image, push the image to Docker Hub, and deploy the updated application to an AWS EC2 instance.

The goal is to automate application testing, containerization, and deployment while reducing manual effort.

## Architecture

Developer → GitHub → Jenkins → Unit Tests → Docker Build → Docker Hub → AWS EC2 → Running Web Application

## Technologies Used

- **Source Control:** Git, GitHub
- **CI/CD:** Jenkins
- **Programming Language:** Python
- **Web Framework:** Flask
- **Testing:** Python unittest
- **Containerization:** Docker
- **Image Registry:** Docker Hub
- **Cloud Platform:** AWS EC2
- **Operating System:** Ubuntu Linux
- **Deployment:** SSH and Docker

## How the Project Works

### 1. Code Development

The developer updates the Flask application and its unit tests locally.

### 2. Push Code to GitHub

Changes are committed and pushed to the GitHub repository.

Repository: https://github.com/Vishwa0456/CI-CD-for-Web-Application

### 3. Jenkins Pipeline

Jenkins checks out the repository and executes the pipeline stages defined in the `Jenkinsfile`.

### 4. Install Dependencies

Jenkins creates a Python virtual environment and installs the required packages from `requirements.txt`.

### 5. Run Unit Tests

The pipeline runs the unit tests using:

```bash
python -m unittest -v
```

If the tests fail, the pipeline should stop before building and deploying the application.

### 6. Build Docker Image

Jenkins builds a Docker image for the Flask application and tags it with the Jenkins build number and `latest`.

Example image tags:

```text
vishwanath57/cicd-webapp:<BUILD_NUMBER>
vishwanath57/cicd-webapp:latest
```

### 7. Push Image to Docker Hub

Jenkins authenticates to Docker Hub using a Jenkins credential and pushes the image tags to the registry.

Docker Hub repository:

https://hub.docker.com/r/vishwanath57/cicd-webapp

### 8. Deploy to AWS EC2

Jenkins connects to the EC2 instance through SSH and performs the following operations:

1. Pulls the new Docker image.
2. Stops the existing application container.
3. Removes the old container.
4. Starts a new container using the updated image.
5. Checks whether the container is running.

### 9. Access the Application

Application URL:

http://15.252.236.50:5000

The application returns JSON containing its name, version, and deployment message.

## Jenkins Pipeline Stages

The pipeline includes the following stages:

1. Checkout
2. Install Dependencies
3. Run Unit Tests
4. Build Docker Image
5. Push Docker Image
6. Deploy to EC2

The pipeline also performs Docker logout and reports the final build status.

## Project Structure

```text
cicd-webapp/
├── app.py
├── test_app.py
├── requirements.txt
├── Dockerfile
├── Jenkinsfile
└── README.md
```

## Run the Application Locally

Clone the repository:

```bash
git clone https://github.com/Vishwa0456/CI-CD-for-Web-Application.git
cd CI-CD-for-Web-Application
```

Create and activate a virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Run unit tests:

```bash
python -m unittest -v
```

Start the Flask application:

```bash
python app.py
```

## Run the Application Using Docker

Build the image:

```bash
docker build -t cicd-webapp .
```

Run the container:

```bash
docker run -d \
  --name cicd-webapp \
  --restart unless-stopped \
  -p 5000:5000 \
  cicd-webapp
```

Check the container:

```bash
docker ps
docker logs cicd-webapp
```

## Jenkins Configuration

The Jenkins pipeline uses the following credential IDs:

- `dockerhub-credentials` — Docker Hub authentication.
- `ec2-deploy-key` — SSH access to the EC2 instance.

Configure credentials securely in Jenkins rather than storing passwords, access tokens, or private keys in the repository.

The pipeline runs on the Jenkins server and uses its installed Git, Python, Docker, and SSH Agent capabilities.

## GitHub Integration

GitHub is the source code repository for this project.

Configure the Jenkins job and GitHub webhook or polling mechanism according to your chosen Jenkins job type so that new code changes can trigger the pipeline.

README-only changes can be excluded from expensive build and deployment stages through appropriate pipeline conditions or webhook filtering.

## Troubleshooting

### Jenkins Build Fails

- Review the Jenkins Console Output.
- Verify that the repository URL and branch are correct.
- Check the Jenkins credentials.
- Confirm that the required tools are installed.

### Unit Tests Fail

Run the tests locally:

```bash
python -m unittest -v
```

Fix failing tests before deploying the application.

### Docker Build or Push Fails

Check Docker availability and confirm that the Docker Hub credential is valid.

```bash
docker --version
docker login
```

Avoid entering credentials in scripts or committing them to Git.

### SSH Deployment Fails

Verify that the EC2 instance is running, the SSH key is configured correctly, and the security group permits SSH from the Jenkins server.

### Application Is Not Accessible

Check the container:

```bash
docker ps
docker logs cicd-webapp
```

Verify the application port, EC2 security group rules, and application process.

## Security Considerations

- Store credentials in Jenkins Credentials.
- Restrict SSH access to trusted sources.
- Avoid exposing application ports publicly unless necessary.
- Use HTTPS and a reverse proxy or load balancer for a production deployment.
- Consider health checks and rollback procedures for safer deployments.
- Keep dependencies and container base images updated.

## Future Improvements

- Configure verified GitHub webhook triggering.
- Skip build and deployment stages for README-only changes.
- Add Docker image vulnerability scanning.
- Add application health checks and automated rollback.
- Introduce Terraform for infrastructure provisioning.
- Add Prometheus and Grafana monitoring.
- Configure HTTPS through a reverse proxy or load balancer.

## Project Outcome

The project demonstrates an automated workflow from source code changes to deployment on AWS EC2 using Jenkins and Docker.

The Flask application was successfully updated to version `1.1`, and the pipeline completed its unit tests, image build, image push, and EC2 deployment.

---

**Author:** Vishwanath Veerapur

