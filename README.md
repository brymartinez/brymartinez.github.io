# brymartinez.github.io

Starter Astro site for a multi-page personal website and blog, designed to deploy to GitHub Pages for free.

## What is included

- Multi-page site structure: Home, About, Projects, Blog, Notes
- Content collections for blog posts and technical notes
- Markdown/MDX-ready content workflow
- Static Astro setup that fits `brymartinez.github.io`
- GitHub Actions workflow for Pages deployment

## Local setup

Use Node `22.12.0` or newer.

```bash
npm install
npm run dev
```

Open the local URL Astro prints in the terminal.

## Content workflow

- Add blog posts in `src/content/blog`
- Add technical notes or standalone knowledge pages in `src/content/notes`
- Edit the top-level pages in `src/pages`

## Deploying to GitHub Pages

1. Create a GitHub repo named `brymartinez.github.io`
2. Push this project to that repo
3. In GitHub, enable Pages with GitHub Actions as the source
4. Push to `main` to trigger the included deploy workflow
