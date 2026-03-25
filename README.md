# ETX Agentic AI Showroom

Lab guide for the ETX Agentic AI workshop, built with [Docsify](https://docsify.js.org/).

## Local Development

### Quick Start

1. Install Docsify CLI (optional, for local preview):

```bash
npm install -g docsify-cli
```

2. Serve the docs locally:

```bash
docsify serve docs
```

3. Open http://localhost:3000 in your browser.

### Without Docsify CLI

You can use any static file server to serve the `docs/` directory. For example, with Python:

```bash
cd docs && python -m http.server 3000
```

## Project Structure

```
docs/
├── index.html          # Docsify entry point and configuration
├── _sidebar.md         # Navigation sidebar
├── README.md           # Home page content
├── module-*.md         # Lab module pages
├── .nojekyll           # Prevents GitHub Pages Jekyll processing
├── css/
│   └── custom.css      # Custom styles
└── images/             # Screenshots and diagrams
```

## Deployment

The site is automatically deployed to GitHub Pages on push to `main` via `.github/workflows/gh-pages.yml`. Docsify renders content client-side, so no build step is needed — the `docs/` folder is deployed directly.

## Lab Variable Substitution

Lab environment variables (usernames, passwords, cluster URLs) are defined in `docs/index.html` under the `defined_vars` object. Update these values for your specific deployment. They are referenced in the markdown files using `{{variable_name}}` syntax and substituted at render time by Docsify's `beforeEach` hook.
