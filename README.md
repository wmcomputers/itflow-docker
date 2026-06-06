# About this Image
This is a community Docker image of [ITFlow](https://github.com/itflow-org/itflow), forked and maintained by [WM Computers](https://github.com/wmcomputers/itflow-docker). ITFlow is not officially supported via Docker.

Please read the wiki: https://docs.itflow.org

## What's different in this fork
This fork extends the upstream image with:
- **php-zip** and **php-xml** PHP extensions (required by ITFlow but missing from the upstream image)
- **Integrated cron** — all ITFlow scheduled tasks run inside the main container, no separate cron container needed
- Cron jobs run as `www-data` with correct log file ownership under `/var/log/itflow_cron_*.log`

## Included Cron Jobs
| Script | Schedule | Purpose |
|--------|----------|---------|
| `cron/mail_queue.php` | Every minute | Sends queued outbound emails |
| `cron/ticket_email_parser.php` | Every minute | Parses inbound email into tickets (requires IMAP config) |
| `cron/domain_refresher.php` | Daily midnight | Refreshes domain expiry data |
| `cron/certificate_refresher.php` | Daily 1am | Refreshes SSL certificate data |
| `cron/cron.php` | Daily 2am | General scheduled tasks |

---

# Usage
## ITFlow Only (no Reverse Proxy)
1. Copy [docker-compose.yml](https://raw.githubusercontent.com/wmcomputers/itflow-docker/main/docker-compose.yml) to a directory.
2. Within docker-compose.yml, adjust the `environment:` variables such as `ITFLOW_NAME`, `ITFLOW_URL` and `ITFLOW_REPO` (to your own MSPs fork).
3. Copy the [.env](https://raw.githubusercontent.com/wmcomputers/itflow-docker/main/.env) file to the same directory.
> Enter your timezone, root domain and database password within this file. You can avoid this step entirely by adding the information to your docker-compose.yml file directly instead. Or being safe, by using docker secrets.
4. Run `docker compose up -d`
5. Go to your domain. You should be redirected to setup.php. Enter server information correlated to your set up .env and docker-compose.yml files.
> Defaults: Username: itflow, Password: $ITFLOW_DB_PASS from .env, Database: itflow, Server: itflow-db

## Complete [Traefik](https://doc.traefik.io/traefik/getting-started/quick-start/) Solution (Reverse Proxy)
1. Copy the traefik [docker-compose.yml](https://raw.githubusercontent.com/wmcomputers/itflow-docker/main/traefik-complete/docker-compose.yml) to a directory.
2. Within docker-compose.yml, adjust the `environment:` variables such as `ITFLOW_NAME`, `ITFLOW_URL` and `ITFLOW_REPO` (to your own MSPs fork).
3. Copy the [.env](https://raw.githubusercontent.com/wmcomputers/itflow-docker/main/traefik-complete/.env) file to the same directory.
> Enter your docker path (/srv/docker, ., etc), cloudflare info, timezone, root domain and database password within this file.
4. Create your A records for your host.
5. Run `docker compose up -d`
6. Verify you are getting certificates through LetsEncrypt. You will have two public URLs, traefik.$ROOT_DOMAIN and $ITFLOW_URL.
7. Go to your domain. You should be redirected to setup.php. Enter server information correlated to .env and docker-compose.yml
> Defaults: Username: itflow, Password: $ITFLOW_DB_PASS from .env, Database: itflow, Server: itflow-db

---

## Environment Variables
```
ENV TZ Etc/UTC

ENV ITFLOW_NAME ITFlow

ENV ITFLOW_REPO github.com/itflow-org/itflow

ENV ITFLOW_REPO_BRANCH master

ENV ITFLOW_URL demo.itflow.org

ENV ITFLOW_PORT 8443

# apache2 log levels: emerg, alert, crit, error, warn, notice, info, debug
ENV ITFLOW_LOG_LEVEL warn

ENV ITFLOW_DB_HOST itflow-db

ENV ITFLOW_DB_PASS null

ENV ITFLOW_CRON_KEY changeme
```

---

## Microsoft 365 Email Integration (SMTP & IMAP)
This image supports Microsoft 365 OAuth2 for both outbound SMTP and inbound IMAP (email-to-ticket parsing).

### Setup
1. Create an **App Registration** in [Azure Entra ID](https://portal.azure.com):
   - Add a **Redirect URI** (Web): `https://YOUR_ITFLOW_URL/admin/oauth_microsoft_mail_callback.php`
   - Add **Delegated API permissions**: `IMAP.AccessAsUser.All`, `SMTP.Send`, `offline_access` (Microsoft Graph)
   - Grant admin consent for all permissions
   - Note your **Client ID**, **Tenant ID**, and create a **Client Secret**

2. In ITFlow go to **Admin → Settings → Mail**:
   - Set SMTP Provider and IMAP Provider to **Microsoft 365 (OAuth)**
   - Enter Client ID, Client Secret, Tenant ID
   - Set IMAP Username to the mailbox address (e.g. `support@yourdomain.com`)
   - Click **Connect Microsoft 365** and sign in with a **licensed user** that has Full Access to the mailbox

3. Go to **Admin → Settings → Ticketing** and enable **Email-to-ticket parsing**

### Shared Mailbox Note
When using a **shared mailbox** for IMAP, the OAuth token is issued to the licensed user who completes the OAuth flow (not the shared mailbox itself). For SMTP, the `config_smtp_username` in the database must be set to that licensed user's UPN (e.g. `william@yourdomain.com`), not the shared mailbox address. The licensed user will also need **Send As** permission on the shared mailbox in Exchange Online.

---

## Changing ITFLOW_REPO* Environment Variables
Please go about this by deleting your volume location `./itflow`

---

### In Beta
* This project is still in early beta and is considered a **work in progress**. Many changes are being performed and may cause breakage upon updates.
* Currently, we strongly recommend against storing confidential information in ITFlow; ITFlow has not undergone a third-party security assessment.
* I *strongly* recommend putting your solution behind [Authelia](https://www.authelia.com/). If requested, I can supply more information on this topic.
