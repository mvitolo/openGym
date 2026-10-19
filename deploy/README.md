# Deploying openGym on the shared VPS (fork only)

Upstream's published images, behind fm-mono's Caddy, at `https://myog.duckdns.org`.

```
Internet ──443──▶ Caddy (fm-mono) ──edge──▶ opengym-web:80 ──internal──▶ opengym-api:3000
```

Only files under `deploy/` and the two fork workflows differ from upstream, so syncing
upstream never conflicts.

## One-time setup

Assumes the `edge` network, `/opt/caddy-sites` and the `deploy` user exist (familyhub setup).

```bash
sudo -u deploy git clone https://github.com/mvitolo/openGym.git /opt/opengym
cd /opt/opengym
cp deploy/.env.example deploy/.env
docker compose -f deploy/docker-compose.yml up -d
cp deploy/opengym.caddy /opt/caddy-sites/
docker compose -f /opt/fm-mono/deploy/docker-compose.yml \
  exec -T caddy caddy reload --config /etc/caddy/Caddyfile
curl https://myog.duckdns.org/api/health   # -> {"ok":true,...}
```

Then sign up (you become admin via `FIRST_USER_ADMIN=1`), set `INVITE_ONLY=1` in
`deploy/.env`, and `docker compose -f deploy/docker-compose.yml up -d --force-recreate api`.

GitHub secrets on `mvitolo/openGym`, same values as familyhub: `SSH_HOST`, `SSH_USER`,
`SSH_DEPLOY_KEY`. Enable Settings → Actions → "Allow GitHub Actions to create and approve
pull requests".

## How updates work

- **New upstream release:** `upstream-release.yml` opens a "Deploy openGym X.Y.Z" PR daily
  if one exists. Merge it, and `vps-deploy.yml` rolls the stack and checks `/api/health`.
- **Upstream code into the fork:** "Sync fork" on GitHub. It doesn't redeploy (no `deploy/` change).
- **Rollback:** revert the bump PR.

## Backup

`/opt/opengym/data` holds users, passkeys and every workout: `tar czf opengym-$(date +%F).tgz -C /opt/opengym data`.
