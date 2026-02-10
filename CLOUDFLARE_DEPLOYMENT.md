# Deployment Guide: Hugo Blog to Cloudflare Pages

This guide covers deploying your Hugo blog to Cloudflare Pages with your custom domain `adriangiacometti.net`.

## Why Cloudflare Pages?

- ✅ **Integrated DNS**: Domain and hosting in the same platform
- ✅ **Free SSL/TLS**: Automatic HTTPS certificates
- ✅ **Global CDN**: Fast content delivery worldwide
- ✅ **Automatic deployments**: Deploy on every git push
- ✅ **Unlimited bandwidth**: No traffic limits
- ✅ **Preview deployments**: Test changes before going live

---

## Prerequisites

- [x] Hugo blog ready in local repository
- [x] Content migrated from WordPress
- [ ] GitHub account (for repository)
- [ ] Cloudflare account
- [ ] Domain `adriangiacometti.net` added to Cloudflare

---

## Step 1: Create GitHub Repository

### Option A: Using GitHub Web Interface

1. Go to https://github.com/new
2. Repository name: `blog` (or your preference)
3. Visibility: **Public**
4. **Do NOT** initialize with README, .gitignore, or license
5. Click **Create repository**

### Option B: Using GitHub CLI

```bash
cd /Users/aegiacometti/Documents/projects/blog
gh repo create blog --public --source=. --remote=origin
```

---

## Step 2: Push Code to GitHub

```bash
cd /Users/aegiacometti/Documents/projects/blog

# If you created repo via web interface:
git remote add origin https://github.com/YOUR_USERNAME/blog.git
git branch -M main
git push -u origin main

# If you used GitHub CLI:
git push -u origin main
```

---

## Step 3: Create Cloudflare Pages Project

### 3.1 Connect to Git

1. Log in to [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. Go to **Workers & Pages**
3. Click **Create application**
4. Select **Pages** tab
5. Click **Connect to Git**
6. Authorize Cloudflare to access your GitHub account
7. Select your `blog` repository

### 3.2 Configure Build Settings

| Setting | Value |
|---------|-------|
| **Project name** | `adriangiacometti-blog` (or your preference) |
| **Production branch** | `main` |
| **Framework preset** | Hugo |
| **Build command** | `hugo` |
| **Build output directory** | `public` |
| **Deploy command** | (leave blank) |

> [!NOTE]
> Do NOT set a Deploy command—Cloudflare Pages handles static file deployment automatically. The `wrangler deploy` command is only for Workers, not Pages.

### 3.3 Environment Variables

Add the following environment variable:

| Variable | Value |
|----------|-------|
| `HUGO_VERSION` | `0.155.3` |

> [!IMPORTANT]
> Make sure to use the same Hugo version you're using locally to avoid build issues.

### 3.4 Deploy

1. Click **Save and Deploy**
2. Wait for the first build to complete (1-2 minutes)
3. Your site will be available at: `https://adriangiacometti-blog.pages.dev`

---

## Step 4: Configure Custom Domain

### 4.1 Add Custom Domain

1. In your Cloudflare Pages project, go to **Custom domains**
2. Click **Set up a custom domain**
3. Enter: `adriangiacometti.net`
4. Click **Continue**

### 4.2 DNS Configuration

Cloudflare will automatically configure the DNS records:

**For apex domain (adriangiacometti.net):**
- Type: `CNAME`
- Name: `@`
- Target: `adriangiacometti-blog.pages.dev`
- Proxy status: **Proxied** (orange cloud)

**For www subdomain:**
- Type: `CNAME`
- Name: `www`
- Target: `adriangiacometti-blog.pages.dev`
- Proxy status: **Proxied** (orange cloud)

> [!NOTE]
> Unlike GitHub Pages, Cloudflare Pages works perfectly with the proxy enabled (orange cloud).

### 4.3 Activate Custom Domain

1. Click **Activate domain**
2. Wait for SSL certificate provisioning (usually 1-2 minutes)
3. Your site will be available at: `https://adriangiacometti.net`

---

## Step 5: SSL/TLS Configuration

### 5.1 SSL/TLS Settings

1. Go to **SSL/TLS** → **Overview**
2. Set encryption mode to: **Full (strict)**
3. Enable:
   - ✅ **Always Use HTTPS**
   - ✅ **Automatic HTTPS Rewrites**
   - ✅ **Minimum TLS Version**: TLS 1.2

### 5.2 Edge Certificates

1. Go to **SSL/TLS** → **Edge Certificates**
2. Verify that **Universal SSL** is active
3. Enable:
   - ✅ **Always Use HTTPS**
   - ✅ **HTTP Strict Transport Security (HSTS)** (optional but recommended)

---

## Step 6: Verification

### 6.1 Check Deployment Status

1. Go to **Workers & Pages** → Your project
2. Check **Deployments** tab
3. Verify latest deployment shows "Success"

### 6.2 Test Your Site

```bash
# Check DNS resolution
dig adriangiacometti.net +short
# Should show Cloudflare IPs

# Test HTTPS
curl -I https://adriangiacometti.net
# Should return 200 OK with HTTPS

# Test www redirect
curl -I https://www.adriangiacometti.net
# Should redirect to https://adriangiacometti.net
```

### 6.3 Browser Verification

Visit your site and verify:
- ✅ Site loads at `https://adriangiacometti.net`
- ✅ HTTPS certificate is valid (green padlock)
- ✅ All posts are visible
- ✅ Images load correctly
- ✅ Navigation works
- ✅ Tags page displays correctly

---

## Automatic Deployments

Every time you push to the `main` branch, Cloudflare Pages will automatically:

1. Pull the latest code
2. Build your Hugo site
3. Deploy to production
4. Invalidate CDN cache

**Workflow:**
```bash
# Make changes locally
hugo new posts/my-new-post.md
# Edit your post...

# Commit and push
git add .
git commit -m "Add new post: My New Post"
git push

# Cloudflare Pages automatically deploys in 1-2 minutes
```

---

## Preview Deployments

Cloudflare Pages creates preview deployments for:
- Pull requests
- Non-production branches

Each preview gets a unique URL like:
`https://abc123.adriangiacometti-blog.pages.dev`

---

## Troubleshooting

### Build Fails

**Check Hugo version:**
- Ensure `HUGO_VERSION` environment variable matches your local version
- Current version: `0.155.3`

**Check build logs:**
1. Go to **Deployments** tab
2. Click on the failed deployment
3. Review build logs for errors

### Site Not Loading

**Check DNS:**
```bash
dig adriangiacometti.net +short
```
Should return Cloudflare IPs.

**Check SSL:**
- Verify SSL/TLS mode is **Full (strict)**
- Check that Universal SSL certificate is active

### Images Not Loading

**Check image paths:**
- Images should be in `static/images/`
- Paths in posts should be `/images/filename.jpg`

**Check build output:**
- Verify `public` directory contains images after build

---

## Performance Optimization

### Enable Additional Features

1. **Speed** → **Optimization**
   - ✅ Auto Minify: HTML, CSS, JS
   - ✅ Brotli compression

2. **Caching** → **Configuration**
   - Browser Cache TTL: Respect Existing Headers
   - Edge Cache TTL: 2 hours (for static content)

3. **Speed** → **Image Optimization** (optional)
   - ✅ Polish (lossless or lossy)
   - ✅ WebP conversion

---

## Rollback

If you need to rollback to a previous version:

1. Go to **Deployments** tab
2. Find the working deployment
3. Click **⋯** (three dots)
4. Select **Rollback to this deployment**

---

## Cost

Cloudflare Pages is **completely free** for:
- Unlimited sites
- Unlimited requests
- Unlimited bandwidth
- 500 builds per month
- 1 build at a time

---

## Next Steps

After deployment:

1. ✅ Test all functionality
2. ✅ Set up analytics (Cloudflare Web Analytics)
3. ✅ Configure redirects if needed
4. ✅ Update any external links pointing to old WordPress site
5. ✅ Monitor performance in Cloudflare Dashboard

---

## Resources

- [Cloudflare Pages Documentation](https://developers.cloudflare.com/pages/)
- [Hugo on Cloudflare Pages](https://developers.cloudflare.com/pages/framework-guides/deploy-a-hugo-site/)
- [Custom Domains](https://developers.cloudflare.com/pages/platform/custom-domains/)
- [Build Configuration](https://developers.cloudflare.com/pages/platform/build-configuration/)
