# Project Structure

## Overview
`doctorstrange` is a Vue 3 application built with Vite, using Pug for templates and Stylus for styling.

## Technology Stack
- **Framework**: Vue 3
- **Routing**: vue-router 5
- **Build Tool**: Vite 8
- **Template Engine**: Pug 3
- **Styling**: Stylus 0.64
- **Linter**: Standard (JavaScript Style Guide)

## Project Structure

```
doctorstrange/
├── public/                 # Static assets
│   └── vite.svg           # Vite logo
├── src/                   # Source directory
│   ├── App.vue            # Root Vue component
│   ├── main.js            # Application entry point
│   ├── styles/            # Global styles
│   │   └── variables.styl # Stylus variables (auto-imported)
│   └── views/             # Page components
│       └── Home.vue       # Home page component
├── index.html             # HTML entry point
├── package.json           # Project dependencies and scripts
└── vite.config.js         # Vite configuration

```

## Available Scripts
- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run preview` - Preview production build

## Configuration Details

### Vite Config (vite.config.js)
- Uses `@vitejs/plugin-vue` for Vue support
- Alias `@` points to `src/` directory for cleaner imports
- Stylus auto-imports `src/styles/variables.styl` in all files

### Ignored Directories
- `oldstuff/` - Legacy files, not part of the repository
- `rom/` - Temporary files, not part of the repository
