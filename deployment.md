# Deployment Strategy for News Synthesizer

## Deployment Philosophy
The application requires careful deployment due to local LLM model dependency, GPU requirements, and real-time processing needs. Focus on isolated environments, reliable model loading, and scalable RSS processing.

## Infrastructure Requirements

### Hardware Requirements
- **Minimum:** CPU with AVX2, 8GB RAM, 10GB storage
- **Recommended:** NVIDIA GPU (RTX 30-series+), 16GB+ RAM, SSD storage
- **Local Models:** GPU acceleration for llama.cpp inference

### Software Environment
- **Backend:** Python 3.8+, FastAPI, llama.cpp compiled with GPU support
- **Frontend:** Node.js 18+, Next.js build tools
- **Model:** Pre-downloaded GGUF file (mlabonne_gemma-3-27b-it-abliterated-IQ4_XS.gguf)

## Deployment Options

### 1. Local Development
- **Setup:**
  ```bash
  cd backend
  python -m venv venv
  source venv/bin/activate
  pip install -r requirements.txt
  cp llama.cpp path/to/compiled/llamacpp
  ```
- **Frontend:** `npm install && npm run dev`
- **Backend:** `uvicorn main:app --reload`

### 2. Docker Containerization
- **Backend Dockerfile:**
  ```dockerfile
  FROM python:3.11-slim
  # Install system dependencies for llama.cpp
  RUN apt-get update && apt-get install -y cmake build-essential
  # Copy llama.cpp and compile
  COPY llama.cpp /llama.cpp
  RUN cd /llama.cpp && mkdir build && cd build && cmake .. && make
  # Install Python dependencies
  COPY requirements.txt .
  RUN pip install -r requirements.txt
  # Copy model file
  COPY mlabonne_gemma-3-27b-it-abliterated-IQ4_XS.gguf /models/
  COPY . .
  CMD ["uvicorn", "main:app", "--host", "0.0.0.0"]
  ```
- **Frontend Dockerfile:**
  ```dockerfile
  FROM node:18-alpine
  COPY package*.json .
  RUN npm ci
  COPY . .
  RUN npm run build
  CMD ["npm", "start"]
  ```
- **Docker Compose:**
  ```yaml
  version: '3.8'
  services:
    backend:
      build: ./backend
      ports:
        - "8000:8000"
      volumes:
        - ./backend:/app
    frontend:
      build: ./frontend
      ports:
        - "3000:3000"
      environment:
        - NEXT_PUBLIC_API_URL=http://backend:8000
  ```

### 3. Production Deployment
- **Cloud Options:** AWS EC2 with GPU instances, Google Cloud Compute Engine
- **Cost Considerations:** GPU instances ($0.50+/hour), focus on spot instances
- **Scaling:** Horizontal scaling for multiple users, model loading optimization

## CI/CD Pipeline

### GitHub Actions Configuration
```yaml
name: CI/CD
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - name: Install dependencies
        run: pip install -r backend/requirements.txt
      - name: Run tests
        run: cd backend && pytest
  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to server
        # Add deployment steps (Docker build and push, etc.)
```

### Automated Pipelines
- **Build Step:** Compile llama.cpp if needed
- **Test Step:** Run unit/integration tests
- **Security Scan:** Dependency and code analysis
- **Deploy Step:** Docker image push to registry
- **Rollback:** Automated rollback on deployment failures

## Environment Configuration

### Environment Variables

#### Backend Configuration
```bash
# LLM Model Settings
LLAMA_CPP_MODEL_PATH=/path/to/mlabonne_gemma-3-27b-it-abliterated-IQ4_XS.gguf
LLM_CONTEXT_SIZE=4096
LLM_GPU_LAYERS=35  # Adjust based on GPU VRAM
LLM_MAX_TOKENS=300

# Database Configuration
DATABASE_URL=sqlite:///news_synthesizer.db

# RSS Processing
RSS_FETCH_CONCURRENCY=10
RSS_FETCH_TIMEOUT=30

# Application Settings
HOST=0.0.0.0
PORT=8000
LOG_LEVEL=INFO
AUDIO_CACHE_DIR=./audio_cache

# API Configuration
API_V1_PREFIX=/api/v1
SECRET_KEY=your-secret-key-here  # Change in production!

# TTS Configuration
TTS_DEFAULT_VOICE=en-US-AriaNeural
TTS_DEFAULT_SPEED=1.0
TTS_CACHE_ENABLED=true
```

#### Frontend Configuration
```bash
# API Configuration
NEXT_PUBLIC_API_BASE_URL=http://localhost:8000
NEXT_PUBLIC_API_TIMEOUT=30000

# Application Settings
NEXT_PUBLIC_APP_NAME="News Synthesizer"
NEXT_PUBLIC_APP_VERSION=1.0.0

# Feature Flags
NEXT_PUBLIC_ENABLE_TTS=true
NEXT_PUBLIC_ENABLE_CHAT=true
NEXT_PUBLIC_MAX_ARTICLES=100

# Audio/TTS Settings
NEXT_PUBLIC_TTS_VOICE=en-US-AriaNeural
NEXT_PUBLIC_TTS_VOICES_AVAILABLE=en-US-AriaNeural,en-US-GuyNeural,en-GB-SoniaNeural,en-AU-NatashaNeural

# Development
NODE_ENV=development
```

## Monitoring and Operations

### Performance Monitoring
- **Model Performance:** Monitor inference latency and memory usage
  - Target: <5s per article for metadata generation
  - Alert if inference time >10s consistently
  - Track GPU memory usage and temperature

### System Health
- **RSS Feed Health:** Alert on failed fetches or parsing errors
  - Monitor feed availability (uptime %)
  - Track parsing success rates
  - Alert on feed content changes

- **User Experience:** Track chat interaction quality and TTS generation
  - Measure TTS generation time (<3s target)
  - Monitor audio cache hit rates
  - Track user satisfaction metrics

- **Logging:** Structured logs for debugging RAG and persona composition
  - JSON-formatted logs with timestamps
  - Separate log levels for different components
  - Retention policy: 30 days for operational logs

### Alerting Strategy
```yaml
alerts:
  - name: High LLM Inference Latency
    condition: avg_inference_time > 10s
    severity: warning
    
  - name: RSS Feed Failures
    condition: failed_fetches > 5 in 1h
    severity: error
    
  - name: Disk Space Critical
    condition: disk_usage > 90%
    severity: critical
    
  - name: Model Not Loaded
    condition: model_status == "error"
    severity: critical
```

### Backup Strategy
- **Daily Backups:** Database and configuration files
- **Weekly Backups:** Model files and persona configurations
- **Monthly Archives:** Complete system snapshots
- **Retention:** 7 daily, 4 weekly, 12 monthly backups

## Production Deployment Considerations

### Scaling Strategy
- **Vertical Scaling:** Increase GPU memory for larger models
- **Horizontal Scaling:** Multiple backend instances with load balancer
- **Caching Layer:** Redis for article metadata and synthesis results
- **CDN:** For static frontend assets

### Security Hardening
- Change default SECRET_KEY
- Enable HTTPS with valid certificates
- Implement rate limiting (10 req/min per IP)
- Regular security updates and patches
- Model file access controls (chmod 600)

### Performance Optimization
- Enable gzip compression
- Implement database connection pooling
- Use production ASGI server (Gunicorn + Uvicorn workers)
- Optimize Next.js build with compression
- Enable HTTP/2 and keep-alive connections

## Checklist
- [ ] Docker containers built and tested
- [ ] CI/CD pipeline configured
- [ ] Environment variables documented and secured
- [ ] Monitoring and alerting setup
- [ ] Backup procedures implemented
- [ ] Performance benchmarks established
- [ ] Security hardening completed
- [ ] Documentation updated

## Ledger
| Component | Deployment Status | Last Updated | Notes |
|-----------|-------------------|--------------|-------|
| Backend Container | Not Tested | 2025-10-28 | llama.cpp compilation needed |
| Frontend Build | Planned | 2025-10-28 | Next.js static export options |
| CI Pipeline | Planned | 2025-10-28 | GitHub Actions for automated tests |
| Production Scaling | Designed | 2025-10-28 | GPU instance requirements documented |
| Monitoring Setup | Planned | 2025-10-28 | Prometheus + Grafana integration |
| Backup Strategy | Documented | 2025-10-28 | Automated backup scripts needed |

---

*Document Version: 1.1 | Last Updated: 2025-10-28 | Review Frequency: Monthly*
