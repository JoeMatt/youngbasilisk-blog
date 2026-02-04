# DNS Setup for youngbasilisk.com

## Namecheap DNS Configuration

You're currently on the Advanced DNS page. Here's what to configure:

### Step 1: Remove Default Records

Delete these default records:
- ✅ CNAME Record: www → parkingpage.namecheap.com (already removed or being removed)
- ✅ URL Redirect: @ → http://www.youngbasilisk.com/ (already removed or being removed)

### Step 2: Add GitHub Pages A Records

Click "Add New Record" and add these 4 A Records:

**A Record 1:**
- Type: A Record
- Host: @
- Value: 185.199.108.153
- TTL: Automatic

**A Record 2:**
- Type: A Record
- Host: @
- Value: 185.199.109.153
- TTL: Automatic

**A Record 3:**
- Type: A Record
- Host: @
- Value: 185.199.110.153
- TTL: Automatic

**A Record 4:**
- Type: A Record
- Host: @
- Value: 185.199.111.153
- TTL: Automatic

### Step 3: Add CNAME for www

Click "Add New Record":
- Type: CNAME Record
- Host: www
- Value: joematt.github.io
- TTL: Automatic

### Step 4: Save All Changes

Click "Save All Changes" button at the bottom of the Host Records section.

---

## After DNS Configuration

### Create GitHub Repository

```bash
# Go to: https://github.com/new
# Repository name: youngbasilisk-blog
# Public repository
# Do NOT initialize with README (we already have one)
```

### Push to GitHub

```bash
cd /Users/jmattiello/.openclaw/workspace/youngbasilik-blog

# Add GitHub remote
git remote add origin https://github.com/JoeMatt/youngbasilisk-blog.git

# Push to GitHub
git push -u origin main
```

### Enable GitHub Pages

1. Go to repository → Settings → Pages
2. Source: Select "GitHub Actions"
3. Wait 2-3 minutes for first deployment
4. Go to Settings → Pages → Custom domain
5. Enter: `youngbasilisk.com`
6. Save (DNS verification will happen automatically)
7. After DNS propagates (~15-60min), check "Enforce HTTPS"

### Verify Deployment

1. Check: https://github.com/JoeMatt/youngbasilisk-blog/actions
2. Wait for green checkmark ✓
3. Visit: https://youngbasilisk.com (after DNS propagates)

---

## DNS Propagation Check

```bash
# Check if DNS is live
dig youngbasilisk.com

# Should show GitHub IPs:
# 185.199.108.153
# 185.199.109.153
# 185.199.110.153
# 185.199.111.153
```

---

**Status:** All files configured for youngbasilisk.com (with S). Ready to push!
