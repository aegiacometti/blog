# Adrian Giacometti Blog

Blog personal sobre automatización, NetDevOps, y DevOps construido con Hugo y el tema PaperMod.

🌐 **Sitio web**: [adriangiacometti.net](https://adriangiacometti.net)

## Desarrollo Local

### Requisitos Previos

- [Hugo Extended](https://gohugo.io/installation/) v0.155.3 o superior
- Git

### Instalación

```bash
# Clonar el repositorio
git clone https://github.com/[tu-usuario]/blog.git
cd blog

# Inicializar submódulos (tema)
git submodule update --init --recursive
```

### Comandos de Hugo

```bash
# Iniciar servidor de desarrollo
hugo server -D

# Construir sitio para producción
hugo --minify

# Crear nuevo post
hugo new content/posts/mi-nuevo-post.md
```

El sitio estará disponible en `http://localhost:1313`

## Estructura del Proyecto
## Deployment
Resumen rápido:
# Adrian Giacometti Blog

Personal blog about automation, NetDevOps, and DevOps built with Hugo and the PaperMod theme.

🌐 **Website**: [adriangiacometti.net](https://adriangiacometti.net)

## Local Development

### Prerequisites

- [Hugo Extended](https://gohugo.io/installation/) v0.155.3 or later
- Git

### Installation

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/blog.git
cd blog

# Initialize submodules (theme)
git submodule update --init --recursive
```

### Hugo Commands

```bash
# Start the development server
hugo server -D

# Build the site for production
hugo --minify

# Create a new post
hugo new content/posts/my-new-post.md
```

The site will be available at `http://localhost:1313` when running the dev server.

## Project Structure

```
.
├── content/          # Site content
│   ├── posts/        # Blog posts
│   └── about.md      # About page
├── static/           # Static files
│   ├── images/       # Images
│   └── CNAME         # Custom domain config
├── themes/           # Hugo themes
│   └── PaperMod/     # PaperMod theme
├── .github/
│   └── workflows/    # CI workflows
└── hugo.toml         # Hugo configuration
```

## Deployment

The site is hosted on **Cloudflare Pages**. For full setup instructions, custom domain configuration, SSL, and automated deployments, see [CLOUDFLARE_DEPLOYMENT.md](CLOUDFLARE_DEPLOYMENT.md).

Quick summary:

- **Hosting platform:** Cloudflare Pages
- **Production branch:** `main`
- **Build command:** `hugo --minify`
- **Output directory:** `public`
- **Custom domain:** `adriangiacometti.net` is recorded in `static/CNAME` and DNS/SSL are managed in Cloudflare (CNAME/apex pointing to the Pages project, proxied).
- **Automatic deployments:** Cloudflare Pages deploys on every push to `main` and creates preview builds for branches/PRs.

If you need step-by-step instructions (creating the Cloudflare project, environment variables, checks, or troubleshooting), consult the deployment guide: [CLOUDFLARE_DEPLOYMENT.md](CLOUDFLARE_DEPLOYMENT.md).

## Technologies

- **Hugo**: Static site generator
- **PaperMod**: Fast, minimal Hugo theme
- **Cloudflare Pages**: Hosting and CDN
- **GitHub Actions**: (optional) CI workflows if used locally

## License

Content © 2026 Adrian Giacometti. All rights reserved.
