## Project structure

Use the following as a reference on how to structure the project

```
├── icons
│   ├── 128.png
│   ├── 16.png
│   └── 48.png
├── LICENSE
├── manifest.json
├── options
│   ├── options.css
│   └── options.html
├── popup
│   ├── popup.css
│   └── popup.html
├── content
│   ├── content.css
│   └── content.html
├── background
│   └── service-worker.js
└── README.md
```

# Manifest

Use the following code as a reference for the manifest file

```json
{
    "manifest_version": 3,
    "name": "Chrome Extension v3 Starter",
    "description": "A minimal example of a chrome extension using manifest v3",
    "version": "0.0.1",
    "icons": {
        "16": "icons/16.png",
        "48": "icons/48.png",
        "128": "icons/128.png"
    },
    "options_page": "options/options.html",
    "action": {
        "default_title": "Chrome Addon v3 Starter",
        "default_popup": "popup/popup.html"
    },
    "permissions": [],
    "host_permissions": [
        "*://*/*"
    ],
    "background": {
        "service_worker": "service-worker.js"
    },
    "content_scripts": [{
        "js": ["content/content.js"],
        "css": ["content/content.css"],
        "matches": ["https://github.com/*"]
    }]
}
```

# Icons

Icons will be created an placed in the icons folder by the user. Just create the icons folder and leave it empty