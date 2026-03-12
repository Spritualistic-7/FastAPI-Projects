
# Insurance Premium Risk Classifier — FastAPI on AWS (Docker + EC2)

Production-style deployment of an ML model (Random Forest) that predicts whether an insurance premium category will be **High**, **Medium**, or **Low** based on user inputs. Built with **FastAPI**, containerized with **Docker**, and deployed on **AWS EC2**.

> **Note:** Endpoint currently runs over **HTTP** (AWS Free Tier). Some corporate networks may block non‑HTTPS endpoints.

---

## 🧭 Project Overview
This project operationalizes a trained Random Forest classifier behind a REST API. It demonstrates end‑to‑end skills across **ML integration**, **API development**, **containerization**, and **cloud deployment** with a focus on **production readiness** even within Free Tier constraints.

**What it does:**
- Serves `/predict` for model inference
- Exposes `/health` to report service and **MODEL_VERSION**
- Uses **Pydantic** schemas for strict input validation
- Implements **rate limiting** middleware and **request/response logging** middleware

---

## 🧱 Architecture
```
Client → FastAPI (Uvicorn) → Model (RandomForest)
      ↘ Logging + Rate Limiting Middleware
Docker Container → AWS EC2 Instance → Security Group (inbound rule for API port)
```

### High-level flow
1. Client sends request to FastAPI endpoint
2. Pydantic validates input schema
3. `predict.py` loads model artifact and returns class (High/Medium/Low)
4. Logging middleware records request, response, and latency
5. Rate limiter protects from abuse (HTTP 429 on burst traffic)

---

## 🧰 Tech Stack
- **Language/Framework:** Python, FastAPI, Uvicorn
- **ML:** Random Forest (pre‑trained binary)
- **Packaging:** Docker
- **Cloud:** AWS EC2 (Free Tier)
- **Observability:** App‑level logging (rotating files); optional CloudWatch Agent for system metrics

---

## 📁 Repository Structure
```
.
├─ app.py
├─ main.py
├─ model/
│  └─ predict.py            # Contains MODEL_VERSION and inference logic
├─ schemas/
├─ configurations/
├─ config/
│  └─ logging_config.py     # Centralized logging (console + rotating file)
├─ logs/                    # Runtime logs (rotating)
├─ Dockerfile
├─ requirements.txt
└─ README.md
```

---

## ⚙️ Local Development
### 1) Create & activate environment (optional)
```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\\Scripts\\activate
```

### 2) Install dependencies
```bash
pip install -r requirements.txt
```

### 3) Run locally
```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --reload --access-log
```

Open: `http://localhost:8000/docs`

---

## 🐳 Run with Docker (Local)
### 1) Build image
```bash
docker build -t premium-classifier:latest .
```

### 2) Run container
```bash
docker run -d \
  --name premium-api \
  -p 8000:80 \
  premium-classifier:latest
```

Open: `http://localhost:8000/docs`

---

## ☁️ Deploy on AWS EC2 (Summary)
1. Launch EC2 (Free Tier) and connect via SSH
2. Install Docker and enable on boot
3. Pull image and run container exposing API port (e.g., 80)
4. **Security Group:** Add inbound rule to allow public access to the API port
5. (Optional) Configure Docker restart policy to auto‑start after reboot

> Endpoint is HTTP (no TLS) due to Free Tier/demo constraints. For HTTPS, consider **ALB + ACM** or **API Gateway**.

---

## 🔒 Production Readiness Features
- **Input validation:** Pydantic models with field constraints & descriptions
- **Error handling:** Structured responses and appropriate status codes (`200/4xx/5xx`)
- **Rate limiting:** Lightweight in‑app middleware returning **HTTP 429** on abuse
- **Request/Response logging:** Centralized logger (console + rotating files) capturing method, path, IP, status, latency, and exceptions
- **Model versioning:** `MODEL_VERSION` surfaced via `/health`
- **Auto‑start:** Docker service enabled to start on reboot
- **Cloud monitoring (optional):** CloudWatch Agent for memory/disk metrics; CPU/Network via EC2 basic monitoring

---

## 🧪 API Endpoints
### `GET /health`
Returns service health and model version.

**Response**
```json
{
  "status": "ok",
  "model_version": "<MODEL_VERSION>",
  "uptime_seconds": 1234
}
```

### `POST /predict`
Accepts validated input payload and returns premium category.

**Request (example)**
```json
{
  "age": 42,
  "income": 950000,
  "vehicle_age_years": 3,
  "previous_claims": 0,
  "marital_status": "single"
}
```

**Response (example)**
```json
{
  "premium_category": "High",
  "confidence": 0.87,
  "model_version": "<MODEL_VERSION>"
}
```

> Exact fields depend on your `schemas/` definitions; Swagger UI (`/docs`) lists authoritative schema with examples and descriptions.

---

## 📊 Observability & Logs
- Application logs are written to **console** and `logs/api.log` (rotating 5 MB x 3 files)
- Sample lines:
```
[REQ] 1.2.3.4 POST /predict
[RES] 1.2.3.4 POST /predict -> 200 in 34.11ms
[429] Rate limit exceeded for 1.2.3.4
```

### (Optional) CloudWatch Agent on Ubuntu EC2
Install and start agent to collect memory & disk usage metrics:
```bash
wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
sudo dpkg -i amazon-cloudwatch-agent.deb
sudo bash -lc 'cat > /opt/aws/amazon-cloudwatch-agent/bin/config.json <<EOF
{
  "metrics": {
    "append_dimensions": {"InstanceId": "${aws:InstanceId}"},
    "metrics_collected": {
      "mem": {"measurement": ["mem_used_percent"], "metrics_collection_interval": 60},
      "disk": {"measurement": ["used_percent"], "metrics_collection_interval": 60, "resources": ["/"]}
    }
  }
}
EOF'
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a start -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s
```
Then view metrics in **CloudWatch → Metrics → CWAgent**.

---

## 🔐 Security Notes
- Public HTTP endpoint is for demo/testing. For production, terminate TLS using **ALB + ACM** or **API Gateway + ACM**, and restrict inbound IP ranges.
- Avoid logging sensitive PII in request/response logs.

---

## 🚀 Future Improvements
- HTTPS via **ALB/API Gateway + ACM**
- CI/CD (GitHub Actions) for build/test/push/deploy
- Container orchestration (**ECS/EKS**) or serverless inference (**Lambda/SageMaker**)
- Advanced monitoring & tracing (structured JSON logs, log shipper to CloudWatch)
- Automated model versioning & promotion pipeline

---

## 🧠 Cloud Skills Demonstrated
- Provisioned and connected to **AWS EC2**
- Created **key pair** and secured SSH access
- Installed and configured **Docker** on EC2
- Pulled and ran **Docker image** for FastAPI service
- Enabled Docker to **auto‑start** on reboot
- Configured **Security Group inbound rules** for public API access
- Implemented **rate limiting**, **logging**, **error handling**, and **input validation**
- (Optional) Installed **CloudWatch Agent** for memory/disk metrics; created alarms for CPU/memory/disk (if enabled)



