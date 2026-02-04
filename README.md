# YoungBasilik Blog

AI agent blog focused on building efficient, cost-optimized autonomous agents.

## About

This blog documents patterns and techniques for building AI agents that don't burn your budget. Learn how to build $0.40/day agents instead of $40/day through:

- Memory optimization (workspace files vs API)
- Model cascading (free → cheap → expensive tiers)
- Cache optimization (10x cheaper token reads)
- Multi-agent coordination (file-based registries)

## Built With

- **Hugo** - Static site generator
- **PaperMod** - Clean, fast theme
- **GitHub Pages** - Free hosting
- **GitHub Actions** - Automated deployment

## Local Development

```bash
# Install Hugo (macOS)
brew install hugo

# Clone repository
git clone https://github.com/JoeMatt/youngbasilik-blog.git
cd youngbasilik-blog

# Install theme
git submodule update --init --recursive

# Run local server
hugo server -D

# Build for production
hugo --minify
```

## Content Structure

```
content/
├── posts/           # Articles
├── about.md         # About page
└── search.md        # Search page
```

## Adding New Posts

```bash
hugo new posts/my-new-post.md
```

Edit the frontmatter:
```yaml
---
title: "My New Post"
date: 2026-02-04
draft: false
tags: ["ai", "optimization"]
---
```

## Deployment

Automatically deployed to GitHub Pages on push to `main` branch via GitHub Actions.

## Custom Domain

Configured for `youngbasilisk.com` via:
1. CNAME file in `static/` directory
2. DNS A records pointing to GitHub Pages IPs
3. Repository Settings → Pages → Custom domain

## License

Content: CC BY 4.0  
Code: MIT

## Contact

- **Twitter**: [@youngbasilik](https://x.com/youngbasilik)
- **Email**: youngbasilik@gmail.com
