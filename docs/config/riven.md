# Riven

Guide for setting fully automated media server using Riven, a replacement for Arr services.
Setup is secured using Traefik reverse proxy by generating certificates and providing basic auth for any exposed web UIs.

**Services:** Jellyfin, Jellyseerr, Riven, Zurg, Rclone

## Requirements
- Domain
- Docker / Docker Compose
- [Real-debrid.com](http://real-debrid.com/?id=13771120) subscription (referral link :heart:)

## Preparations
Configuration of environment variables, domain records and basic auth before moving to installation and starting services.

### Clone repository
``` sh
git clone https://github.com/EdyTheCow/docker-media-center.git
``` 

### **Riven** environment variables
Navigate to `dmc-riven/riven/compose/.env` and edit variables below.

**Required variables**

| Variable                      | Default    | Description                                             |
|-------------------------------|------------|---------------------------------------------------------|
| DOMAIN                        | domain.com | Domain used to access Jellyfin and other services       |
| DMC_RIVEN_DB_USER_PASS        |      -     | Generate a password for Riven's database user           |
| DMC_RIVEN_REAL_DEBRID_API_KEY |      -     | Your real-debrid API key, can be found on their website | 

**Optional variables**, these variables can be left as is or changed to your liking.

| Variable             | Default   | Description                                                                                                                      |
|----------------------|-----------|----------------------------------------------------------------------------------------------------------------------------------|
| COMPOSE_PROJECT_NAME | dmc-riven | Prefix for all containers running in docker-compose file                                                                         |
| CONFIG_DIR           |  ../data  | Directory for storing configs of all services                                                                                    |
| TIMEZONE             |  Etc/UTC  | Timezone used by services, valid options can be found [here ](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List) |
| PUID / PGID          | 1000      | User and group ID, you can check yours with command: id                                                                          |
| SUB_DOMAIN_X         | -         | Subdomains for services, leave it or change it to your liking                                                                    |

### **Debrid** environment variables
Navigate to `dmc-riven/debrid/compose/.env`, this file includes mostly the same variables as above which are all optional.

For service Zurg we'll have to manually set real-debrid API key direcly in `config.yml`. Navigate to `dmc-riven/debrid/data/zurg/config.yml` and paste your real-debrid API key in `token:`
``` yaml title="snippet of config.yml"
zurg: v1
token: your-api-key-here
```

### DNS records
You are free to use whatever subdomains you want as long as it matches the subdomain specified in `dmc-riven/riven/compose/.env` file. The example below shows the default values.

| Sub domain        | Record | Target           |
|-------------------|:------:|------------------|
| dmc.domain.com    |    A   | Your server's IP |
| stream.domain.com |  CNAME | dmc.domain.com   |
| add.domain.com    |  CNAME | dmc.domain.com   |
| riven.domain.com  |  CNAME | dmc.domain.com   |

By setting DNS records this way, you can later on change server's IP by changing only one value instead of modifying every subdomains target value.

### Create required directories


## Traefik

### Setting correct permissions for `acme.json`

Navigate to `_base/data/traefik/` and run
``` sh
sudo chmod 600 acme.json
```
This is where Traefik will store all of generated certificates for our services.

### Basic auth

Navigate to `_base/data/traefik/.htpasswd` this is where we'll define basic auth user/pass for all services exposed to internet.
You can generate user / pass pair by using command below. Replace `USER` and `PASSWORD` with your own.
``` sh
printf "USER:$(openssl passwd -apr1 PASSWORD)\n"
```
The command should output something like this
```
myUser:$apr1$qJicUGAO$ci9xH5pr1M02VZAFo.4mo.
```
Copy paste the output to `_base/data/traefik/.htpasswd` file. You can add multiple users by adding more values per new line.

### Start Traefik
Navigate to `_base/compose` and run
``` sh
docker compose up -d
```

## Zurg / Rclone
First make sure you pasted the real-debrid API key in Zurg's config file at `dmc-riven/debrid/data/zurg/config.yml`
By default real-debrid's drive will be mounted on your server at the path `/mnt/zurg`. It's recommended to leave it as is.

### Start Zurg and Rclone
It's important that Zurg and Rclone are started before other services. 
Zurg / Rclone mounts a drive from real-debrid which is used by Jellyfin and Riven, if mount isn't found the rest of services will fail to function.

Navigate to `dmc-riven/debrid/compose` and run
``` sh
docker compose up -d
```

## Jellyfin
Navigate to `dmc-riven/riven/compose` and run

``` sh
docker compose up -d jellyfin
```

### Configuration
Navigate to `https://stream.domain.com` in your browser and follow the installation instructions. When selecting library folder follow these paths:

| Library name | Content type | Path                             |
|--------------|:------------:|----------------------------------|
| Movies       |    Movies    | /mnt/media_symlinks/movies       |
| Shows        |     Shows    | /mnt/media_symlinks/shows        |
| Anime Movies |    Movies    | /mnt/media_symlinks/anime_movies |
| Anime Shows  |     Shows    | /mnt/media_symlinks/anime_shows  |

Library names can be whatever you want as long as paths are correct. `Anime Movies` and `Anime Shows` libraries are also optional.

## Jellyseerr
Navigate to `dmc-riven/debrid/compose` and run

``` sh
docker compose up -d jellyseerr
```

### Configuration
Navigate to `https://add.domain.com` in your browser and follow the installation instructions.



## Riven
Navigate to `dmc-riven/debrid/compose` and run

``` sh
docker compose up -d riven riven-db riven-frontend
```

Riven may take a couple of minutes to fully start. Navigate to `https://riven.domain.com` in your browser. 
This time you will be prompted by basic auth by Traefik. Login with the credentials you chosen earlier in [basic auth](#basic-auth) section.

### Configuration
