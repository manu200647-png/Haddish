services:
  # 1. FastAPI API Backend
  - type: web
    name: tigrinya-api
    runtime: python
    rootDir: backend
    buildCommand: "pip install -r requirements.txt"
    startCommand: "uvicorn app.main:app --host 0.0.0.0 --port $PORT"
    envVars:
      - key: DATABASE_URL
        fromDatabase:
          name: tigrinya-db
          property: connectionString
      - key: REDIS_URL
        fromService:
          type: redis
          name: tigrinya-redis
          property: connectionString
      - key: CELERY_RESULT_BACKEND
        fromService:
          type: redis
          name: tigrinya-redis
          property: connectionString
      - key: JWT_SECRET_KEY
        generateValue: true
      - key: SIGNING_SECRET_KEY
        generateValue: true

  # 2. Celery Worker (Track Changes & Docx Generator)
  - type: worker
    name: tigrinya-worker
    runtime: python
    rootDir: backend
    buildCommand: "pip install -r requirements.txt"
    startCommand: "celery -A app.tasks.celery_app worker --loglevel=info --concurrency=2"
    envVars:
      - key: DATABASE_URL
        fromDatabase:
          name: tigrinya-db
          property: connectionString
      - key: REDIS_URL
        fromService:
          type: redis
          name: tigrinya-redis
          property: connectionString
      - key: CELERY_RESULT_BACKEND
        fromService:
          type: redis
          name: tigrinya-redis
          property: connectionString

  # 3. React Frontend Website
  - type: web
    name: tigrinya-frontend
    runtime: static
    rootDir: frontend
    buildCommand: "npm install && npm run build"
    staticPublishPath: "./dist"
    routes:
      - type: rewrite
        source: "/*"
        destination: "/index.html"
    envVars:
      - key: VITE_API_URL
        fromService:
          type: web
          name: tigrinya-api
          property: host

# 4. Managed PostgreSQL Database
databases:
  - name: tigrinya-db
    databaseName: tigrinyadb
    user: postgres
    plan: free

# 5. Managed Redis Broker
redis:
  - name: tigrinya-redis
    plan: free
    ipAllowList: []
render. yaml
