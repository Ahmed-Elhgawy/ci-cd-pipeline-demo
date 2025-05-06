# 🚀 CI/CD Pipeline with Jenkins for Todo App

This repository contains a **Jenkins pipeline** that automates the CI/CD process for a Node.js-based **Todo App**.

---

## 🔧 What It Does

The Jenkins pipeline performs the following:

1. ✅ Clones the source code from GitHub
2. ✅ Installs dependencies (`npm ci`)
3. ✅ Runs tests (Jest + Supertest)
4. ✅ Builds a Docker image
5. ✅ Pushes the image to Docker Hub
6. ✅ Deploys to a EC2 instance via SSH

---

## 📁 Project Structure

```bash
.
├── Jenkinsfile       # Defines the pipeline stages
└── README.md
```

---

## 🛠️ Tools Used

- Jenkins
- Docker
- Node.js / Express / Mongoose
- Jest / Supertest
- Docker Hub
- SSH for deployment

## 🔐 Required Jenkins Credentials

- `docker-hub`: __Docker Hub__ username & password
- `gethub-private-key`: SSH key for Authentication with __Github__ repo
- `mongo_username`: MongoDB username 
- `mongo_password`: MongoDB password
- `aws-credentials`: __ACCESS KEY__ and __SECRET ACCESS KEY__ for Authentication with __AWS__
- `devops-linux-private-key`: SSH key for deployment server (EC2 instance)

---

## 🙌 Author

Ahmed Elhgawy – [GitHub](https://github.com/Ahmed-Elhgawy) | [LinkedIn](https://linkedin.com/in/ahmed-mahmoud-a16310268)
