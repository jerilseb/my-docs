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

1. Use python-dotenv package for loading dotfiles

2. No need to create a README file. User will create it manually, if needed


# Important Files

---

File: requirements.txt

To get the latest versions of all packages, don't specify a version. For example,
```
fastapi
uvicorn
websockets
```

File: vite.config.js

Keep it simple
```
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()]
})
```