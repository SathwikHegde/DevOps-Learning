# CI CD Tutorial - gitLabs

# GitLab CI/CD Pipeline for a Python Application 🚀

This repository demonstrates how to build a full **GitLab CI/CD pipeline** for a Python application. The pipeline includes **testing, building a Docker image, pushing it to a private Docker registry, and deploying it to a remote server on DigitalOcean**.

## 📌 What You'll Learn in 1 Hour
- **What GitLab CI/CD is**
- **Comparison of GitLab with other CI/CD platforms**
- **GitLab Architecture Overview**
- **Setting up a GitLab CI/CD Pipeline**
  - Running automated tests
  - Building a Docker image
  - Pushing the image to a private Docker registry
  - Deploying the application to a remote server
- **Key GitLab Concepts:**
  - Pipelines & Jobs
  - Stages
  - GitLab Runners & Executors
  - Variables (Variable & File Type)
  - Docker-in-Docker (DinD) for containerized CI/CD workflows

---

## 📂 Repository Structure

```plaintext
├── .gitlab-ci.yml      # GitLab CI/CD configuration file
├── .gitignore          # Ignored files for version control
├── app/                # Python application code
│   ├── main.py         # Main application script
│   ├── requirements.txt# Python dependencies
│   └── tests/          # Unit tests
├── Dockerfile          # Docker container setup
├── deploy/             # Deployment scripts
└── README.md           # Project documentation

