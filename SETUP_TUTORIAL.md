# Disease Prediction App - Complete Setup Guide

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Environment Setup](#environment-setup)
3. [Database Setup](#database-setup)
4. [Application Setup](#application-setup)
5. [Running the Application](#running-the-application)
6. [Testing](#testing)
7. [GitHub Actions CI/CD](#github-actions-cicd)
8. [Project Features](#project-features)

---

## Prerequisites

### Required Software
- **Python 3.12+** - [Download](https://www.python.org/downloads/)
- **Node.js 18+** - [Download](https://nodejs.org/)
- **PostgreSQL 15+** - [Download](https://www.postgresql.org/download/) or use Docker
- **Git** - [Download](https://git-scm.com/)
- **uv** (Python package manager) - Install with: `pip install uv`

### Optional Software
- **Docker Desktop** - For containerized database
- **VS Code** - Recommended IDE with Python extensions

---

## Environment Setup

### 1. Clone the Repository
```bash
git clone <your-repo-url>
cd DiseasePredictionApp
```

### 2. Create Python Virtual Environment
```bash
# Create virtual environment using uv
uv venv .venv

# Activate virtual environment
# On Windows:
.venv\Scripts\activate

# On macOS/Linux:
source .venv/bin/activate

# Upgrade pip
uv pip install --upgrade pip
```

### 3. Install Python Dependencies
```bash
uv pip install -r Project/requirements.txt
```

### 4. Install Frontend Dependencies (Optional - for linting)
```bash
cd Project/frontend
npm install
cd ../..
```

### 5. Configure Environment Variables
```bash
# Copy the example environment file
cp .env.example .env

# Edit the .env file with your settings
nano .env
```

**Required `.env` configuration:**
```env
# Database (choose one option)

# Option 1: Local PostgreSQL
NEON_DATABASE_URL=postgresql://postgres:postgres@localhost:5432/disease_prediction

# Option 2: Docker PostgreSQL (run command below first)
# NEON_DATABASE_URL=postgresql://postgres:postgres@localhost:5433/disease_prediction

# JWT Configuration (CHANGE IN PRODUCTION!)
JWT_SECRET=your-super-secret-key-minimum-32-characters-long
JWT_ALGORITHM=HS256
JWT_EXPIRES_SECONDS=3600

# Model Paths
MODEL_PATH=Project/model/disease_model.pkl
SYMPTOM_LIST_PATH=Project/model/symptom_list.pkl
DATA_DIR=Project/data

# CORS Origins
ALLOWED_ORIGINS=http://localhost:3000,http://127.0.0.1:5500,http://localhost:5500
```

---

## Database Setup

### Option 1: Local PostgreSQL Installation

1. **Install PostgreSQL** from [postgresql.org](https://www.postgresql.org/download/)

2. **Start PostgreSQL service:**
   ```bash
   # On Windows (run as Administrator):
   net start postgresql-x64-15

   # On macOS:
   brew services start postgresql@15

   # On Linux (Ubuntu/Debian):
   sudo systemctl start postgresql
   ```

3. **Create database and user:**
   ```bash
   psql -U postgres
   
   # In psql shell:
   CREATE DATABASE disease_prediction;
   CREATE USER postgres WITH PASSWORD 'postgres';
   GRANT ALL PRIVILEGES ON DATABASE disease_prediction TO postgres;
   \c disease_prediction
   GRANT ALL ON SCHEMA public TO postgres;
   \q
   ```

### Option 2: Docker PostgreSQL (Recommended)

1. **Install Docker Desktop** from [docker.com](https://www.docker.com/products/docker-desktop/)

2. **Start PostgreSQL container:**
   ```bash
   # Run PostgreSQL container
   docker run --name disease-prediction-db \
     -e POSTGRES_USER=postgres \
     -e POSTGRES_PASSWORD=postgres \
     -e POSTGRES_DB=disease_prediction \
     -p 5433:5432 \
     -d postgres:15-alpine

   # Verify container is running
   docker ps
   ```

3. **Test connection:**
   ```bash
   PGPASSWORD=postgres psql -h localhost -p 5433 -U postgres -d disease_prediction
   ```

4. **Update `.env` file:**
   ```env
   NEON_DATABASE_URL=postgresql://postgres:postgres@localhost:5433/disease_prediction
   ```

### Verify Database Connection
```bash
# Test database connection
.venv\Scripts\python -c "import os; from databases import Database; db = Database(os.getenv('NEON_DATABASE_URL')); print('Database connected!')"
```

---

## Application Setup

### 1. Initialize Database Tables
```bash
cd Project/backend

# Create all tables
.venv\Scripts\python -c "
import asyncio
from db import database, metadata, engine
from models import users, chats, messages

async def init_db():
    await database.connect()
    metadata.create_all(bind=engine)
    await database.disconnect()
    print('Database tables created!')

asyncio.run(init_db())
"
```

### 2. Train the Model (if not exists)
```bash
cd Project/model

# Train the disease prediction model
.venv\Scripts\python train_model.py

# Verify model was created
ls -la *.pkl
```

### 3. Prepare Frontend
```bash
cd Project/frontend

# The frontend is vanilla HTML/CSS/JS
# No build step required

# Optional: Install linting tools
npm install

# Run linting (optional)
npm run lint
```

---

## Running the Application

### 1. Start the Backend Server
```bash
cd Project/backend

# Start FastAPI server
.venv\Scripts\python -m uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

**Expected output:**
```
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [xxxx] using StatReload
INFO:     Started server process [xxxx]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

### 2. Start the Frontend
```bash
# Option 1: Using Live Server (VS Code extension)
# Open Project/frontend/index.html with Live Server

# Option 2: Using Python HTTP server
cd Project/frontend
.venv\Scripts\python -m http.server 5500

# Option 3: Using npm serve (if installed)
npx serve .
```

### 3. Access the Application
- **Frontend:** http://localhost:5500
- **Backend API:** http://localhost:8000
- **API Documentation:** http://localhost:8000/docs

---

## Testing

### Run Backend Tests
```bash
cd Project/backend

# Run all tests
.venv\Scripts\python -m pytest test_api.py -v

# Run with coverage
.venv\Scripts\python -m pytest test_api.py -v --cov=.

# Run specific test
.venv\Scripts\python -m pytest test_api.py::TestHealthCheck -v
```

### Test Endpoints Manually

Using `curl`:
```bash
# Health check
curl http://localhost:8000/

# Register user
curl -X POST http://localhost:8000/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "fullName": "Test User",
    "email": "test@example.com",
    "password": "password123",
    "dob": "1990-01-01",
    "gender": "male",
    "nationality": "USA"
  }'

# Login
curl -X POST http://localhost:8000/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "password123"
  }'

# Predict disease (replace TOKEN with actual token)
curl -X POST http://localhost:8000/predict_text \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -d '{"user_input": "I have fever and cough"}'

# Get disease details
curl "http://localhost:8000/get_details?disease=fever"

# Get chat history
curl http://localhost:8000/chats \
  -H "Authorization: Bearer TOKEN"

# Get user profile
curl http://localhost:8000/user/profile \
  -H "Authorization: Bearer TOKEN"
```

Using HTTPie:
```bash
# Install HTTPie
uv pip install httpie

# Register
http POST http://localhost:8000/auth/register fullName="Test User" email="test@example.com" password="password123" dob="1990-01-01" gender="male" nationality="USA"

# Login
http POST http://localhost:8000/auth/login email="test@example.com" password="password123"

# Predict (save token from login response)
http POST http://localhost:8000/predict_text user_input="I have fever and cough" "Authorization: Bearer YOUR_TOKEN"

# Get profile
http http://localhost:8000/user/profile "Authorization: Bearer YOUR_TOKEN"
```

### Test Frontend
```bash
# Open browser and navigate to http://localhost:5500
# Test the following:
# 1. User registration
# 2. User login
# 3. Start new chat
# 4. Enter symptoms
# 5. View prediction results
# 6. Open profile modal
# 7. Check chat history sidebar
# 8. Logout
```

---

## GitHub Actions CI/CD

### Setup GitHub Repository
1. **Create GitHub repository** at https://github.com/new
2. **Push code:**
   ```bash
   git init
   git add .
   git commit -m "Initial commit: Disease Prediction App with chat features"
   git branch -M main
   git remote add origin <your-github-repo-url>
   git push -u origin main
   ```

### Configure GitHub Secrets
Go to your repository → Settings → Secrets and variables → Actions

Add the following secrets:
- `NEON_DATABASE_URL` - Your PostgreSQL connection string (for production)
- `JWT_SECRET` - Your JWT secret key (for production)
- `DOCKER_PASSWORD` - Docker Hub password (if using Docker Hub)

### GitHub Actions Workflow
The workflow (`.github/workflows/ci-cd.yml`) runs:

1. **Security Scan** - Bandit, Safety, Semgrep
2. **Lint & Format** - Black, isort, flake8, mypy
3. **Backend Tests** - pytest with PostgreSQL service
4. **Frontend Tests** - ESLint, HTML validation
5. **Dependency Check** - pip-audit, npm audit
6. **Docker Build** - Only on main branch
7. **Deploy Preview** - On pull requests

### Monitor Workflow
1. Go to your repository → Actions tab
2. Watch the workflow run
3. Check for any failures
4. Review security scan reports

### Manual Trigger
You can manually trigger the workflow:
1. Go to Actions → CI/CD Pipeline
2. Click "Run workflow"
3. Select branch and run

---

## Project Features

### 1. ChatGPT-like Chat Interface
- Interactive chat UI with animated messages
- Typing indicator during predictions
- Message bubbles (user/assistant)
- Auto-expanding input textarea
- Responsive design for mobile

### 2. Chat History System
- Create new chat sessions
- View previous chats in sidebar
- Load old conversations
- Automatic title generation from first message
- Persistent storage in PostgreSQL

### 3. User Profile Management
- View profile information
- Edit profile (name, DOB, gender, nationality)
- Change password
- View statistics (total chats, messages)
- Modal-based UI

### 4. Disease Prediction
- Natural language symptom input
- ML-based disease prediction
- Confidence scores
- Disease descriptions
- Precaution recommendations

### 5. Security Features
- JWT token authentication
- Password hashing (bcrypt_sha256)
- Input validation and sanitization
- SQL injection prevention
- XSS protection
- CORS configuration
- Path traversal prevention

### 6. API Endpoints

**Authentication:**
- `POST /auth/register` - Register new user
- `POST /auth/login` - Login and get token

**Prediction:**
- `POST /predict_text` - Predict disease from symptoms
- `GET /get_details` - Get disease description and precautions

**Chat History:**
- `GET /chats` - List all chats
- `POST /chats` - Create new chat
- `GET /chats/{id}` - Get chat with messages
- `DELETE /chats/{id}` - Delete chat
- `POST /chats/{id}/messages` - Add message

**User Profile:**
- `GET /user/profile` - Get profile
- `PUT /user/profile` - Update profile
- `POST /user/change-password` - Change password
- `GET /user/chat-stats` - Get statistics

---

## Troubleshooting

### Common Issues

#### 1. Database Connection Failed
```bash
# Check PostgreSQL is running
pg_isready -h localhost -p 5432

# Check connection string
echo $NEON_DATABASE_URL

# Test with Python
.venv\Scripts\python -c "
import os
from databases import Database
db = Database(os.getenv('NEON_DATABASE_URL'))
print('Connected!' if db else 'Failed!')
"
```

#### 2. Model File Not Found
```bash
# Check model files exist
ls -la Project/model/

# If missing, train the model
cd Project/model
.venv\Scripts\python train_model.py
```

#### 3. CORS Errors
```bash
# Update ALLOWED_ORIGINS in .env
ALLOWED_ORIGINS=http://localhost:3000,http://127.0.0.1:5500

# Restart backend server
```

#### 4. JWT Token Issues
```bash
# Generate new secret
.venv\Scripts\python -c "import secrets; print(secrets.token_hex(32))"

# Update JWT_SECRET in .env
```

#### 5. Port Already in Use
```bash
# Find process using port
netstat -ano | findstr :8000

# Kill process (replace PID)
taskkill /PID <PID> /F

# Or use different port
.venv\Scripts\python -m uvicorn main:app --port 8001
```

### Getting Help
1. Check the API documentation at http://localhost:8000/docs
2. Check browser console for frontend errors
3. Check backend console for server errors
4. Review the GitHub Actions workflow logs

---

## Development Tips

### Code Style
```bash
# Format Python code
black Project/backend/

# Sort imports
isort Project/backend/

# Lint code
flake8 Project/backend/
mypy Project/backend/
```

### Adding New Features
1. Create new endpoint in `main.py`
2. Add Pydantic models for request/response
3. Update frontend JavaScript in `script.js`
4. Add tests in `test_api.py`
5. Update this tutorial!

### Database Migrations
```bash
# For production, use Alembic or Drizzle
# See drizzle.config.ts if using Drizzle
```

---

## Production Deployment Checklist

- [ ] Generate strong `JWT_SECRET` (32+ random characters)
- [ ] Configure production database (not localhost)
- [ ] Set `ALLOWED_ORIGINS` to production domain only
- [ ] Enable HTTPS/TLS
- [ ] Set up proper logging
- [ ] Configure rate limiting
- [ ] Set up monitoring and alerting
- [ ] Run security scan and fix all issues
- [ ] Test all endpoints with production database
- [ ] Document all environment variables
- [ ] Set up backup strategy for database
- [ ] Configure CORS for production domain
- [ ] Set up CI/CD secrets in GitHub

---

## License

This project is for educational purposes. Always consult healthcare professionals for medical advice.

---

## Support

For issues and questions:
1. Check the troubleshooting section
2. Review GitHub Actions logs
3. Open a GitHub issue
4. Check FastAPI documentation

---

**Last Updated:** January 2026
**Version:** 1.0.0
