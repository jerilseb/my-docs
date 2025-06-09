# Directory Structure

## App with plain HTML and JS Frontend

If the app has a frontend which uses plain html and javascript, adopt a directory structure like below.

```
├── main.py
├── static/
├── .env.example
└── requirements.txt
```

To serve static files use the static folder feature.
```
from fastapi.staticfiles import StaticFiles

app.mount("/", StaticFiles(directory="static", html=True), name="static")
```

DO NOT use StaticFiles if the app uses a frontend framework like Vue.

## App with a Vue Frontend

If the app has a frontend which Vue as the frontend, adopt a directory structure as follows.

```
├── backend
│   ├── main.py
│   └── requirements.txt
└── frontend
    └── src
    │    └── App.vue
    ├── main.js
    ├── index.html
    ├── vite.config.js
    └── package.json
```

# General Guidelines

1. Create a virtualenv named `.venv` inside `backend` directory and install dependencies ino that.

2. Use python-dotenv package for loading dotfiles

3. No need to create a README file. User will create it manually, if needed

4. Use lifespan for managing app lifecycle

```
@asynccontextmanager
async def lifespan(app: FastAPI):
  ...

app = FastAPI(lifespan=lifespan)
```

5. Allow all origins, methods and headers in CORS
```
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```