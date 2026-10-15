# HealthHeart™ | LeadlessGuard™

<div align="center">

![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)
![React](https://img.shields.io/badge/React-18.3-61dafb?logo=react)
![TypeScript](https://img.shields.io/badge/TypeScript-5.6-3178c6?logo=typescript)
![Vite](https://img.shields.io/badge/Vite-5.4-646cff?logo=vite)
![Node](https://img.shields.io/badge/Node-Express-339933?logo=node.js)
![MongoDB](https://img.shields.io/badge/MongoDB-Atlas%20Ready-47A248?logo=mongodb)
![Tests](https://img.shields.io/badge/Verification%20Suite-25%2B%20Passing-success)

**Enterprise-Grade Cardiac Telemetry, Electrophysiology & Leadless Pacemaker Research Simulator**

[Quickstart](#-quickstart-guide) • [Architecture](#-system-architecture--trust-boundaries) • [Feature Tour](#-platform-capabilities) • [Documentation](docs/) • [Security Policy](SECURITY.md)

</div>

---

> [!IMPORTANT]
> ### ⚠️ Safety & Research Scope Boundary
> **HealthHeart™ / LeadlessGuard™ is an engineering simulation platform for digital health research, signal visualization, reproducible public-data exploration, and controlled cybersecurity analysis.**  
> - **No Real Therapeutic Device Commands:** Contains **no real implant firmware, clinical programmer protocols, or physical BLE characteristics capable of pacing a human heart**. Real implanted pacemaker programming is outside the scope of this project.
> - **Non-Diagnostic:** Not approved by the FDA, CE, or any regulatory body for clinical diagnostic, therapeutic, or monitoring use.

---

## 📖 Overview

**HealthHeart™ / LeadlessGuard™** models an end-to-end telemetry architecture for a next-generation smartphone-integrated leadless cardiac pacemaker (*LeadlessGuard VR-Pro*). Built with **React 18**, **TypeScript**, and **Express.js**, the platform simulates electrophysiological signal dynamics, device sensing/pacing logic, energy consumption, cryptographic integrity, and clinician-patient telemetry interfaces.

## 🚀 Deploy on DigitalOcean with Docker

The repository includes a production Docker Compose setup containing the React frontend, Express API, and a private MongoDB container. The database is persisted in a Docker volume and is not exposed to the public internet.

### 1. Create a Droplet

In DigitalOcean, create an **Ubuntu 24.04 LTS** Droplet. The smallest Basic Droplet is enough for a demonstration deployment. Add your SSH key during creation and create a firewall that allows **SSH (22)**, **HTTP (80)**, and **HTTPS (443)**.

### 2. Install Docker and download the project

Connect to the Droplet over SSH, then run:

```bash
sudo apt update
sudo apt install -y ca-certificates curl git
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker "$USER"
newgrp docker
git clone https://github.com/KotgireOmkar/healthheart-leadlessguard.git
cd healthheart-leadlessguard
```

### 3. Start the complete application

```bash
docker compose up -d --build
docker compose ps
```

Open the Droplet IP address in a browser. The first startup can take a few minutes because the images and dependencies must be downloaded.

Useful commands:

```bash
docker compose logs -f
docker compose logs -f server
docker compose restart
docker compose down
```

Do not run `docker compose down -v` unless you intentionally want to delete the MongoDB data volume.

### 4. Add a domain and HTTPS

Create an `A` record at your domain provider pointing to the Droplet IP, then use a reverse proxy such as Caddy or DigitalOcean's documented HTTPS setup to terminate TLS on port 443 and forward traffic to port 80. Do not expose MongoDB publicly.

This deployment is for the project's research simulator and does not make it a clinical device or diagnostic service. Configure backups and HTTPS before putting any non-public data on the server.

## ☁️ Deploy on Heroku

Heroku can run the complete application as one web app. This repository includes a `Procfile`; the Heroku build compiles both the React frontend and Express server, and Express serves the frontend.

Heroku does not provide the local MongoDB container used by Docker Compose. Create a MongoDB Atlas database, allow the Heroku connection (for a quick demo you can temporarily allow `0.0.0.0/0`), and copy the Atlas connection string.

Install the Heroku CLI, sign in, and run these commands from the repository root:

```bash
heroku login
heroku create your-healthheart-app
heroku config:set NODE_ENV=production
heroku config:set MONGODB_URI="mongodb+srv://USER:PASSWORD@CLUSTER/healthheart?retryWrites=true&w=majority"
git push heroku main
heroku open
```

If your Git branch is not named `main`, use:

```bash
git push heroku HEAD:main
```

View logs with:

```bash
heroku logs --tail
```

Heroku hosting is billed according to the selected dyno plan; check the current [Heroku pricing](https://www.heroku.com/pricing) before creating the app. The app is a research simulator and is not approved for clinical diagnosis or therapy.

## 🌐 Deploy on Render

Render is another simple option and can deploy this repository directly from GitHub. The included `render.yaml` creates a Node web service, builds the React frontend and Express API, and checks `/api/health`.

1. Create a MongoDB Atlas database and copy its connection string.
2. In the [Render dashboard](https://dashboard.render.com/), choose **New → Blueprint**.
3. Connect the GitHub repository and select the branch containing `render.yaml`.
4. When Render asks for `MONGODB_URI`, paste the MongoDB Atlas connection string as a secret.
5. Create the service and open the generated `onrender.com` URL when the deployment finishes.

The free Render plan is suitable for a demo but may pause after inactivity, so the first request after a pause can take longer. Upgrade the web service if the application must stay continuously available. Do not put MongoDB credentials in source code.

### Core Objectives
1. **Electrophysiological Fidelity:** High-rate 4-track biosignal oscilloscope (ECG, PPG, SpO2, BIA) with deterministic synthetic waveform generation and PhysioNet MIT-BIH Arrhythmia database exploration.
2. **Transparent Pacing Emulation:** Mathematical VVI and VVIR pacing logic modeling refractory periods (250 ms), sensitivity thresholds (1.5 mV), and Lower Rate Limits (60 BPM).
3. **Cybersecurity & Trust Boundaries:** Demonstration of STRIDE threat modeling, BLE 5.3 cryptographic replay protection, and SHA-256 tamper-evident audit logging.
4. **Clinical & Patient Workstations:** Multi-patient clinician electrophysiology dashboard contrasted with an empathetic, calming patient mobile companion view.

---

## 🏛 System Architecture & Trust Boundaries

HealthHeart enforces a strict **Three-Domain Trust Architecture** to model isolation between embedded device firmware, local wireless gateways, and enterprise hospital applications:

```mermaid
graph TD
    subgraph "Domain 1: Leadless Pacemaker Core (Embedded Emulation)"
        FW["Virtual Firmware Engine\n(Model: LeadlessGuard VR-Pro | FW: v3.4.1)"]
        SENSE["Ventricular Sense Amplifier\n(Sensitivity: 1.5 mV | Refractory: 250 ms)"]
        VVI["VVI / VVIR Pacing Logic\n(LRL: 60 BPM | URL: 120 BPM)"]
        DIAG["Device Diagnostics\n(Li/CFx 240 mAh | Impedance: 580Ω | Temp: 37.1°C)"]
    end

    subgraph "Domain 2: Telemetry Gateway (Simulated BLE 5.3)"
        PACKET["Telemetry Serialization\n(ECG, PPG, SpO2, BIA, Diagnostics)"]
        CRYPTO["Cryptographic Pipeline\n(ECDH Key Exchange | AES-GCM-256)"]
        REPLAY["Replay Protection Filter\n(Monotonic Counters | 5000ms Expiry Window)"]
        GATE["Strict Read-Only Enforcement\n(Rejects Direct Pacing Modification)"]
    end

    subgraph "Domain 3: Enterprise Application Layer"
        ADMIN["Clinician EP Workstation\n(Multi-Patient Tabs: PAT-001 to PAT-005)"]
        PATIENT["Patient Companion View\n(Calming UI | Battery Gauge | Hotline)"]
        SIGNALS["Synchronized 4-Track Oscilloscope\n(ECG with Calipers, PPG, SpO2, BIA)"]
        IMPULSE["5-Step Cardiac Conduction Visualizer\n(Intrinsic → Sense → Decision → Pulse → Evoked)"]
        SEC["Cybersecurity Center\n(Interactive 10-Threat Attack Console)"]
        POWER["Battery Duty-Cycle Model\n(Li/CFx Chemistry | Longevity Estimator)"]
        TESTS["Verification Suite\n(25+ Automated Safety & Functional Tests)"]
    end

    subgraph "Persistence Layer"
        ATLAS[("MongoDB Atlas Database\n(Patients, Telemetry, Alerts, Audit Ledger)")]
    end

    FW --> SENSE
    SENSE --> VVI
    VVI --> DIAG
    DIAG --> PACKET
    PACKET --> CRYPTO
    CRYPTO --> REPLAY
    REPLAY --> GATE
    GATE --> ADMIN
    GATE --> PATIENT
    ADMIN --> ATLAS
```

---

## ⚡ Platform Capabilities

| Module | Features & Technical Highlights |
| :--- | :--- |
| **Synchronized 4-Track Oscilloscope** | Real-time rendering of ECG (Lead II), PPG (photoplethysmogram), SpO2, and thoracic Bioimpedance (BIA) with interactive millivolt/millisecond caliper measurement tools. |
| **5-Step Cardiac Impulse Engine** | Interactive anatomical visualization tracking electrophysiological conduction: *Intrinsic Depolarization → Electrode Sensing → Microcontroller Decision → Output Pulse (2.5V @ 0.4ms) → Evoked Response*. |
| **VVI / VVIR Pacing Simulator** | Deterministic state machine modeling intrinsic inhibition, escape intervals, magnet mode (async VOO @ 85 BPM), rate response, and battery-depletion fallback. |
| **Cybersecurity Attack Console** | Interactive simulation of **10 STRIDE attack vectors** (BLE replay attacks, MITM packet tampering, RF battery-depletion jamming, telemetry spoofing) with real-time defensive rejection telemetry. |
| **Battery Duty-Cycle Longevity Model** | Accurate mathematical simulation of Lithium-Carbon Monofluoride (Li/CFx 240 mAh) battery drain across sleep (8 µA), sensing (45 µA), pacing bursts (1.2 mA), and RF transmissions (12 mA). |
| **AI Rhythm Analytics & Benchmark** | 9 cardiac rhythm classification patterns (Normal Sinus, Bradycardia, Tachycardia, PVCs, AFib, Asystole) with transparent confidence scoring and external related-work benchmark citation. |
| **Verification & Safety Suite** | Automated test suite validating 25+ safety invariants, pacing limits, cryptographic expiry windows, and alert thresholds with PASS/FAIL/BLOCKED reporting. |
| **Patient Companion Mobile View** | Empathetic patient-facing interface with intuitive device status, calming color palettes, pacing battery gauges, and single-tap emergency assistance. |

---

## 📁 Repository Structure

```text
healthheart-leadlessguard/
├── .github/workflows/          # Automated GitHub Actions CI workflow
├── docs/                       # 21 Engineering architecture & specification docs
│   ├── 01_system_architecture.md ... 20_future_development_roadmap.md
│   └── specifications/         # Original Prompt Packs & Track documentation
├── server/                     # Telemetry & Audit Express/Mongoose backend
│   ├── src/
│   │   ├── config/             # Database connection management
│   │   ├── models/             # AuditRecord, ClinicalAlert, Patient, Telemetry
│   │   ├── routes/             # REST API route handlers
│   │   ├── seed.ts             # Initial patient & telemetry seed generator
│   │   └── server.ts           # Server bootstrap
│   ├── .env.example            # Environment configuration template
│   └── package.json
├── src/                        # React 18 + Vite Frontend Application
│   ├── components/             # ECGCanvas, FaultInjectionPanel, CardiacImpulsePanel
│   ├── panels/
│   │   ├── admin/              # 11 Clinician workstation dashboards
│   │   └── user/               # Patient Companion smartphone interface
│   ├── services/               # Pacing engine, math signal generators, API client
│   ├── types/                  # Strict TypeScript domain interfaces
│   ├── App.tsx                 # View state coordinator
│   └── index.css               # Clinical dark/light theme design system
├── .gitignore                  # Git secrets and build exclusion rules
├── LICENSE                     # MIT License
├── README.md                   # Repository documentation
└── SECURITY.md                 # Security boundary & reporting policy
```

---

## 🚀 Quickstart Guide

### 1. Prerequisites
- **Node.js**: v18.x or higher
- **npm**: v9.x or higher
- **MongoDB**: Local MongoDB instance or free [MongoDB Atlas Cluster](https://www.mongodb.com/cloud/atlas)

### 2. Clone & Install Dependencies
```bash
git clone https://github.com/<your-username>/healthheart-leadlessguard.git
cd healthheart-leadlessguard

# Install dependencies for both frontend and backend
npm run install:all
```

### 3. Configure Environment Variables
Copy the server environment template:
```bash
cp server/.env.example server/.env
```
Edit `server/.env` with your settings (optional; the app falls back gracefully to in-memory mock data if MongoDB is disconnected):
```env
PORT=5000
MONGODB_URI=mongodb+srv://<username>:<password>@cluster.mongodb.net/healthheart?retryWrites=true&w=majority
NODE_ENV=development
```

### 4. Run the Development Server
Launch both client and server concurrently:
```bash
npm run dev
```
- **Web Application:** `http://localhost:5173`
- **Telemetry API:** `http://localhost:5000`

---

## 📡 Backend API Reference

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `GET` | `/api/health` | System health check and database connectivity status |
| `GET` | `/api/patients` | Retrieve list of synthetic monitored patients (`PAT-001` - `PAT-005`) |
| `GET` | `/api/telemetry/live` | Stream current multi-track telemetry packet |
| `GET` | `/api/alerts` | Retrieve active clinical alerts (Tier-1 Critical to Tier-3 Advisory) |
| `GET` | `/api/audit` | Fetch immutable audit ledger with cryptographic SHA-256 chain verification |
| `POST`| `/api/scenarios/run/:code` | Execute an automated verification test scenario |

---

## 🧪 Verification & Automated Testing

HealthHeart includes a built-in automated verification suite accessible via the **Verification Suite** tab in the admin workstation, as well as command-line compilation checks:

```bash
# Typecheck & build frontend
npm run build

# Typecheck & build backend
npm --prefix server run build
```

---

## 🔬 Academic Citations & Related Work

HealthHeart draws conceptual inspiration and signal standards from the following open biomedical engineering resources:

- **PhysioNet MIT-BIH Arrhythmia Database:** Gold-standard annotated ambulatory ECG recordings used for validation. [PhysioNet MIT-BIH Record](https://physionet.org/content/mitdb/1.0.0/)
- **HeartGPT (Davies et al., 2024):** *Interpretable Pre-Trained Transformers for Heart Time-Series Data* ([arXiv:2407.20775](https://arxiv.org/abs/2407.20775)). Cited as related benchmark work in the Research Workspace. *Note: HealthHeart uses deterministic simulation and does not bundle or execute learned transformer model checkpoints.*

---

## 📄 License

Distributed under the **MIT License**. See [`LICENSE`](LICENSE) for more information.
