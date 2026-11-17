# FastAPI Chatbot

A containerized FastAPI chatbot application with automated CI/CD via Jenkins, SonarQube code analysis, Docker Hub publishing, and Kubernetes GitOps deployment.

## Project Structure

```text
fast-api-chatbot/
├── k8s/
│   ├── Deployment.yaml
│   └── Service.yaml
├── static/
├── .env.example
├── .gitignore
├── Dockerfile
├── Jenkinsfile
├── main.py
├── README.md
└── requirements.txt





Ensure the following credentials and tools are configured in Jenkins:
 * docker-cred: Docker Hub username and password/access token.
 * github-cred: GitHub username and Personal Access Token (PAT) with write access to the manifest repository.
 * sonarqube_env: SonarQube server configuration name.
 * sonar-scanner: SonarQube Scanner tool installation name.