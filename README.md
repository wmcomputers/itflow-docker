# About this Image
This is the Docker image of [ITFlow](https://github.com/itflow-org/itflow). This image was created by a community member, we don't "officially" support Docker.

Please read the wiki: https://docs.itflow.org

# Usage
## ITFlow Only (no Reverse Proxy) 
1. Copy [docker-compose.yml](https://raw.githubusercontent.com/itflow-org/itflow-docker/main/docker-compose.yml) to a directory.
2. Within docker-compose.yml, adjust the ```environment:``` variables such as ITFLOW_NAME, ITFLOW_URL and ITFLOW_REPO (to your own MSPs fork).
3. Copy the [.env](https://raw.githubusercontent.com/itflow-org/itflow-docker/main/.env) file to the same directory.
> Enter your timezone, root domain and database password within this file. You can avoid this step entirely by adding the information to your docker-compose.yml file directly instead. Or being safe, by using docker secrets.
4. Run ```docker compose up -d```
5. Go to your domain. You should be redirected to setup.php. Enter server information correlated to your set up .env and docker-compose.yml files.
> Defaults:  Username: itflow, Password: $ITFLOW_DB_PASS from .env, Database: itflow, Server: itflow-db 

## Complete [Traefik](https://doc.traefik.io/traefik/getting-started/quick-start/) Solution (Reverse Proxy)
1. Copy the traefik [docker-compose.yml](https://raw.githubusercontent.com/itflow-org/itflow-docker/main/traefik-complete/docker-compose.yml) to a directory.
2. Within docker-compose.yml, adjust the ```environment:``` variables such as ITFLOW_NAME, ITFLOW_URL and ITFLOW_REPO (to your own MSPs fork).
3. Copy the [.env](https://raw.githubusercontent.com/itflow-org/itflow-docker/main/traefik-complete/.env) file to the same directory. 
> Enter your docker path (/srv/docker, ., etc), cloudflare info, timezone, root domain and database password within this file.
4. Create your A records for your host. 
5. Run ```docker compose up -d```
6. Verify you are getting certificates through LetsEncrypt. You will have two public URLs, traefik.$ROOT_DOMAIN and $ITFLOW_URL. 
7. Go to your domain. You should be redirected to setup.php. Enter server information correlated to .env and docker-compose.yml
> Defaults:  Username: itflow, Password: $ITFLOW_DB_PASS from .env, Database: itflow, Server: itflow-db



## Environment Variables
```
ENV TZ Etc/UTC

ENV ITFLOW_NAME ITFlow

ENV ITFLOW_REPO github.com/itflow-org/itflow

ENV ITFLOW_REPO_BRANCH master

ENV ITFLOW_URL demo.itflow.org

ENV ITFLOW_PORT 8080

# apache2 log levels: emerg, alert, crit, error, warn, notice, info, debug
ENV ITFLOW_LOG_LEVEL warn

ENV ITFLOW_DB_HOST itflow-db

ENV ITFLOW_DB_PASS null
```

## Changing ITFLOW_REPO* Environment Variables
Please go about this by deleting your volume location ```./itflow``` 

### In Beta
* I *strongly* recommend putting your solution behind [Authelia](https://www.authelia.com/). If requested, I can supply more information on this topic. 
* This project is still in early beta and is considered a **work in progress**.  Many changes are being performed and may cause breakage upon updates. 
* Currently, we strongly recommend against storing confidential information in ITFlow; ITFlow has not undergone a third-party security assessment.
