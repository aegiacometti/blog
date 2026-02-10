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

```
.
├── content/          # Contenido del sitio
│   ├── posts/       # Posts del blog
│   └── about.md     # Página About Me
├── static/          # Archivos estáticos
│   ├── images/      # Imágenes
│   └── CNAME        # Configuración de dominio
├── themes/          # Temas de Hugo
│   └── PaperMod/    # Tema PaperMod
├── .github/
│   └── workflows/   # GitHub Actions
└── hugo.toml        # Configuración de Hugo
```

## Deployment

El sitio se despliega automáticamente a GitHub Pages cuando se hace push a la rama `main` mediante GitHub Actions.

### Configuración de GitHub Pages

1. Ve a Settings → Pages en tu repositorio
2. En "Build and deployment", selecciona "GitHub Actions" como source
3. El workflow se ejecutará automáticamente en cada push

### Configuración de Dominio Personalizado

El dominio `adriangiacometti.net` está configurado mediante:

1. **Archivo CNAME**: `static/CNAME` contiene el dominio
2. **DNS en Cloudflare**: 
   - 4 registros A apuntando a GitHub Pages IPs
   - Registro CNAME para www
   - Modo "DNS only" (nube gris)
3. **SSL/TLS**: Configurado en Cloudflare en modo "Full"

## Tecnologías

- **Hugo**: Generador de sitios estáticos
- **PaperMod**: Tema minimalista y rápido
- **GitHub Pages**: Hosting
- **GitHub Actions**: CI/CD
- **Cloudflare**: DNS y SSL

## Licencia

Contenido © 2026 Adrian Giacometti. Todos los derechos reservados.
