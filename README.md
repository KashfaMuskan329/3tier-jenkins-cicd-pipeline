# 3-Tier CI/CD Pipeline with Jenkins & Docker

A full-stack 3-tier application (React, Node.js, MySQL) with a fully automated CI/CD pipeline built using **Jenkins**, **Docker & Docker Compose**, and **AWS EC2**. Every push to GitHub is picked up by Jenkins, which builds and deploys the updated containers to a live server — no manual steps involved.

---

## 🏗️ Architecture

```
GitHub Repo
     │  (push)
     ▼
Jenkins Server (AWS EC2)
     │  1. Checkout latest code
     │  2. SSH into App Server
     ▼
App Server (AWS EC2)
     │  docker compose down
     │  docker compose up -d --build
     ▼
 ┌─────────────┬──────────────┬──────────────┐
 │  React      │  Node.js /   │  MySQL       │
 │  Frontend   │  Express API │  Database    │
 │  (port 3001)│  (port 3000) │  (port 3307) │
 └─────────────┴──────────────┴──────────────┘
```

Two separate EC2 instances are used: one dedicated to running **Jenkins**, and one dedicated to running the **application containers** — mirroring how build and deployment environments are typically separated in real-world DevOps setups.

---

## 🛠️ Tech Stack

- **CI/CD:** Jenkins (Pipeline-as-Code / Jenkinsfile)
- **Containerization:** Docker, Docker Compose
- **Cloud:** AWS EC2 (2 instances), Security Groups
- **Frontend:** React.js
- **Backend:** Node.js, Express
- **Database:** MySQL 5.7
- **Deployment:** SSH-based automated deployment
- **Version Control:** Git & GitHub

---

## ⚙️ How the Pipeline Works

1. Code is pushed to the `main` branch on GitHub.
2. Jenkins job checks out the latest commit.
3. Jenkins connects to the App Server over SSH (using stored credentials).
4. On the App Server, the pipeline pulls the latest code, tears down old containers, and rebuilds/starts new ones with `docker compose`.
5. The updated app is live within minutes — no manual intervention required.

---

## 🧩 Challenges & Solutions

Real issues encountered and resolved while building this pipeline:

- **Jenkins repository signing key had expired.** Jenkins rotated its Debian/Ubuntu package signing key in December 2025; updated the local key to the current `2026` key per Jenkins' official migration guide.
- **Jenkins failed to start after installation.** Root cause was a Java version mismatch — newer Jenkins releases require Java 21, while Java 17 was installed. Resolved by installing OpenJDK 21 and setting it as the default.
- **Jenkins node went offline due to low disk space.** The `/tmp` partition (tmpfs) was too small on a `t3.micro` instance. Resolved by adjusting the Free Disk Space Monitor thresholds for the built-in node.
- **`docker-compose: command not found` during deployment.** The App Server had the newer Docker Compose plugin (`docker compose`, no hyphen) rather than the standalone binary — updated the Jenkinsfile to use the correct syntax.
- **`permission denied` connecting to the Docker daemon.** The deployment user wasn't part of the `docker` group at the time the SSH session started; added the user to the group and rebooted the instance to apply it cleanly.
- **Frontend couldn't reach the backend API.** The frontend had `localhost:3000` hardcoded as the API base URL, which only works when frontend and backend share a browser context. Updated it to point to the App Server's public IP.
- **Backend failing to connect to MySQL (`Access denied for user 'root'`).** The database credentials and database name in `docker-compose.yml` didn't match what the backend code expected. Resolved by explicitly setting matching environment variables (`DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`) for the backend service.
- **React build failing on ESLint config error.** A newer ESLint version flagged an unrecognized key in the inherited `eslint-config-react-app` config. Resolved by disabling the ESLint plugin during the Docker build step.

---

## 🚀 Live Demo

> Note: This is a personal learning/demo deployment running on a free-tier EC2 instance and may not always be live.

Frontend: `http://<app-server-ip>:3001`

---

## 🙏 Credit

This project started from a public starter template for a Dockerized React + Node.js + MySQL app. On top of it, I added:
- A complete Jenkins CI/CD pipeline (Jenkinsfile)
- A two-server AWS EC2 deployment architecture
- SSH-based automated deployment
- Fixed database connectivity, environment configuration, and build issues

---

## 📌 What This Project Demonstrates

- Setting up and troubleshooting Jenkins from scratch on a Linux server
- Writing a working Jenkins Pipeline (Jenkinsfile) for automated deployment
- Configuring AWS EC2, Security Groups, and SSH-based access between servers
- Debugging real Docker, networking, and database connectivity issues
- Understanding the difference between SSH-based deployment and Jenkins master-agent architecture

## Usage

This example serves as a beginner-friendly resource to learn about full-stack Docker containerization in a practical application. It provides a simplified implementation of a full-stack application using React.js, Node.js, and MySQL, all orchestrated with Docker Compose.
