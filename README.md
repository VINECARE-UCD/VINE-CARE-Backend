# 🌿 VineCare — Backend

**Precision Viticulture Platform for KOKOTOS ESTATE**

A production-grade Django microservices backend deployed on AWS, enabling vineyard managers with drone imagery analysis, phenology tracking, agronomic KPIs, and AI-driven yield predictions.

---

## 📋 Overview

VineCare Backend is a distributed system of three Django REST API microservices that power the core business logic for precision agriculture:

| **Service** | **Port** | **Purpose** |
|---|---|---|
| **Data Collection API** | 8000 | Receives drone imagery, flight metadata, and processes uploads to S3 |
| **Notification Service** | 8001 | Sends alerts and recommendations triggered by phenology events or KPI thresholds |
| **Phenology Analysis** | 8002 | Tracks vine growth stages throughout the growing season |

All services are containerized with Docker, deployed on AWS EC2 instances in private subnets within a VPC, and fronted by Application Load Balancers.

---

## 🏗️ Architecture

### High-Level Design
```
Internet → IGW → ALBs → Frontend (React/Next.js) → Backend Microservices → S3 + Aurora PostgreSQL
```

**Key Infrastructure:**
- **Region**: eu-central-1 (Frankfurt)
- **Compute**: 5 × t3.micro EC2 instances (2 frontend, 3 backend)
- **Load Balancing**: 3 × Application Load Balancers (Dualstack IPv4/IPv6)
- **Database**: Amazon Aurora PostgreSQL Serverless v2 (auto-scaling, multi-AZ)
- **Storage**: Amazon S3 (vine-care-bucket) for drone imagery
- **Network**: VPC with public/private subnet isolation, NAT Gateway for secure outbound access

**For detailed infrastructure documentation, see** [`AWS_Architecture.pdf`](./docs/VineCare_AWS_Architecture.pdf) **in the** `docs/` **folder.**

---

## 🚀 Quick Start

### Prerequisites
- Docker & Docker Compose (v2)
- Python 3.10+ (for local development)
- AWS credentials configured (for S3 access)
- PostgreSQL client (optional, for database debugging)

### 1. Clone the Repository
```bash
git clone https://github.com/VINECARE-UCD/VINE-CARE-Backend.git
cd VINE-CARE-Backend
```

### 2. Configure Environment Variables

Create a `.env` file in the project root with:

```env
# AWS Configuration
AWS_REGION=eu-central-1
AWS_ACCESS_KEY_ID=<your-key>
AWS_SECRET_ACCESS_KEY=<your-secret>
AWS_S3_BUCKET=vine-care-bucket

# Database Configuration
DB_HOST=<aurora-endpoint>
DB_PORT=5432
DB_NAME=vinecare
DB_USER=<db-user>
DB_PASSWORD=<db-password>

# Django Settings
DEBUG=False
SECRET_KEY=<generate-strong-key>
ALLOWED_HOSTS=<alb-dns-endpoint>

# Service URLs (for inter-service communication)
DATA_COLLECTION_URL=http://vine-care-instance-data-collection:8000
NOTIFICATION_URL=http://vine-care-instance-notification:8001
PHENOLOGY_URL=http://vine-care-instance-phenology:8002
```

### 3. Run Services Locally with Docker Compose

```bash
docker-compose up -d
```

This will start all three services on their respective ports:
- Data Collection: `http://localhost:8000`
- Notification: `http://localhost:8001`
- Phenology: `http://localhost:8002`

### 4. Run Database Migrations

```bash
docker-compose exec data-collection python manage.py migrate
docker-compose exec notification python manage.py migrate
docker-compose exec phenology python manage.py migrate
```

### 5. Check Health

```bash
curl http://localhost:8000/api/health/
curl http://localhost:8001/api/health/
curl http://localhost:8002/api/health/
```

---

## 📁 Project Structure

```
VINE-CARE-Backend/
├── data-collection/          # Django microservice (Port 8000)
│   ├── manage.py
│   ├── requirements.txt
│   ├── Dockerfile
│   └── vinecare/
│       ├── settings.py
│       ├── urls.py
│       └── wsgi.py
│
├── notification/             # Django microservice (Port 8001)
│   ├── manage.py
│   ├── requirements.txt
│   ├── Dockerfile
│   └── vinecare/
│
├── phenology/                # Django microservice (Port 8002)
│   ├── manage.py
│   ├── requirements.txt
│   ├── Dockerfile
│   └── vinecare/
│
├── docker-compose.yml        # Multi-service orchestration
├── .env.example              # Environment variables template
└── docs/
    └── VineCare_AWS_Architecture.pdf  # Full infrastructure documentation
```

---

## 🔌 API Endpoints

### Data Collection Service (Port 8000)
```
GET    /api/health/                    Health check
POST   /data/upload/                   Upload drone images
GET    /data/images/<block_id>/        Retrieve images for a block
POST   /data/process/                  Trigger image processing
```

### Notification Service (Port 8001)
```
GET    /api/health/                    Health check
POST   /notifications/send/            Send alert to farm manager
GET    /notifications/history/         Retrieve notification history
```

### Phenology Service (Port 8002)
```
GET    /api/health/                    Health check
POST   /phenology/track/               Log vine growth stage
GET    /phenology/stages/<block_id>/   Get phenology timeline for block
```

---

## 🐳 Docker & Deployment

### Build Individual Services
```bash
docker build -t vinecare/data-collection ./data-collection
docker build -t vinecare/notification ./notification
docker build -t vinecare/phenology ./phenology
```

### Deploy to AWS EC2
Each service runs in a Docker container on its own private EC2 instance, managed by Gunicorn:

```bash
# SSH into the instance
ssh -i your-key.pem ec2-user@<instance-ip>

# Pull latest code
cd /opt/vinecare
git pull origin main

# Restart service with docker-compose
docker-compose up -d --build
```

---

## 🔒 Security & Network Isolation

- **Private Subnets**: All backend services run in private subnets with no public IPs — unreachable directly from the internet.
- **ALB Routing**: Traffic from the internet is routed through Application Load Balancers using path-based rules (`/data/*`, `/notifications/*`, `/phenology/*`).
- **Database Isolation**: Aurora PostgreSQL runs in dedicated private subnets; only backend services can access it.
- **S3 Access**: All file uploads are written to S3 via IAM roles — no public S3 access.
- **NAT Gateway**: Backend instances can make outbound calls (package updates, external APIs) through a NAT Gateway without exposing inbound ports.

---

## 📊 Key Features

| Feature | Service | Status |
|---|---|---|
| **Drone Image Upload** | Data Collection | ✅ Active |
| **Phenology Tracking** | Phenology Service | ✅ Active |
| **KPI Dashboards** | Frontend + Backend | ✅ Active |
| **Yield Predictions** | ML Pipeline | 🔄 In Development |
| **Disease Risk Alerts** | Notification Service | ✅ Active |
| **Multi-Block Management** | Data Collection | ✅ Active |

---

## 🛠️ Development Workflow

### Local Testing
```bash
# Start services
docker-compose up -d

# Run tests
docker-compose exec data-collection python manage.py test
docker-compose exec notification python manage.py test
docker-compose exec phenology python manage.py test

# View logs
docker-compose logs -f data-collection
docker-compose logs -f notification
docker-compose logs -f phenology
```

### Deploying to Production
1. Commit and push to the `main` branch
2. SSH into the EC2 instance running the service
3. Pull latest changes: `git pull origin main`
4. Rebuild and restart: `docker-compose up -d --build`
5. Verify health check: `curl http://localhost:8000/api/health/`

---

## 📖 Documentation

**Full AWS Infrastructure Documentation**: [`VineCare_AWS_Architecture.pdf`](./docs/VineCare_AWS_Architecture.pdf)

Covers:
- Complete VPC design with subnet layout
- EC2 instance inventory and roles
- Load balancer configuration
- Database (Aurora Serverless v2) setup
- S3 bucket configuration
- Security groups and network isolation
- High availability and disaster recovery strategy

---

## 🌍 Environment Variables Reference

See [`.env.example`](./.env.example) for a complete template.

Key variables:
- `DB_HOST` — Aurora endpoint
- `AWS_S3_BUCKET` — S3 bucket name for image storage
- `DEBUG` — Django debug mode (False in production)
- `SECRET_KEY` — Django secret key (generate with `django-admin.py shell -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"`)

---

## 📞 Support & Contact

For questions or issues related to the VineCare platform:
- Check the [AWS Architecture documentation](./docs/VineCare_AWS_Architecture.pdf)
- Review Django microservices logs via `docker-compose logs`
- Verify network connectivity to ALBs and Aurora endpoint

---

## 📄 License

This project is developed for KOKOTOS ESTATE and is confidential. All rights reserved.

---

**Last Updated**: June 2026  
**Deployment Region**: eu-central-1 (Frankfurt)  
**Status**: Production
