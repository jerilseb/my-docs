# Creating a new Project

```
npm create vite@latest my-vue-app -- --template vue
```

# General Guidelines

1. Always use Vue 3 Composition API


# Important Files

---

File: vite.config.js

Keep it simple
```javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()]
})
```