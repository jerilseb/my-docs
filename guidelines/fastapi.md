# Directory Structure

## App with plain HTML and JS Frontend

If the app has a frontend which uses plain html and javascript, adopt a directory structure like below.

```
├── .vscode
│   └── launch.json
├── main.py
├── static/
├── .env.example
└── requirements.txt
```

To serve static files use the static folder feature.

```python
from fastapi.staticfiles import StaticFiles

app.mount("/", StaticFiles(directory="static", html=True), name="static")
```

## App with a Vue Frontend

If the app has a frontend with Vue, adopt a directory structure as follows.

```
├── .vscode
│   ├── tasks.json
│   └── launch.json
├── backend
│   ├── main.py
│   └── requirements.txt
├── frontend
│   ├── src
│   │   └── App.vue
│   ├── main.js
│   ├── index.html
│   ├── vite.config.js
│   └── package.json
├── .dockerignore
└── Dockerfile
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

## Setup CORS

Allow all origins, methods and headers in CORS
```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

## Debugging Setup

Create launch.json and tasks.json to for debugging

launch.json
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "FastAPI",
            "type": "debugpy",
            "request": "launch",
            "program": "main.py",
            "console": "integratedTerminal",
            "justMyCode": true,
            "cwd": "${workspaceFolder}/backend",
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

## Uvicorn launch

At the end of `main.py`, add the following section to start the app

```python
if __name__ == "__main__":
    uvicorn.run("main:app", host="0.0.0.0", port=8000, reload=True)
```

## Setup API Routes

Use a `/api` prefix for all REST APIs. For example,

```python
router = APIRouter(prefix="/api")

@router.get("/notes")
async def get_notes():
    ...
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

COPY --from=frontend /frontend/dist /frontend/dist

EXPOSE 8000
USER appuser
CMD ["uvicorn", "backend.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Use a .dockerignore like this

```
backend/.env
backend/.venv
.vscode
frontend/node_modules
frontend/dist
__pycache__
```

# Implementing Login with Google

Add `python-jose` to the requirements. In the .env file add the following

```
# .env file
GOOGLE_CLIENT_ID="YOUR_GOOGLE_CLIENT_ID.apps.googleusercontent.com"
GOOGLE_CLIENT_SECRET="YOUR_GOOGLE_CLIENT_SECRET"
GOOGLE_REDIRECT_URI="http://localhost:8000/auth/callback"
```

Add the following api routes

```python
import requests
from jose import JWTError, jwt
from datetime import datetime, timedelta, timezone

@app.get("/auth/google")
async def login_google():
    auth_url = (
        "https://accounts.google.com/o/oauth2/v2/auth"
        "?response_type=code"
        f"&client_id={GOOGLE_CLIENT_ID}"
        f"&redirect_uri={GOOGLE_REDIRECT_URI}"
        "&scope=openid%20email%20profile"
    )
    return RedirectResponse(auth_url)


@app.get("/auth/callback")
async def auth_callback(code: str, db: Session = Depends(get_db)):
    token_url = "https://oauth2.googleapis.com/token"
    data = {
        "code": code,
        "client_id": GOOGLE_CLIENT_ID,
        "client_secret": GOOGLE_CLIENT_SECRET,
        "redirect_uri": GOOGLE_REDIRECT_URI,
        "grant_type": "authorization_code",
    }
    response = requests.post(token_url, data=data)
    token_data = response.json()
    id_token = token_data.get("id_token")
    google_access_token = token_data.get("access_token")

    if not id_token:
        raise HTTPException(
            status_code=400,
            detail="ID token not found in response from Google",
        )

    try:
        user_info = jwt.decode(
            id_token,
            key=None,
            options={"verify_signature": False},
            audience=GOOGLE_CLIENT_ID,
            access_token=google_access_token,
        )
        email = user_info.get("email")
        name = user_info.get("name")
        google_id = user_info.get("sub")
    except JWTError as e:
        print("Error", e)
        raise HTTPException(status_code=400, detail="Invalid ID token")

    # If server manages users, fetch/create user object here

    access_token = jwt.encode(
        {
            "sub": email,
            "exp": datetime.now(datetime.timezone.utc) + timedelta(minutes=60 * 12),
        },
        SECRET_KEY,
        algorithm="HS256"
    )

    redirect_response = RedirectResponse(url="/")
    redirect_response.set_cookie(
      key="access_token",
      value=access_token,
      httponly=False,
    )
    return redirect_response

```

The frontend code can read the cookie and log in the user. For example in Vue,
```vue
<script setup>
import { onMounted } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();

onMounted(() => {
  const token = document.cookie.split('; ').find(row => row.startsWith('access_token='));
  if (token && window.location.pathname === '/') {
    router.push('/todos');
  }
});
</script>
```

