# FusionSynth Local — Setup & Configuration Guide

> **Multimodal Fusion Synthetic Data Platform for Biomedical Research**
> Generate privacy-preserving, biologically coherent synthetic cohorts across 6 data modalities.

---

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Quick Start (5 Minutes)](#2-quick-start)
3. [Backend Setup (FastAPI)](#3-backend-setup)
4. [Frontend Setup (React)](#4-frontend-setup)
5. [Database Configuration](#5-database-configuration)
6. [Environment Variables](#6-environment-variables)
7. [Configuration File Reference](#7-configuration-file-reference)
8. [Research Workflow — End to End](#8-research-workflow)
9. [API Endpoint Reference](#9-api-endpoint-reference)
10. [Supported Data Modalities](#10-supported-data-modalities)
11. [GPU / Compute Configuration](#11-gpu-compute-configuration)
12. [Troubleshooting](#12-troubleshooting)
13. [Architecture Overview](#13-architecture-overview)

---

## 1. Prerequisites

| Requirement | Minimum | Recommended |
|---|---|---|
| **Python** | 3.11+ | 3.12 |
| **Node.js** | 18+ | 20 LTS |
| **PostgreSQL** | 14+ | 16 (via Docker) |
| **Docker** | 20+ | Latest |
| **RAM** | 8 GB | 16+ GB |
| **Storage** | 10 GB free | 50+ GB (for datasets) |
| **GPU** *(optional)* | — | NVIDIA CUDA 12+ |
| **OS** | macOS / Linux / WSL2 | macOS (Apple Silicon) or Linux |

### Install System Dependencies

```bash
# macOS (Homebrew)
brew install python@3.12 node docker

# Ubuntu / Debian
sudo apt update && sudo apt install -y python3.12 python3.12-venv nodejs npm docker.io docker-compose
```

---

## 2. Quick Start (5 Minutes)

```bash
# 1. Clone and enter the repository
cd multimodal_synth_repo

# 2. Start PostgreSQL via Docker
docker compose up -d

# 3. Set up Python environment
python3 -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"

# 4. Configure environment
cp .env.example .env

# 5. Run database migrations
alembic upgrade head

# 6. Start the backend API
uvicorn src.api.main:app --host 0.0.0.0 --port 8000 --reload

# 7. (New terminal) Start the frontend
cd frontend
npm install
npm run dev
```

**Access Points:**
| Service | URL |
|---|---|
| Frontend (React) | http://localhost:5173 |
| Backend API | http://localhost:8000 |
| API Docs (Swagger) | http://localhost:8000/docs |
| Health Check | http://localhost:8000/health |

---

## 3. Backend Setup (FastAPI)

### 3.1 Python Virtual Environment

```bash
# Create and activate virtual environment
python3 -m venv .venv
source .venv/bin/activate    # macOS/Linux
# .venv\Scripts\activate     # Windows (PowerShell)

# Install all dependencies (including dev tools)
pip install -e ".[dev]"
```

### 3.2 Core Dependencies

| Category | Packages | Purpose |
|---|---|---|
| **Web Framework** | FastAPI, Uvicorn, Pydantic | REST API, async server, validation |
| **Database** | SQLAlchemy (async), asyncpg, Alembic | ORM, PostgreSQL driver, migrations |
| **Data Formats** | PyArrow, Pandas, Zarr, NumPy | Parquet/CSV/Zarr read/write |
| **ML / Deep Learning** | PyTorch, TorchVision, MONAI | Image encoders, fusion models |
| **Tabular Synthesis** | SDV (CTGAN, TVAE) | Synthetic tabular data generation |
| **Omics** | AnnData, Scanpy | Single-cell / spatial data handling |
| **Evaluation** | Scikit-learn, Matplotlib, Plotly | Metrics, visualization |

### 3.3 Start the API Server

```bash
# Development (auto-reload)
uvicorn src.api.main:app --host 0.0.0.0 --port 8000 --reload

# Production
uvicorn src.api.main:app --host 0.0.0.0 --port 8000 --workers 4
```

### 3.4 Run Database Migrations

```bash
# Apply all migrations
alembic upgrade head

# Create a new migration after model changes
alembic revision --autogenerate -m "description_of_change"
```

---

## 4. Frontend Setup (React)

```bash
cd frontend
npm install
npm run dev          # Development → http://localhost:5173
npm run build        # Production build → frontend/dist/
```

### 4.1 Frontend Stack

| Technology | Purpose |
|---|---|
| **Vite** | Build tool, dev server with HMR |
| **React 19** | UI library |
| **TypeScript** | Type-safe development |
| **Framer Motion** | Scroll-triggered animations |
| **React Router** | Client-side routing (8 pages) |
| **Lucide React** | Icon system |
| **Recharts** | Data visualization |

### 4.2 Frontend Pages

| Route | Page | Function |
|---|---|---|
| `/` | Home | Landing page — platform overview, modalities, research impact |
| `/dashboard` | Dashboard | Project stats, quick navigation |
| `/projects` | Projects | Create / manage research projects |
| `/data` | Data Ingestion | Register 6 modality types (tabular, images, embeddings, omics, spatial, perturbation) |
| `/train` | Training | Configure fusion encoders (ResNet-18, ViT), synthesizers (CTGAN, TVAE) |
| `/generate` | Generation | Generate synthetic cohorts with entity type, fidelity, conditions |
| `/evaluate` | Evaluation | Run realism, utility, privacy, coherence evaluations |
| `/export` | Export | Package multi-fidelity bundles (Parquet, Zarr, CSV) |

---

## 5. Database Configuration

### 5.1 Using Docker (Recommended)

```bash
# Start PostgreSQL 16
docker compose up -d

# Verify it's running
docker ps
docker logs fusionsynth_db
```

The `docker-compose.yml` creates a PostgreSQL 16 Alpine container with:
- **Username**: `fusionsynth`
- **Password**: `fusionsynth`
- **Database**: `fusionsynth`
- **Port**: `5432`
- **Persistent volume**: `fusionsynth_pgdata`

### 5.2 Using an Existing PostgreSQL Server

If you have an existing PostgreSQL installation:

```bash
# Create the database and user
psql -U postgres -c "CREATE USER fusionsynth WITH PASSWORD 'fusionsynth';"
psql -U postgres -c "CREATE DATABASE fusionsynth OWNER fusionsynth;"
psql -U postgres -c "GRANT ALL PRIVILEGES ON DATABASE fusionsynth TO fusionsynth;"
```

Then update your `.env` file (see Section 6).

### 5.3 Database Schema

The schema includes these core tables (managed by Alembic):

| Table | Purpose |
|---|---|
| `projects` | Research project metadata (name, root path, timestamps) |
| `manifests` | Data registration records per modality |
| `jobs` | Async job tracking (training, generation, evaluation, export) |
| `synthetic_samples` | Generated synthetic sample metadata (three-layer schema) |

---

## 6. Environment Variables

Copy `.env.example` to `.env` and customize:

```bash
cp .env.example .env
```

```env
# ── Database ──
FUSIONSYNTH_DB_HOST=localhost
FUSIONSYNTH_DB_PORT=5432
FUSIONSYNTH_DB_USER=fusionsynth
FUSIONSYNTH_DB_PASSWORD=fusionsynth
FUSIONSYNTH_DB_NAME=fusionsynth

# ── Compute ──
FUSIONSYNTH_DEVICE=auto          # "auto" | "cpu" | "cuda"

# ── Server ──
FUSIONSYNTH_API_PORT=8000
```

| Variable | Default | Description |
|---|---|---|
| `FUSIONSYNTH_DB_HOST` | `localhost` | PostgreSQL host |
| `FUSIONSYNTH_DB_PORT` | `5432` | PostgreSQL port |
| `FUSIONSYNTH_DB_USER` | `fusionsynth` | PostgreSQL username |
| `FUSIONSYNTH_DB_PASSWORD` | `fusionsynth` | PostgreSQL password |
| `FUSIONSYNTH_DB_NAME` | `fusionsynth` | PostgreSQL database name |
| `FUSIONSYNTH_DEVICE` | `auto` | Compute device: `auto` detects GPU, `cpu` forces CPU, `cuda` for NVIDIA |
| `FUSIONSYNTH_API_PORT` | `8000` | API server port |

---

## 7. Configuration File Reference

### `configs/default.yaml`

```yaml
# ── Paths ──
project_root: "."
data_dir: "data"              # Where raw datasets are stored
models_dir: "models"          # Where trained model checkpoints are saved
outputs_dir: "outputs"        # Where generated synthetic data is written

# ── Server ──
api_host: "0.0.0.0"
api_port: 8000

# ── Compute ──
device: "auto"                # "auto", "cpu", or "cuda"

# ── Training Defaults ──
default_latent_dim: 256       # Shared latent space dimensionality
default_epochs: 50            # Training epochs for fusion models
default_batch_size: 64        # Mini-batch size
default_learning_rate: 0.001  # Adam optimizer LR

# ── Image Encoder ──
image_encoder:
  arch: "resnet18"            # Options: resnet18, vit_lite, small_cnn
  pretrained: true            # Use ImageNet pretrained weights
  embedding_dim: 512          # Output embedding dimensionality

# ── Tabular Synthesizer ──
tabular_synth:
  model: "ctgan"              # Options: ctgan, tvae
  epochs: 300                 # CTGAN/TVAE training epochs
  batch_size: 500             # Tabular batch size

# ── Fusion ──
fusion:
  method: "contrastive_projection"  # Options: contrastive_projection, concat_projection
  latent_dim: 256                   # Shared latent dimension

# ── Generation ──
generation:
  default_num_samples: 1000
  split_strategy: "train_val_test"  # Options: train_val_test, train_test, none
  split_ratios:
    train: 0.7
    val: 0.15
    test: 0.15

# ── Evaluation ──
evaluation:
  realism_metrics:
    - wasserstein              # Wasserstein distance between distributions
    - ks_statistic             # Kolmogorov-Smirnov test
    - mmd                      # Maximum Mean Discrepancy
  privacy_tests:
    - nearest_neighbor_ratio   # NNDR — detect memorization
    - membership_inference     # MIA — membership inference attack
  utility_benchmark: "response_prediction"  # Downstream task for utility
```

---

## 8. Research Workflow — End to End

### Step 1: Create a Research Project

```bash
curl -X POST http://localhost:8000/projects/ \
  -H "Content-Type: application/json" \
  -d '{"name": "lung_cancer_cohort", "root_path": "./data/lung_cancer"}'
```

Or use the **Projects** page in the frontend (http://localhost:5173/projects).

### Step 2: Register Your Multimodal Data

Register each data modality against your project:

#### Tabular (Clinical Data)
```bash
curl -X POST http://localhost:8000/projects/lung_cancer_cohort/data/register-tabular \
  -H "Content-Type: application/json" \
  -d '{
    "path": "./data/lung_cancer/clinical.csv",
    "id_column": "patient_id",
    "label_columns": ["cancer_type", "stage", "treatment_response"]
  }'
```

#### Pathology Images
```bash
curl -X POST http://localhost:8000/projects/lung_cancer_cohort/data/register-images \
  -H "Content-Type: application/json" \
  -d '{
    "path": "./data/lung_cancer/tiles/",
    "patient_id_map_path": "./data/lung_cancer/tile_patient_map.csv"
  }'
```

#### Omics (Gene Expression)
```bash
curl -X POST http://localhost:8000/projects/lung_cancer_cohort/data/register-omics \
  -H "Content-Type: application/json" \
  -d '{
    "path": "./data/lung_cancer/rnaseq.h5ad",
    "patient_id_column": "patient_id"
  }'
```

#### Spatial Transcriptomics
```bash
curl -X POST http://localhost:8000/projects/lung_cancer_cohort/data/register-spatial \
  -H "Content-Type: application/json" \
  -d '{
    "path": "./data/lung_cancer/visium_spots.h5ad",
    "patient_id_column": "patient_id",
    "coordinates_key": "spatial"
  }'
```

#### Perturbation Bundles (CRISPR / Drug)
```bash
curl -X POST http://localhost:8000/projects/lung_cancer_cohort/data/register-perturbation \
  -H "Content-Type: application/json" \
  -d '{
    "baseline_path": "./data/lung_cancer/baseline.h5ad",
    "post_perturbation_path": "./data/lung_cancer/post_treatment.h5ad",
    "conditions_path": "./data/lung_cancer/conditions.csv",
    "patient_id_column": "patient_id"
  }'
```

#### Precomputed Embeddings
```bash
curl -X POST http://localhost:8000/projects/lung_cancer_cohort/data/register-embeddings \
  -H "Content-Type: application/json" \
  -d '{
    "path": "./data/lung_cancer/embeddings.npy",
    "patient_id_map_path": "./data/lung_cancer/embedding_patient_map.csv"
  }'
```

### Step 3: Train Fusion Models

```bash
curl -X POST http://localhost:8000/projects/lung_cancer_cohort/train \
  -H "Content-Type: application/json" \
  -d '{
    "encoder": "resnet18",
    "model": "ctgan",
    "fusion_method": "contrastive_projection",
    "latent_dim": 256,
    "epochs": 50
  }'
```

### Step 4: Generate Synthetic Data

```bash
curl -X POST http://localhost:8000/projects/lung_cancer_cohort/generate \
  -H "Content-Type: application/json" \
  -d '{
    "num_samples": 5000,
    "entity_type": "patient_case",
    "split_strategy": "train_val_test",
    "fidelity_level": "full_multimodal",
    "conditions": {"cancer_type": "NSCLC", "stage": "III"},
    "modalities": ["tabular", "image_embedding", "omics"]
  }'
```

### Step 5: Evaluate Quality

```bash
curl -X POST http://localhost:8000/projects/lung_cancer_cohort/evaluate \
  -H "Content-Type: application/json" \
  -d '{
    "real_ref": "./data/lung_cancer/clinical.csv",
    "synth_ref": "./outputs/lung_cancer_cohort/synthetic_latest.parquet",
    "benchmark_task": "response_prediction"
  }'
```

### Step 6: Export Research Bundle

```bash
curl -X POST http://localhost:8000/projects/lung_cancer_cohort/export \
  -H "Content-Type: application/json" \
  -d '{
    "fidelity_level": "full_multimodal",
    "formats": ["parquet", "zarr"],
    "include_report": true,
    "include_data_card": true
  }'
```

---

## 9. API Endpoint Reference

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/health` | Health check |
| `POST` | `/projects/` | Create a project |
| `GET` | `/projects/` | List all projects |
| `DELETE` | `/projects/{id}` | Delete a project |
| `POST` | `/projects/{id}/data/register-tabular` | Register tabular data |
| `POST` | `/projects/{id}/data/register-images` | Register pathology images |
| `POST` | `/projects/{id}/data/register-embeddings` | Register precomputed embeddings |
| `POST` | `/projects/{id}/data/register-omics` | Register omics data (h5ad) |
| `POST` | `/projects/{id}/data/register-spatial` | Register spatial transcriptomics |
| `POST` | `/projects/{id}/data/register-perturbation` | Register perturbation bundles |
| `POST` | `/projects/{id}/validate` | Validate dataset integrity |
| `POST` | `/projects/{id}/train` | Start training job |
| `POST` | `/projects/{id}/generate` | Start generation job |
| `POST` | `/projects/{id}/evaluate` | Start evaluation job |
| `POST` | `/projects/{id}/export` | Start export job |
| `GET` | `/jobs/{job_id}` | Check job status |

📄 **Interactive API docs**: http://localhost:8000/docs (Swagger UI)

---

## 10. Supported Data Modalities

| Modality | Data Family | Formats | Example |
|---|---|---|---|
| **Clinical Tabular** | A | CSV, Parquet | Demographics, staging, biomarkers, outcomes |
| **Pathology Images** | B | PNG, JPEG, TIFF (tiles) | H&E-stained whole-slide image tiles |
| **Precomputed Embeddings** | F | .npy, .pt, Zarr | ResNet-18, ViT, foundation model encodings |
| **Omics (RNA/Protein)** | C | AnnData (.h5ad) | Single-cell gene expression, protein panels |
| **Spatial Transcriptomics** | C | AnnData (.h5ad) + coordinates | Visium spots, MERFISH cells with x/y |
| **Perturbation Bundles** | D | AnnData (.h5ad) + conditions CSV | CRISPR knockouts, drug treatment pairs |

### Data Directory Structure (Recommended)

```
data/
└── lung_cancer/
    ├── clinical.csv                  # Tabular — patient records
    ├── tiles/                        # Images — H&E patches
    │   ├── patient_001_tile_0.png
    │   └── ...
    ├── tile_patient_map.csv          # Image-to-patient mapping
    ├── embeddings.npy                # Precomputed embeddings (N × 512)
    ├── embedding_patient_map.csv     # Embedding-to-patient mapping
    ├── rnaseq.h5ad                   # Omics — AnnData format
    ├── visium_spots.h5ad             # Spatial — with obsm['spatial']
    ├── baseline.h5ad                 # Perturbation — pre-treatment
    ├── post_treatment.h5ad           # Perturbation — post-treatment
    └── conditions.csv                # Perturbation — conditions
```

---

## 11. GPU / Compute Configuration

### Auto-Detection (Default)

```env
FUSIONSYNTH_DEVICE=auto
```

The platform automatically detects your hardware:
- **NVIDIA GPU** → Uses CUDA for PyTorch operations
- **Apple Silicon** → Falls back to CPU (MPS support planned)
- **No GPU** → Uses CPU

### Force CPU Mode

```env
FUSIONSYNTH_DEVICE=cpu
```

Useful when GPU memory is limited or when running on a shared server.

### NVIDIA CUDA Setup

```bash
# Verify CUDA availability
python -c "import torch; print(torch.cuda.is_available())"
python -c "import torch; print(torch.cuda.get_device_name(0))"

# Check GPU memory
nvidia-smi
```

### Training Performance Guide

| Hardware | Clinical Only (1K rows) | Multi-Modal (1K rows) | Large Cohort (50K rows) |
|---|---|---|---|
| CPU (8-core) | ~2 min | ~15 min | ~4 hours |
| RTX 3060 (12 GB) | ~30 sec | ~3 min | ~45 min |
| RTX 4090 (24 GB) | ~15 sec | ~1 min | ~15 min |
| A100 (80 GB) | ~10 sec | ~30 sec | ~8 min |

---

## 12. Troubleshooting

### Database Connection Failed

```
sqlalchemy.exc.OperationalError: could not connect to server
```

**Fix**: Ensure PostgreSQL is running:
```bash
docker compose up -d
docker logs fusionsynth_db    # Check for errors
```

### Port Already in Use

```
uvicorn.error: [Errno 48] Address already in use
```

**Fix**: Kill the existing process or use a different port:
```bash
lsof -i :8000 | grep LISTEN    # Find the process
kill -9 <PID>                   # Kill it

# Or change port
uvicorn src.api.main:app --port 8001
```

### PyTorch CUDA Out of Memory

```
torch.cuda.OutOfMemoryError
```

**Fix**: Reduce batch size in `configs/default.yaml`:
```yaml
default_batch_size: 32        # Reduce from 64
tabular_synth:
  batch_size: 250             # Reduce from 500
```

### Frontend Can't Reach Backend (CORS)

The backend has permissive CORS (`allow_origins=["*"]`) for local dev. If the frontend shows network errors:
1. Verify backend is running: `curl http://localhost:8000/health`
2. Check that `frontend/src/api.ts` points to `http://localhost:8000`

### Alembic Migration Conflict

```bash
# Reset and reapply all migrations
alembic downgrade base
alembic upgrade head
```

> **⚠️ Warning**: This drops all data. Only use in development.

---

## 13. Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                    Frontend (React)                  │
│            http://localhost:5173                     │
│  Home │ Dashboard │ Projects │ Data │ Train │ ...   │
└────────────────────┬────────────────────────────────┘
                     │ HTTP/JSON
┌────────────────────▼────────────────────────────────┐
│                FastAPI Backend                       │
│            http://localhost:8000                     │
│                                                     │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐            │
│  │ Projects │ │   Data   │ │ Training │            │
│  │  Router  │ │  Router  │ │  Router  │            │
│  └──────────┘ └──────────┘ └──────────┘            │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐            │
│  │Generate  │ │ Evaluate │ │  Export  │            │
│  │  Router  │ │  Router  │ │  Router  │            │
│  └──────────┘ └──────────┘ └──────────┘            │
│                                                     │
│  ┌─────────────────────────────────────────┐        │
│  │           Core Services                 │        │
│  │  Ingest │ Fusion │ Synth │ Validate     │        │
│  └─────────────────────────────────────────┘        │
└────────────────────┬────────────────────────────────┘
                     │ asyncpg
┌────────────────────▼────────────────────────────────┐
│              PostgreSQL 16                          │
│         localhost:5432 (Docker)                     │
│                                                     │
│  projects │ manifests │ jobs │ synthetic_samples     │
└─────────────────────────────────────────────────────┘

Data Flow: data/ ──▶ models/ ──▶ outputs/
```

### Three-Layer Canonical Schema

Every synthetic sample produced by FusionSynth follows this standardized structure:

| Layer 1 — Identity | Layer 2 — Modalities | Layer 3 — Supervision |
|---|---|---|
| `synthetic_id` | `modality_name` | `task_labels` |
| `entity_type` | `storage_uri` | `conditioning_vars` |
| `cohort_id` | `shape` | `perturbation_vars` |
| `generator_version` | `dtype` | `consistency_score` |
| `split` | `paired_group_id` | `privacy_score` |
| `is_synthetic` | `quality_score` | `provenance` |

---

## Running Tests

```bash
# All tests
pytest tests/ -v

# With coverage
pytest tests/ -v --cov=src --cov-report=html

# Specific test file
pytest tests/test_api.py -v
```

---

## Production Checklist

- [ ] Change default database passwords in `.env`
- [ ] Set `FUSIONSYNTH_DEVICE=cuda` if GPU available
- [ ] Use `uvicorn --workers 4` for multi-process serving
- [ ] Run `npm run build` and serve frontend via Nginx or similar
- [ ] Set up PostgreSQL backups for `fusionsynth_pgdata` volume
- [ ] Review and tune `configs/default.yaml` for your dataset sizes
- [ ] Run `alembic upgrade head` before first API start

---

*Last updated: March 2026 · FusionSynth Local v0.1.0*
