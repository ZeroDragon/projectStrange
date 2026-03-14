# Project Structure

## Overview
`doctorstrange` is a Vue 3 application for loading and managing ROM databases. Built with Vite, using Pug for templates and Stylus for styling.

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
│       └── Home.vue       # ROM loading and management
├── index.html             # HTML entry point
├── package.json           # Project dependencies and scripts
└── vite.config.js         # Vite configuration

```

## Available Scripts
- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run preview` - Preview production build

## Application Features

### ROM Management System
The system manages two ROM databases:
- **BIOS**: Database loaded from CSV format (the old ROM)
- **ROM**: Database loaded from CSV format (the new ROM)

#### CSV Format
- 6 columns (no headers)
- All cells contain integers
- One record per line

#### Data Storage
- Data is stored in `localStorage` as an array of arrays
- Structure: `{ bios: [[col1, col2, col3, col4, col5, col6], ...], rom: [[col1, col2, col3, col4, col5, col6], ...] }`

#### Operations
- **Upload**: Load CSV files for both BIOS and ROM
- **Persistence**: Data persists across sessions via localStorage
- **Clear Memory**: Remove all data from memory and localStorage

## Configuration Details

### Vite Config (vite.config.js)
- Uses `@vitejs/plugin-vue` for Vue support
- Alias `@` points to `src/` directory for cleaner imports
- Stylus auto-imports `src/styles/variables.styl` in all files

### Ignored Directories
- `oldstuff/` - Legacy files, not part of the repository
- `rom/` - Temporary files, not part of the repository
