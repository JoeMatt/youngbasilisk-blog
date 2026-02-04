# Deployment Instructions

## Step 1: Create GitHub Repository

1. Go to https://github.com/new
2. Repository name: `youngbasilik-blog`
3. Make it **Public** (required for free GitHub Pages)
4. **DO NOT** initialize with README (we already have one)
5. Click "Create repository"

## Step 2: Push to GitHub

```bash
cd /Users/jmattiello/.openclaw/workspace/youngbasilik-blog

# Add GitHub remote
git remote add origin https://github.com/JoeMatt/youngbasilik-blog.git

# Push to GitHub
git push -u origin main
```

## Step 3: Enable GitHub Pages

1. Go to repository → Settings → Pages
2. Source: Select "GitHub Actions"
3. Wait 2-3 minutes for first deployment
4. Site will be live at: https://joematt.github.io/youngbasilik-blog/

## Step 4: Configure Custom Domain

### In GitHub:
1. Repository → Settings → Pages → Custom domain
2. Enter: `youngbasilisk.com`
3. Check "Enforce HTTPS" (after DNS propagates)

### In Namecheap:
1. Domain Dashboard → Advanced DNS
2. Add these A Records:
   ```
   Type: A Record | Host: @ | Value: 185.199.108.153
   Type: A Record | Host: @ | Value: 185.199.109.153
   Type: A Record | Host: @ | Value: 185.199.110.153
   Type: A Record | Host: @ | Value: 185.199.111.153
   ```
3. Add CNAME Record:
   ```
   Type: CNAME | Host: www | Value: joematt.github.io
   ```
4. Save all records

### DNS Propagation:
- Wait 15-60 minutes for DNS to propagate
- Check status: `dig youngbasilisk.com` (should show GitHub IPs)
- Once propagated, site will be live at https://youngbasilisk.com

## Step 5: Verify Deployment

1. Check GitHub Actions: Repository → Actions tab
2. "Deploy Hugo site to Pages" workflow should be green ✓
3. Visit https://youngbasilisk.com (after DNS propagates)
4. Verify:
   - Homepage loads
   - About page works
   - Theme renders correctly
   - Avatar displays

## Troubleshooting

### Site not deploying?
- Check Actions tab for errors
- Verify Pages is set to "GitHub Actions" (not "Deploy from a branch")

### Custom domain not working?
- Wait 30-60 minutes for DNS propagation
- Check DNS records: `dig youngbasilisk.com`
- Verify CNAME file exists in static/ directory

### Theme not loading?
- Ensure submodule was cloned: `git submodule update --init --recursive`
- Check themes/PaperMod directory exists

## Next Steps: Adding Content

To add your first article:

```bash
cd /Users/jmattiello/.openclaw/workspace/youngbasilik-blog

# Create new post
hugo new posts/memory-without-the-memory-tax.md

# Edit the file
# Set draft: false when ready to publish

# Commit and push
git add content/posts/memory-without-the-memory-tax.md
git commit -m "Add first article: Memory Without the Memory Tax"
git push

# GitHub Actions will automatically build and deploy
```

Site will update within 2-3 minutes of pushing to main.

---

**Status:** ✅ Site built locally, ready to push to GitHub
