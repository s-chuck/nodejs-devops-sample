# Node.js DevOps Sample Project

A simple Node.js web application that I am using to learn and implement end-to-end DevOps practices including CI/CD, containerization, Kubernetes, and GitOps.

## 🚀 Project Goal

To transform a basic Node.js app into a fully automated, production-ready application using modern DevOps tools and workflows.

---

## 🧰 Tech Stack

- **Node.js** (Express web server)
- **Docker** (Multi-stage build)
- **GitHub Actions** (CI/CD pipeline)
- **Kubernetes** (Manifests & deployment)
- **Helm** (Reusable charts for multiple environments)
- **Argo CD** (GitOps continuous delivery)
- **Ingress + DNS** (Expose app with domain mapping)
- **Monitoring** (Prometheus/Grafana - optional future addition)

---

## 💻 Running Locally

```bash
# Install dependencies
npm install

# Start the app
npm start

# Open http://localhost:5000 in your browser


🐳 Running with Docker
# Build the Docker image
docker build -t nodejs-devops-sample .

# Run the container
docker run -p 5000:5000 nodejs-devops-sample



📦 Project Structure
.
├── Dockerfile          # Multi-stage build for Node.js app
├── index.js            # Main server file
├── package.json        # Dependencies and scripts
├── README.md           # You’re reading it


