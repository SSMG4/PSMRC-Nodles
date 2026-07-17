# Deployment: Self-hosted

The app now runs as a systemd service on
your own server, bound to `127.0.0.1` only, exposed to the internet through
Cloudflare (Tunnel is assumed here since it's already in use for other
projects on this box - see the note at the end if you'd rather use a plain
Nginx reverse proxy + DNS record instead).

Do these steps in order.

## 1. Prerequisites on the server

```bash
node --version      # want v24.x - if it's not, see the nvm note below
which node          # note the full path, you'll need it in step 4 (path cannot be in /home otherwise permission will be denied)
convert -version     # confirms ImageMagick's CLI tools are present
```

If `convert`/`identify` aren't found:
```bash
sudo apt update && sudo apt install imagemagick
```
Debian 13 ships ImageMagick 7. It still provides `convert`/`identify` as
compatibility commands (which is all `gm` needs), but if you see conversion
errors after deploying, this is the first thing to check - try running
`convert -version` and doing a manual `convert in.png out.png` on a test
file.

If Node isn't v24, and it was installed via `nvm`:
```bash
nvm install 24
nvm alias default 24
```
Note the resulting path (`nvm which 24` or `which node` after `nvm use 24`) -
systemd doesn't source your shell's nvm setup, so the unit file needs the
literal path, e.g. `/home/youruser/.nvm/versions/node/v24.x.x/bin/node`.

## 2. Get the project onto the server

From your machine (PuTTY includes `pscp`/`psftp` for this), copy the project
up, e.g. with `pscp`:
```
pscp -r psmrc-updated youruser@yourserver:/tmp/psmrc-nodles
```
Or `git clone`/pull if you've pushed this to your own repo instead.

Then on the server:
```bash
sudo mkdir -p /opt/psmrc-nodles
sudo mv /tmp/psmrc-nodles/* /opt/psmrc-nodles/
sudo mv /tmp/psmrc-nodles/.gitignore /tmp/psmrc-nodles/.nvmrc /opt/psmrc-nodles/ 2>/dev/null
cd /opt/psmrc-nodles
npm install --omit=dev
```

## 3. Create a dedicated system user

Matches the pattern used for the other apps on this box - don't run it as
root or your login user.

```bash
sudo useradd --system --shell /usr/sbin/nologin --no-create-home psmrc
sudo chown -R psmrc:psmrc /opt/psmrc-nodles
```

## 4. Install the systemd service

```bash
sudo cp /opt/psmrc-nodles/psmrc.service /etc/systemd/system/psmrc.service
sudo nano /etc/systemd/system/psmrc.service
```

Edit two things before saving:
- `ExecStart=` - replace with the real `node` path from step 1.
- `Environment=PORT=3001` - pick a free port. Check first:
  ```bash
  ss -tlnp | grep 3001
  ```
  If that's taken (or already used by another project on this box), pick a
  different one and use the same number in step 5.

Then:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now psmrc
sudo systemctl status psmrc
```
`status` should show `active (running)`. Check the logs:
```bash
journalctl -u psmrc -f
```
and confirm it responds locally:
```bash
curl -I http://127.0.0.1:3001/
```

## 5. Wire up the domain

**Assumption**: since Cloudflare Tunnel is already used on this server, this
adds the new hostname to that same tunnel rather than opening ports. If
you'd rather use a plain Nginx reverse proxy + a normal DNS record instead,
skip to the alternative at the bottom.

Using Tunnel:
Go to your Cloudflare Dashboard: https://dash.cloudflare.com/

On your domain settings, go to Networking -> Tunnels, then pick your Tunnel (or make one if none).

Click on Add route -> Published application

Fill out the subdomain field (or not), and the Service URL field.

Click Add route and enjoy!

Route the DNS record (uses the Cloudflare account `cloudflared` is already
authenticated with):
```bash
cloudflared tunnel list                                   # confirm the tunnel name/ID
cloudflared tunnel route dns <tunnel-name-or-id> psmrc.ssmg4.dpdns.org
```

Restart the tunnel to pick up the new ingress rule:
```bash
sudo systemctl restart cloudflared     # host service, or:
docker restart <cloudflared-container-name>   # if it's containerized
```

If cloudflared is running in Docker as part of a docker-compose stack for
another project, the cleanest option is usually to add this hostname to that
same stack's config.yml rather than starting a second cloudflared instance.

## 6. Verify

```bash
curl -I https://psmrc.ssmg4.dpdns.org/
```
Then do a real upload through the site in a browser to confirm the full
conversion pipeline works end to end.

## Notes

- **No open ports needed** for the Tunnel approach - the app only listens on
  `127.0.0.1`, and cloudflared makes an outbound-only connection. Nothing to
  forward on your router.
- **HTTPS is handled by Cloudflare** at the edge; the connection from
  cloudflared to the app is local, so no certificates to manage on this app.
- **Cleanup only runs reactively** - `cleanByExpiration()` fires on each new
  upload, not on a timer. If the site goes quiet for a while, stale
  `uploads/`/`public/pack/` files just sit there until the next upload. Not
  a bug, just worth knowing - a systemd timer calling the same cleanup could
  be added later if that ever matters.

### Alternative: Nginx reverse proxy + plain DNS instead of Tunnel

If you'd rather not add this to the existing tunnel:
1. Point a proxied (orange-cloud) `A`/`AAAA` record for
   `psmrc.ssmg4.dpdns.org` at the server's public IP in the Cloudflare
   dashboard, and forward ports 80/443 to this server if not already done.
2. Add an Nginx server block:
   ```nginx
   server {
       listen 80;
       server_name psmrc.ssmg4.dpdns.org;
       location / {
           proxy_pass http://127.0.0.1:3001;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```
3. Set Cloudflare's SSL/TLS mode to "Full" (edge-to-server stays HTTP here,
   which is fine since Cloudflare proxies the connection - use "Full
   (strict)" instead if you also put a cert on the origin).
4. `sudo nginx -t && sudo systemctl reload nginx`.
