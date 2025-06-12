# Directory Structure

If the app has a frontend with Vue, adopt a directory structure as follows.

```
├── .vscode
│   ├── tasks.json
│   └── launch.json
├── backend
│   ├── main.py
│   └── requirements.txt
└── frontend
    ├── src
    │   └── App.vue
    ├── main.js
    ├── index.html
    ├── vite.config.js
    └── package.json
```

To serve static files use the static folder feature.

```python
from fastapi.staticfiles import StaticFiles

app.mount("/", StaticFiles(directory="../frontend/dist", html=True), name="static")
```

# Development Guidelines

1. Create a virtualenv named `.venv` inside `backend` directory and install dependencies ino that.

2. Use python-dotenv package for loading dotfiles

3. No need to create a README file. User will create it manually, if needed

## Setup Lifespan

Use lifespan for managing app lifecycle

```python
@asynccontextmanager
async def lifespan(app: FastAPI):
  ...

app = FastAPI(lifespan=lifespan)
```

## Debugging Setup

Create launch.json and tasks.json to for debugging

launch.json
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "FastAPI (uvicorn)",
            "type": "debugpy",
            "request": "launch",
            "module": "uvicorn",
            "args": [
                "main:app",
                "--reload"
            ],
            "cwd": "${workspaceFolder}/backend",
            "console": "integratedTerminal",
            "justMyCode": true,
            "python": "${workspaceFolder}/backend/.venv/bin/python",
            "preLaunchTask": "npm: build:watch"
        }
    ]
}
```

tasks.json
```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "npm: build:watch",
            "type": "npm",
            "script": "build:watch",
            "path": "frontend/",
            "problemMatcher": [
                {
                    "base": "$tsc-watch",
                    "background": {
                        "activeOnStart": true,
                        "beginsPattern": "^.*build started.*$",
                        "endsPattern": "^.*built in \\d+ms\\.$"
                    }
                }
            ],
            "isBackground": true
        }
    ]
}
```

Modify the package.json to add a `build:watch` script.

```json
  "scripts": {
    "dev": "vite",
    "build:watch": "vite build --watch",
    "build": "vite build",
  },
```


## Setup API Routes

Use a `/api` prefix for all REST APIs. For example,

```python
router = APIRouter(prefix="/api")

@router.get("/notes")
async def get_notes():
    ...
```

# Deploying to Production (Kubernetes)

First we need to create a Dockerfile and helm charts at the root

```
├── .vscode/
├── backend/
├── frontend/
├── helm/
│   ├── Chart.yaml
│   └── values.yaml
│   └── templates
│       ├── api.yaml
│       ├── ingress.yaml
│       └── namespace.yaml
├── .dockerignore
└── Dockerfile
```

## Dockerfile setup

We will use multi-stage build for the dockerfile

```dockerfile
FROM node:lts-slim AS frontend

WORKDIR /frontend
COPY frontend/package*.json ./
RUN npm install
COPY frontend/ ./
RUN npm run build


FROM python:3.12-slim-bookworm

RUN adduser --disabled-password appuser

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app
COPY ./backend/requirements.txt ./
RUN pip install -r requirements.txt

COPY ./ ./

COPY --from=frontend /frontend/dist ./frontend/dist

EXPOSE 8000
USER appuser

ENV FORWARDED_ALLOW_IPS="*"
ENV PROXY_HEADERS="true"

WORKDIR /app/backend
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Use a .dockerignore like this

```
backend/.env*
backend/.venv
.vscode
frontend/node_modules
frontend/dist
__pycache__
```

## Helm Charts

Given below is are example helm chart files for an app named demo. Change the name according to your app.

api.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-api-deployment
  namespace: {{ .Release.Name }}
spec:
  selector:
    matchLabels:
      app: demo-api
  replicas: {{ .Values.api.replicas }}
  template:
    metadata:
      labels:
        app: demo-api
    spec:
      tolerations:
        - key: "lifecycle"
          operator: "Equal"
          value: "spot"
      containers:
        - name: demo-api
          image: "{{ .Values.api.image.repository }}:{{ required "api.image.tag is required" .Values.api.image.tag }}"
          imagePullPolicy: Always
          command:
            ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "9999"]
          ports:
            - containerPort: 9999
          envFrom:
          - secretRef:
              name: demo-secrets

---
apiVersion: v1
kind: Service
metadata:
  name: demo-api-service
  namespace: {{ .Release.Name }}
spec:
  type: ClusterIP
  ports:
    - port: 9999
      targetPort: 9999
  selector:
    app: demo-api
```

ingress.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-api-ingress
  namespace: {{ .Release.Name }}
  annotations:
    cert-manager.io/cluster-issuer: {{ .Values.cert.issuer }}
spec:
  ingressClassName: nginx
  rules:
    - host: {{ .Values.ingress.host }}
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: demo-api-service
                port:
                  number: 9999

  tls:
    - hosts:
        - {{ .Values.ingress.host }}
      secretName: demo-ingress-tls
```

namespace.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Release.Name }}
```

Chart.yaml
```yaml
apiVersion: v2
name: Demo App
description: A Helm chart for Demo App
type: application
version: 0.3.0
appVersion: "1.16.0"
```

values.yaml
```yaml
environment: production

api:
    replicas: 1
    image:
        repository:
        tag: latest

ingress:
    host: 

cert:
    issuer:
```