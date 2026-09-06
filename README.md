<p align="center">
  <img src="https://img.shields.io/badge/Python-3.13+-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/React-19-61DAFB?style=for-the-badge&logo=react&logoColor=black" alt="React">
  <img src="https://img.shields.io/badge/Laravel-13-FF2D20?style=for-the-badge&logo=laravel&logoColor=white" alt="Laravel">
  <img src="https://img.shields.io/badge/FastAPI-0.115+-009688?style=for-the-badge&logo=fastapi&logoColor=white" alt="FastAPI">
  <img src="https://img.shields.io/badge/Licence-Priv%C3%A9e-red?style=for-the-badge" alt="Licence">
</p>

<h1 align="center">GMAO</h1>

<p align="center">
  <strong>Gestion de Maintenance Assistée par Ordinateur</strong><br>
  Plateforme complète de maintenance prédictive propulsée par l'Intelligence Artificielle
</p>

<p align="center">
  <a href="#-architecture">Architecture</a> ·
  <a href="#-services-ia">Services IA</a> ·
  <a href="#-frontend">Frontend</a> ·
  <a href="#-api-backend">Backend</a> ·
  <a href="#-d%C3%A9ploiement">Déploiement</a> ·
  <a href="#-api-r%C3%A9f%C3%A9rence">API</a>
</p>

---

## Vue d'ensemble

GMAO est une application enterprise de **maintenance prédictive et assistée par ordinateur** combinant :

- **5 microservices IA** — prédiction de pannes, assistant RAG, analytics, OCR, et orchestration capteurs
- **Un frontend React moderne** — interface responsive avec scan QR, graphiques temps réel, et chat IA
- **Un backend Laravel** — API REST avec authentification JWT et contrôle d'accès basé sur les rôles
- **Une base de données MySQL** — 15 tables couvrant équipements, interventions, pannes, et documents

---

## Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                     GMAO-FRONTEND  (React 19)                       │
│                     Vite 8 · Tailwind CSS v4 · Port 5173            │
├──────────────────────────────────────────────────────────────────────┤
│                     LARAVEL BACKEND  (PHP 8.3)                      │
│                     JWT/Sanctum · RBAC 4 rôles · Port 8000          │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌──────────────┐  │
│  │  GMAO-ML    │ │  GMAO-RAG   │ │ GMAO-ANALYTICS│ │  GMAO-OCR   │  │
│  │  :8100      │ │  :8500      │ │  :8300        │ │  :8400       │  │
│  │  Prédictif  │ │  Assistant  │ │  KPI/Rapports │ │  Vision QR   │  │
│  └──────┬──────┘ └──────┬──────┘ └──────┬────────┘ └──────┬───────┘  │
│         │               │               │                  │          │
│  ┌──────┴───────────────┴───────────────┴──────────────────┴───────┐ │
│  │                     GMAO-API  :8200                              │ │
│  │              Passerelle capteurs · Orchestrateur ML              │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                      │
├──────────────────────────────────────────────────────────────────────┤
│  MySQL (:3306)        │  Qdrant (:6333)        │  OpenAI / Gemini   │
│  15 tables gmao       │  Vecteurs embeddings   │  Génération LLM    │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Services IA

Le dossier [`GMAO-SERVICES-IA`](./GMAO-SERVICES-IA) contient **5 microservices Python** gérés en workspace `uv`, tous construits avec **FastAPI**.

### GMAO-ML — Maintenance Prédictive

Service de prédiction de pannes basé sur le machine learning.

| Composant | Détail |
|-----------|--------|
| **Modèle** | HistGradientBoosting (scikit-learn) |
| **Dataset** | AI4I 2020 — 10 000 observations capteurs |
| **Performance** | F1-macro : 0.955 · AUC : 0.974 · Rappel pannes : 85.3% |
| **Pipeline** | Prétraitement embarqué dans le Pipeline sklearn (pas de skew train/inference) |
| **Suivi** | MLflow pour le tracking d'expériences |
| **Tests** | 52 unitaires |

**Endpoints** : `POST /predict` · `POST /predict/batch` · `GET /model/info` · `GET /healthz`

---

### GMAO-RAG — Assistant Technique IA

Assistant conversationnel RAG (Retrieval-Augmented Generation) pour les techniciens de maintenance.

| Composant | Détail |
|-----------|--------|
| **Pipeline** | 9 couches : DataSource → Parser → Chunker → Embedding → Storage → Retrieval → Reranker → LLM → API |
| **Base vectorielle** | Qdrant — collection `gmao_chunks` |
| **Embeddings** | `intfloat/multilingual-e5-small` (384 dimensions, sentence-transformers) |
| **Reranking** | Cross-encoder `ms-marco-MiniLM` |
| **LLM** | OpenAI GPT-4o-mini · Gemini 2.5/3.6 Flash |
| **Ingestion** | PDF, DOCX, TXT, HTML, CSV, JSON, XLSX, Markdown + tables MySQL |
| **Tests** | 332 unitaires |

**Endpoints** : `POST /rag/search` · `POST /ingest/file` · `GET /documents/` · `DELETE /documents/{id}` · 12 endpoints au total

---

### GMAO-API — Passerelle Capteurs

Orchestrateur entre les capteurs IoT, le modèle ML, et le backend Laravel.

| Composant | Détail |
|-----------|--------|
| **Rôle** | Reçoit les données capteurs → envoie au ML → crée les interventions dans Laravel si prédiction = panne |
| **Simulation** | Génération de lectures artificielles avec taux de panne configurable |
| **Dashboard** | Page HTML temps réel pour le monitoring |
| **Tolérance** | Fonctionne hors-ligne avec catalogue de 12 équipements de test |

**Endpoints** : `POST /predictions` · `POST /simulate` · `GET /alerts` · `GET /laravel/interventions`

---

### GMAO-ANALYTICS — KPI & Rapports

Moteur d'analytics de maintenance calculant les indicateurs clés.

| Composant | Détail |
|-----------|--------|
| **Métriques** | MTBF (Mean Time Between Failures) · MTTR (Mean Time To Repair) · Disponibilité |
| **Agrégation** | Par équipement + vue flotte globale |
| **Enrichissement ML** | Croisement MTBF ↔ risque prédictif (dégradation gracieuse si ML indisponible) |
| **Export** | JSON · CSV · Markdown |
| **Dashboard** | KPI cards, barres de disponibilité, tableau détaillé |

**Endpoints** : `GET /metrics` · `GET /metrics/equipement/{id}` · `GET /report`

---

### GMAO-OCR — Vision QR Code

Service de lecture de codes QR sur les équipements par photo.

| Composant | Détail |
|-----------|--------|
| **Décodeurs** | pyzbar (principal) + OpenCV (fallback) |
| **Prétraitement** | Rotations multiples (0/90/180/270) + mise à l'échelle |
| **Sécurité** | Validation anti-phishing sur les URLs QR |
| **Formats** | SVG (CairoSVG), PNG, JPEG, EPS |
| **Enrichissement** | Requête API Laravel pour les détails équipement |

**Endpoints** : `POST /qr/scan` · `GET /healthz`

---

## Frontend

Le dossier [`GMAO-FRONTEND`](./GMAO-FRONTEND) contient l'application web React.

### Stack technique

| Technologie | Version | Rôle |
|-------------|---------|------|
| **React** | 19 | Framework UI |
| **TypeScript** | 6 | Typage statique |
| **Vite** | 8 | Build tool & dev server |
| **Tailwind CSS** | 4 | Design system custom (`@theme` tokens) |
| **React Router** | 7 | Routing SPA |
| **TanStack Query** | 5 | Gestion du state serveur |
| **Axios** | — | Client HTTP avec intercepteurs JWT |
| **Recharts** | 3 | Graphiques analytics |
| **html5-qrcode** | — | Scan QR par caméra |
| **oxlint** | 1.79 | Linting (pas ESLint) |

### Modules

| Route | Module | Rôles |
|-------|--------|-------|
| `/` | Dashboard — KPIs, graphiques, état des équipements | Responsable |
| `/equipements` | CRUD équipements + génération QR | Admin, Responsable, Technicien |
| `/demandes` | Demandes d'intervention + scan QR | Responsable, Demandeur |
| `/ordres` | Ordres de travail | Responsable, Technicien |
| `/pannes` | Historique des pannes | Responsable |
| `/affectations` | Affectation techniciens | Responsable, Technicien |
| `/analyse` | Analytics MTBF/MTTR/Disponibilité | Responsable |
| `/prediction` | Maintenance prédictive temps réel | Responsable |
| `/assistant` | Chat IA RAG (assistant technique) | Responsable, Technicien, Demandeur |
| `/utilisateurs` | Gestion des utilisateurs | Admin |
| `/documents` | Base documentaire RAG | Admin, Responsable |
| `/securite` | Journaux d'audit | Admin |

### Design System

Composants UI custom construits avec Tailwind CSS v4 :

- **`Button`** — variants primary/secondary/outline/ghost/danger avec état loading
- **`Card`** — conteneur avec titre et slot d'actions
- **`Badge`** — états colorés (success/danger/warning/info)
- **`Input`** — champs avec labels, erreurs, texte d'aide
- **`Modal`** — dialogues portal-based avec backdrop
- **`Feedback`** — Spinner, PageLoader, EmptyState

**Palette** : Bleu foncé `#102a43` (primary) · Accent teal `#00798c` · Font Instrument Sans

---

## Backend

Le backend **Laravel 13** (PHP 8.3) n'est pas inclus dans ce dépôt mais est requis. Il fournit :

| Fonctionnalité | Détail |
|----------------|--------|
| **Authentification** | JWT via Sanctum |
| **RBAC** | 4 rôles — Admin, Responsable, Technicien, Demandeur |
| **API REST** | CRUD complet pour équipements, interventions, ordres de travail, pannes, affectations, utilisateurs |
| **Base de données** | MySQL `gmao` — 15 tables (voir `db/schema_gmao.sql`) |

### Rôles et permissions

| Rôle | Permissions |
|------|-------------|
| **Admin** | Tout — CRUD utilisateurs, audit, sécurité |
| **Responsable** | Dashboard, analytics, prédiction, ordres de travail, pannes |
| **Technicien** | Lecture équipements, ses affectations, chat IA |
| **Demandeur** | Créer des demandes d'intervention, chat IA |

---

## Démarrage rapide

### Prérequis

- **Python** ≥ 3.13
- **Node.js** ≥ 18
- **PHP** ≥ 8.3 + Composer
- **MySQL** ≥ 8.0
- **Qdrant** (base vectorielle)
- **uv** (gestionnaire de paquets Python)

### 1. Base de données

```bash
# Importer le schéma
mysql -u root -p < GMAO-SERVICES-IA/db/schema_gmao.sql
```

### 2. Services IA

```bash
cd GMAO-SERVICES-IA

# Installer les dépendances
uv sync

# Démarrer tous les services
./start_services.sh start

# Vérifier l'état
./start_services.sh status
```

| Service | Port | URL |
|---------|------|-----|
| GMAO-ML | 8100 | http://localhost:8100/api/v1/healthz |
| GMAO-API | 8200 | http://localhost:8200/api/v1/healthz |
| GMAO-ANALYTICS | 8300 | http://localhost:8300/api/v1/healthz |
| GMAO-OCR | 8400 | http://localhost:8400/api/v1/healthz |
| GMAO-RAG | 8500 | http://localhost:8500/api/v1/health |

### 3. Backend Laravel

```bash
# Installer les dépendances
composer install

# Configurer .env
cp .env.example .env
php artisan key:generate

# Migrer la base
php artisan migrate

# Lancer le serveur
php artisan serve --port=8000
```

### 4. Frontend

```bash
cd GMAO-FRONTEND/frontend

# Installer les dépendances
npm install

# Lancer le dev server
npm run dev
```

L'app sera disponible sur http://localhost:5173

> **Compte de démonstration** : `admin@gmao.com` / `password`

---

## Schéma de la base de données

Le fichier [`db/schema_gmao.sql`](./GMAO-SERVICES-IA/db/schema_gmao.sql) contient le schéma complet MySQL (15 tables) :

```
roles ─┬─ utilisateurs ──┬─ demande_interventions ──┬─ ordre_travails ──┬─ pannes
       │                 │                          │                   │
       └─ specialites    │                          └─ affectations     │
                         │                                             │
                         └─ documents ──┬─ document_chunks             │
                                        └─ conversation_rags ────────┘
                                                └─ message_chats
                        
audit_logs · sessions · cache · cache_locks (Laravel)
```

---

## API Référence

### GMAO-SERVICES-IA

| Service | Documentation |
|---------|--------------|
| GMAO-RAG | [`doc_api/GMAO-RAG_ENDPOINTS.md`](./GMAO-SERVICES-IA/doc_api/GMAO-RAG_ENDPOINTS.md) — 12 endpoints |
| GMAO-ML | Via `/model/info` + OpenAPI auto-généré |
| GMAO-API | [`doc_api/GMAO-API_ENDPOINTS.md`](./GMAO-SERVICES-IA/doc_api/GMAO-API_ENDPOINTS.md) — 5 endpoints |
| GMAO-ANALYTICS | [`doc_api/GMAO-ANALYTICS_ENDPOINTS.md`](./GMAO-SERVICES-IA/doc_api/GMAO-ANALYTICS_ENDPOINTS.md) — 5 endpoints |
| GMAO-OCR | [`doc_api/GMAO-OCR_ENDPOINTS.md`](./GMAO-SERVICES-IA/doc_api/GMAO-OCR_ENDPOINTS.md) — 4 endpoints |

### Documentation interne

| Document | Contenu |
|----------|---------|
| [`documnetations/PIP_LINE_GLOBALE_RAG.md`](./GMAO-SERVICES-IA/documnetations/PIP_LINE_GLOBALE_RAG.md) | Pipeline RAG global |
| [`documnetations/DOC_GESTON_SQL.md`](./GMAO-SERVICES-IA/documnetations/DOC_GESTON_SQL.md) | Commandes MySQL |
| [`documnetations/DOC_REDMY_uv.md`](./GMAO-SERVICES-IA/documnetations/DOC_REDMY_uv.md) | Guide uv |

---

## Structure du projet

```
GMAO/
├── README.md                           ← Ce fichier
├── .gitmodules                         ← Définition des submodules
├── GMAO-FRONTEND/                      ← [Submodule] Application React
│   └── frontend/
│       ├── src/
│       │   ├── api/                    ← Client Axios + types TypeScript
│       │   ├── components/             ← UI custom + layout
│       │   ├── context/                ← AuthContext
│       │   ├── pages/                  ← 14 pages
│       │   └── lib/                    ← Utilitaires
│       ├── package.json
│       └── vite.config.ts
├── GMAO-SERVICES-IA/                   ← [Submodule] Microservices IA
│   ├── GMAO-RAG/                       ← Assistant RAG (port 8500)
│   ├── GMAO-ML/                        ← ML prédictif (port 8100)
│   ├── GMAO-API/                       ← Passerelle capteurs (port 8200)
│   ├── GMAO-ANALYTICS/                 ← KPI & rapports (port 8300)
│   ├── GMAO-OCR/                       ← Vision QR (port 8400)
│   ├── db/                             ← Schéma MySQL
│   ├── doc_api/                        ← Documentation endpoints
│   ├── start_services.sh               ← Script de démarrage
│   └── pyproject.toml                  ← Workspace uv
└── db/
    └── schema_gmao.sql                 ← Schéma BDD complet
```

---

## Technologies

<p align="center">
  <img src="https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white" alt="FastAPI">
  <img src="https://img.shields.io/badge/PyTorch-EE4C2C?style=flat-square&logo=pytorch&logoColor=white" alt="PyTorch">
  <img src="https://img.shields.io/badge/scikit--learn-F7931E?style=flat-square&logo=scikit-learn&logoColor=white" alt="scikit-learn">
  <img src="https://img.shields.io/badge/Qdrant-DC382C?style=flat-square&logo=qdrant&logoColor=white" alt="Qdrant">
  <img src="https://img.shields.io/badge/React-19-61DAFB?style=flat-square&logo=react&logoColor=black" alt="React">
  <img src="https://img.shields.io/badge/Tailwind_CSS-v4-06B6D4?style=flat-square&logo=tailwindcss&logoColor=white" alt="Tailwind">
  <img src="https://img.shields.io/badge/Laravel-13-FF2D20?style=flat-square&logo=laravel&logoColor=white" alt="Laravel">
  <img src="https://img.shields.io/badge/MySQL-8.0-4479A1?style=flat-square&logo=mysql&logoColor=white" alt="MySQL">
  <img src="https://img.shields.io/badge/OpenAI-412991?style=flat-square&logo=openai&logoColor=white" alt="OpenAI">
  <img src="https://img.shields.io/badge/Gemini-4285F4?style=flat-square&logo=google&logoColor=white" alt="Gemini">
  <img src="https://img.shields.io/badge/MLflow-0194E2?style=flat-square&logo=mlflow&logoColor=white" alt="MLflow">
  <img src="https://img.shields.io/badge/uv-DE5FE9?style=flat-square&logo=uv&logoColor=white" alt="uv">
</p>

---

<p align="center">
  <sub>GMAO © 2026 — Projet privé</sub>
</p>
