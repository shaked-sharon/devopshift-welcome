# DevOps | Defense Project | Docker & Kubernetes — Crypto Price Tracker

## Project Overview

DevOps Defense Project — containerizes a Python Frontend (FE) & Backend (BE) App with Docker, orchestrates services using Docker Compose, deploys to Kubernetes (K8), & packages deployments as Helm Charts  
Source Code forked from instructor Repo: `yanivomc/devopshift-welcome` > Branch `workshop/k8s-docker-exam`

**App:** Crypto Price Tracker — Fetches live Bitcoin prices via API & saves to MySQL DB  
**Frontend port:** `5002`  
**Backend port:** `5001`  
**Database:** MySQL 8.0

> **NOTE:** Changed MySQL version from `5.7` > `8.0` — MySQL:5.7 doesn't support Apple Silicon (M1/M2/M3). **For testing/student project use ONLY!!** _Never_ use this in prod!!  
> **Flask dev server warning** — FE & BE run Flask in debug mode. **For testing/student project use ONLY!!** _Never_ use this in prod!!  
> **Passwords in yaml files** — MySQL root password hardcoded per exam instructions. **Never** do this in prod — use K8 secrets / secrets manager!!

---

## What This Project Does

1. **Docker Containerization**
   - Dockerfiles for FE & BE > packages each into `python:3.12-slim` container
   - Custom network `crypto-net` for internal service communication
   - Docker Compose orchestrates all 3 services: FE, BE, MySQL DB

2. **Kubernetes Deployment**
   - Deploys FE, BE & MySQL as separate K8 Deployments
   - FE & BE run 2 replicas each > MySQL runs 1 replica
   - All services use ClusterIP > access FE via `kubectl port-forward`
   - Environment variables passed to BE & MySQL via deployment yaml

3. **Helm Charts (Bonus)**
   - Helm Charts for FE, BE & MySQL
   - Each chart contains `Chart.yaml`, `values.yaml` & templates folder
   - `values.yaml` supports config changes > replicas, ports, DB credentials

---

## Current Implementation

- fe Dockerfile: `python:3.12-slim` > installs deps > copies src > exposes 5002
- be Dockerfile: `python:3.12-slim` > installs deps > copies src > exposes 5001
- docker-compose.yaml: 3 services on custom bridge network `crypto-net`
- MySQL 8.0 (changed from 5.7 for Apple Silicon use)
- k8s folder: 6 yaml files > fe/be/mysql deployments & services (all ClusterIP)
- helm folder: 3 charts > fe, be, mysql > each with values.yaml & templates
- fe & be tested & verified working in both Docker Compose & Kubernetes

---

## Setup & How to Run

### Prereqs
1. **Docker Desktop** Installed & running
2. **Kubernetes** Enabled in Docker Desktop > Settings > Kubernetes > Enable
3. **kubectl** Installed & cluster running (`kubectl get nodes`)
4. **Helm** Installed (`helm version`)

---

### Section 1 | Docker Compose

1. **Go to docker folder**
   ```
   cd exam-code/docker
   ```
2. **Build & start all services**
   ```
   docker-compose up --build
   ```
3. **Open browser**
   ```
   http://localhost:5002
   ```
4. **Click Fetch Prices** > verify Bitcoin price shows & `Saved to database: true`
5. **Stop & remove containers when done**
   ```
   docker-compose down
   ```

---

### Section 2 | Kubernetes

1. **Go to project root**
   ```
   cd /path/to/devops-defense-project
   ```
2. **Build Docker images local** (k8s pulls from local)
   ```
   docker build -t crypto-fe:latest ./exam-code/docker/fe
   docker build -t crypto-be:latest ./exam-code/docker/be
   ```
3. **Deploy all k8s resources**
   ```
   kubectl apply -f k8s/
   ```
4. **Check pods are running**
   ```
   kubectl get pods
   kubectl get svc
   ```
5. **Access FE via port-forward**
   ```
   kubectl port-forward service/fe-service 5002:5002
   ```
6. **Open browser** > `http://localhost:5002` > click Fetch Prices
7. **Tear down when done**
   ```
   kubectl delete -f k8s/
   ```

---

### Section 3 | Helm (Bonus)

1. **Remove existing k8s deployments first** (if running)
   ```
   kubectl delete -f k8s/
   ```
2. **Install Charts** > mysql first, then be, then fe (due to each needing the first in order to execute correctly)
   ```
   helm install mysql ./helm/mysql
   helm install be ./helm/be
   helm install fe ./helm/fe
   ```
3. **Check pods are running**
   ```
   kubectl get pods
   ```
4. **Access FE via port-forward**
   ```
   kubectl port-forward service/fe-service 5002:5002
   ```
5. **Open browser** > `http://localhost:5002` > click Fetch Prices
6. **Uninstall charts when done**
   ```
   helm uninstall fe be mysql
   ```

---

## Files Explained

- **exam-code/docker/fe/Dockerfile** – packages fe into python:3.12-slim container > exposes 5002
- **exam-code/docker/be/Dockerfile** – packages be into python:3.12-slim container > exposes 5001
- **exam-code/docker/docker-compose.yaml** – runs fe, be & mysql on custom network crypto-net
- **exam-code/docker/fe/src/main.py** – fe Flask app > renders page & calls be API
- **exam-code/docker/be/src/main.py** – be Flask app > fetches crypto prices > saves to mysql
- **exam-code/docker/fe/src/templates/index.html** – frontend webpage > Fetch Prices button
- **k8s/fe-deployment.yaml** – k8s deployment for fe > 2 replicas
- **k8s/fe-service.yaml** – ClusterIP service for fe > port 5002
- **k8s/be-deployment.yaml** – k8s deployment for be > 2 replicas > mysql env vars
- **k8s/be-service.yaml** – ClusterIP service for be > port 5001
- **k8s/mysql-deployment.yaml** – k8s deployment for mysql > 1 replica > db env vars
- **k8s/mysql-service.yaml** – ClusterIP service for mysql > port 3306
- **helm/fe/** – Helm chart for fe > Chart.yaml, values.yaml, templates/
- **helm/be/** – Helm chart for be > Chart.yaml, values.yaml, templates/
- **helm/mysql/** – Helm chart for mysql > Chart.yaml, values.yaml, templates/
- **.gitignore** – ignores pycache, venv, secrets, DS_Store
- **README.md** – this file :)

---

## Education Goals

This project demonstrates:
- Docker containerization for multi-service Python app
- Docker Compose orchestration with custom networking
- K8 deployments, services & replica management
- ClusterIP services & kubectl port-forward > local access
- Environment variable management in K8 deployments
- Helm chart creation with configurable values.yaml
- Git workflow using forked repo & feature branch

---

## Project Sections Completed

**Section 1 | Docker**
- Dockerfiles for FE (port 5002) & BE (port 5001) using Python:3.12-slim
- docker-compose.yaml with 3 services on custom bridge network
- Tested & verified > Bitcoin price fetched & saved to DB

**Section 2 | Kubernetes**
- 6 k8s yaml files > deployments & ClusterIP services for fe, be, mysql
- FE & BE running 2 replicas each
- Accessed via kubectl port-forward > verified working in browser
- FE service type changed to ClusterIP - per instructor (not LoadBalancer)

**Section 3 | Helm Charts (Bonus)**
- Helm charts for FE, BE & MySQL
- values.yaml supports replica count, image, port & db credential changes
- Deployed & verified working via port-forward

**Section 4 | Istio** — excluded per instructor

---

## Author

**Sharon Shaked (shaked-sharon)**
- **Email:** sharon.shaked@icloud.com / sharon.shaked24@gmail.com
- **GitHub:** https://github.com/shaked-sharon

---

## License

This project is for **educational purposes only** & is part of a **DevOps Program** defense project demonstrating use of Docker & K8 specifically with Helm Charts as optional addition
