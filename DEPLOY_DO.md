# Deployment Guide — suborbuild.com

## How it works

```
Push to main
    │
    ▼
GitHub Actions: validate (automatic)
    │
    ▼
Waiting for manual approval (production environment)
    │
    ▼
rsync static files to DigitalOcean Droplet
    https://suborbuild.com
```

- **Droplet**: `152.42.192.133` (same droplet as abhijitmohanty.com)
- **Deploy user**: `deploy` (reused from existing setup)
- **Web root**: `/srv/suborbuild/current` (symlink to latest release)

## Approving a production deployment

1. Go to [Actions](https://github.com/mohantyabhijit/suborbuild/actions)
2. Click the latest workflow run
3. `deploy-production` will show "Waiting for review"
4. Click "Review deployments", check "production", click "Approve and deploy"

## Initial server setup (one-time, on Droplet)

### 1) Create site directories

```bash
sudo mkdir -p /srv/suborbuild/releases
sudo chown -R deploy:deploy /srv/suborbuild
```

The `deploy` user already exists from the abhijitmohanty.com setup.

### 2) SSH key for GitHub Actions

The `deploy` user's `authorized_keys` is already configured.
Just reuse the same `DO_SSH_KEY` secret from the other repo (or copy it).

### 3) Nginx config

Create `/etc/nginx/sites-available/suborbuild.conf`:

```nginx
server {
    listen 80;
    server_name suborbuild.com www.suborbuild.com;

    root /srv/suborbuild/current;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;
}
```

Enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/suborbuild.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### 4) HTTPS with certbot

```bash
sudo certbot --nginx -d suborbuild.com -d www.suborbuild.com
```

### 5) GitHub repo secrets

In [repo settings > Secrets > Actions](https://github.com/mohantyabhijit/suborbuild/settings/secrets/actions), add:

| Secret | Value |
|--------|-------|
| `DO_HOST` | `152.42.192.133` |
| `DO_USER` | `deploy` |
| `DO_PORT` | `22` |
| `DO_SSH_KEY` | Contents of `~/.ssh/do_deploy_key` (same private key as abhijitmohanty.com) |

### 6) GitHub environment: production

In [repo settings > Environments](https://github.com/mohantyabhijit/suborbuild/settings/environments):
- Create environment named `production`
- Enable "Required reviewers" and add yourself

## Rollback (instant)

```bash
cd /srv/suborbuild/releases && ls -1dt */
sudo ln -sfn /srv/suborbuild/releases/<previous_release> /srv/suborbuild/current
# Nginx follows the symlink automatically — no reload needed
```
