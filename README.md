# pitagoras3.github.io

Sources for my small programming blog available on [pitagoras3.github.io](https://pitagoras3.github.io). 

To publish new post on blog just push changes to `main` branch. After that, GitHub action builds static pages and commits them to `gh-pages` branch. From there, they're available via GitHub Pages on [pitagoras3.github.io](https://pitagoras3.github.io).

```mermaid
flowchart LR
    id1([main branch]) --> id2([GitHub action]) --static pages--> id3([gh-pages branch])
```

___

This blog is using satnaing's Astro blog theme - [AstroPaper](https://github.com/satnaing/astro-paper).

#### Astro Project Structure

```bash
/
├── public/
│   ├── assets/
│   │   └── logo.svg
│   │   └── logo.png
│   └── favicon.svg
│   └── robots.txt
│   └── toggle-theme.js
├── src/
│   ├── assets/
│   │   └── socialIcons.ts
│   ├── components/
│   ├── contents/
│   │   └── some-blog-posts.md
│   ├── layouts/
│   └── pages/
│   └── styles/
│   └── utils/
│   └── config.ts
│   └── types.ts
└── package.json
```

Astro looks for `.astro` or `.md` files in the `src/pages/` directory. Each page is exposed as a route based on its file name.

Any static assets, like images, can be placed in the `public/` directory.

All blog posts are stored in `src/contents/` directory.

#### Commands

All commands are run from the root of the project, from a terminal:

| Command                | Action                                             |
| :--------------------- | :------------------------------------------------- |
| `npm install`          | Installs dependencies                              |
| `npm run dev`          | Starts local dev server at `localhost:3000`        |
| `npm run build`        | Build your production site to `./dist/`            |
| `npm run preview`      | Preview your build locally, before deploying       |
| `npm run format:check` | Check code format with Prettier                    |
| `npm run format`       | Format codes with Prettier                         |
| `npm run cz`           | Commit code changes with commitizen                |
| `npm run astro ...`    | Run CLI commands like `astro add`, `astro preview` |
| `npm run astro --help` | Get help using the Astro CLI                       |
