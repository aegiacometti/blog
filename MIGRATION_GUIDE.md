# Guía de Migración de WordPress a Hugo

Esta guía detalla el proceso completo para migrar tu blog de WordPress en Hostinger a Hugo en GitHub Pages.

## Estado Actual

✅ **Completado**:
- Hugo instalado (v0.155.3)
- Proyecto inicializado con tema PaperMod
- Configuración básica del sitio
- GitHub Actions configurado
- Servidor de desarrollo funcionando

⏳ **Pendiente**:
- Exportar contenido de WordPress
- Migrar posts y páginas
- Configurar GitHub repository
- Configurar DNS en Cloudflare

---

## Paso 1: Exportar Contenido de WordPress

### Opción A: Exportación Manual (Recomendada)

1. **Acceder al Admin de WordPress**:
   - Ir a `https://adriangiacometti.net/wp-admin`
   - Iniciar sesión con tus credenciales

2. **Exportar Contenido**:
   - Navegar a **Herramientas → Exportar**
   - Seleccionar **"Todo el contenido"**
   - Click en **"Descargar archivo de exportación"**
   - Guardar el archivo XML

3. **Descargar Imágenes**:
   - Navegar a **Medios → Biblioteca**
   - Usar plugin "Export Media Library" o descargar manualmente
   - Alternativamente, usar FTP para descargar `/wp-content/uploads/`

### Opción B: Usando WP-CLI (Si tienes acceso SSH)

```bash
# Conectar por SSH a Hostinger
ssh usuario@tuservidor.hostinger.com

# Exportar contenido
wp export --dir=/tmp/

# Descargar archivo
scp usuario@tuservidor.hostinger.com:/tmp/export.xml ./wordpress-export.xml
```

---

## Paso 2: Convertir Contenido a Hugo

### Usando wordpress-to-hugo-exporter

```bash
# Instalar la herramienta
npm install -g wordpress-export-to-markdown

# Convertir el XML a Markdown
npx wordpress-export-to-markdown \
  --input=wordpress-export.xml \
  --output=content/posts \
  --post-folders=false \
  --save-attached-images=true \
  --save-scraped-images=true

# Mover imágenes a la carpeta correcta
mv content/posts/images/* static/images/
rmdir content/posts/images
```

### Ajustes Manuales Necesarios

Después de la conversión, revisar cada post para:

1. **Verificar Frontmatter**:
   ```yaml
   ---
   title: "Título del Post"
   date: 2024-01-24T10:00:00+01:00
   draft: false
   tags: ["netdevops", "automation", "python"]
   categories: ["DevOps"]
   ---
   ```

2. **Actualizar URLs de Imágenes**:
   - Cambiar rutas de WordPress a rutas de Hugo
   - Ejemplo: `/wp-content/uploads/2024/01/imagen.png` → `/images/imagen.png`

3. **Revisar Formato**:
   - Verificar que el Markdown se convirtió correctamente
   - Ajustar bloques de código si es necesario
   - Verificar enlaces internos

---

## Paso 3: Configurar Repositorio en GitHub

### Crear Repositorio

1. Ir a [github.com/new](https://github.com/new)
2. Nombre del repositorio: `blog` (o el que prefieras)
3. Visibilidad: **Public** (necesario para GitHub Pages gratuito)
4. **NO** inicializar con README (ya lo tenemos)
5. Click en **"Create repository"**

### Subir Código

```bash
# En tu directorio local del blog
cd /Users/aegiacometti/Documents/projects/blog

# Añadir archivos al staging
git add .

# Hacer commit inicial
git commit -m "Initial commit: Hugo blog setup with PaperMod theme"

# Cambiar rama a main (si es necesario)
git branch -M main

# Añadir remote de GitHub
git remote add origin https://github.com/[tu-usuario]/blog.git

# Push al repositorio
git push -u origin main
```

---

## Paso 4: Configurar GitHub Pages

1. **Ir a Settings del Repositorio**:
   - `https://github.com/[tu-usuario]/blog/settings/pages`

2. **Configurar Source**:
   - En "Build and deployment"
   - Source: **GitHub Actions**
   - Guardar cambios

3. **Verificar Deployment**:
   - Ir a la pestaña **Actions**
   - Verificar que el workflow "Deploy Hugo site to GitHub Pages" se ejecutó correctamente
   - El sitio estará disponible en `https://[tu-usuario].github.io/blog`

4. **Configurar Dominio Personalizado** (en GitHub):
   - En Settings → Pages
   - Custom domain: `adriangiacometti.net`
   - Click en **Save**
   - Marcar **"Enforce HTTPS"** (después de configurar DNS)

---

## Paso 5: Configurar DNS en Cloudflare

### Transferir o Configurar Dominio

Si aún no has transferido el dominio a Cloudflare:

1. **Añadir Sitio en Cloudflare**:
   - Ir a [dash.cloudflare.com](https://dash.cloudflare.com)
   - Click en **"Add a Site"**
   - Ingresar `adriangiacometti.net`
   - Seleccionar plan Free
   - Cloudflare escaneará tus DNS actuales

2. **Cambiar Nameservers en Hostinger**:
   - Ir al panel de Hostinger
   - Sección de Dominios
   - Cambiar nameservers a los proporcionados por Cloudflare
   - Ejemplo: `ns1.cloudflare.com` y `ns2.cloudflare.com`

### Configurar Registros DNS

En el panel de Cloudflare, configurar los siguientes registros:

#### Registros A (para apex domain)

| Type | Name | Content | Proxy Status |
|------|------|---------|--------------|
| A | @ | 185.199.108.153 | DNS only (🌥️ gris) |
| A | @ | 185.199.109.153 | DNS only (🌥️ gris) |
| A | @ | 185.199.110.153 | DNS only (🌥️ gris) |
| A | @ | 185.199.111.153 | DNS only (🌥️ gris) |

#### Registro CNAME (para www)

| Type | Name | Content | Proxy Status |
|------|------|---------|--------------|
| CNAME | www | [tu-usuario].github.io | DNS only (🌥️ gris) |

> [!IMPORTANT]
> **Proxy Status debe ser "DNS only"** (nube gris, no naranja) para evitar conflictos con el SSL de GitHub Pages.

### Configurar SSL/TLS en Cloudflare

1. **Ir a SSL/TLS → Overview**:
   - Modo de encriptación: **Full** (o Full strict si GitHub tiene certificado válido)

2. **Configurar Always Use HTTPS**:
   - SSL/TLS → Edge Certificates
   - Activar **"Always Use HTTPS"**
   - Activar **"Automatic HTTPS Rewrites"**

3. **Configurar HSTS** (Opcional pero recomendado):
   - Activar **"HTTP Strict Transport Security (HSTS)"**
   - Max Age: 6 meses
   - Apply to subdomains: Sí
   - Preload: Sí

---

## Paso 6: Verificación Final

### Checklist de Verificación

- [ ] Sitio accesible en `https://adriangiacometti.net`
- [ ] Redirección de `www.adriangiacometti.net` funciona
- [ ] HTTPS activo (candado verde en navegador)
- [ ] Todos los posts migrados están visibles
- [ ] Imágenes se cargan correctamente
- [ ] Navegación funciona (menú, tags, categorías)
- [ ] RSS feed disponible en `/index.xml`
- [ ] Sitemap disponible en `/sitemap.xml`
- [ ] About Me page muestra información correcta

### Comandos de Verificación

```bash
# Verificar DNS
dig adriangiacometti.net
dig www.adriangiacometti.net

# Verificar SSL
curl -I https://adriangiacometti.net

# Verificar sitemap
curl https://adriangiacometti.net/sitemap.xml

# Verificar RSS
curl https://adriangiacometti.net/index.xml
```

---

## Mantenimiento Post-Migración

### Crear Nuevo Post

```bash
# Crear nuevo post
hugo new content/posts/mi-nuevo-post.md

# Editar el archivo
# content/posts/mi-nuevo-post.md

# Previsualizar localmente
hugo server -D

# Cuando esté listo, cambiar draft a false
# Commit y push
git add content/posts/mi-nuevo-post.md
git commit -m "Add new post: Mi Nuevo Post"
git push

# GitHub Actions desplegará automáticamente
```

### Actualizar Tema

```bash
# Actualizar submódulo del tema
git submodule update --remote --merge

# Commit cambios
git add themes/PaperMod
git commit -m "Update PaperMod theme"
git push
```

---

## Troubleshooting

### El sitio no se despliega

1. Verificar que GitHub Actions se ejecutó sin errores
2. Verificar que GitHub Pages está habilitado
3. Verificar que el CNAME está en `static/CNAME`

### Imágenes no se cargan

1. Verificar que las imágenes están en `static/images/`
2. Verificar rutas en los posts (deben ser `/images/nombre.png`)
3. Verificar que las imágenes se commitearon al repositorio

### DNS no resuelve

1. Verificar que los nameservers de Cloudflare están activos
2. Esperar hasta 24 horas para propagación completa
3. Usar `dig` o `nslookup` para verificar

### SSL no funciona

1. Verificar que "Enforce HTTPS" está activado en GitHub Pages
2. Verificar modo SSL en Cloudflare (debe ser "Full")
3. Esperar unos minutos para que GitHub genere el certificado

---

## Recursos Adicionales

- [Documentación de Hugo](https://gohugo.io/documentation/)
- [PaperMod Theme Docs](https://github.com/adityatelange/hugo-PaperMod/wiki)
- [GitHub Pages Docs](https://docs.github.com/en/pages)
- [Cloudflare DNS Docs](https://developers.cloudflare.com/dns/)

---

**¡Migración completada!** 🎉

Tu blog ahora es un sitio estático rápido, seguro y fácil de mantener.
