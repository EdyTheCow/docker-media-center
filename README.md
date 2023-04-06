<p align="center">
  <img width="500" src="https://raw.githubusercontent.com/BeefBytes/Assets/master/Other/container_illustration/v2/dmc.png">
</p>


# 📚 About
Docker Media Center is a fully automated media set-up using Jellyfin and a bunch of other services to achieve full automation. All of the services listed below are running behind Traefik reverse proxy with proper certificates and basic auth. Most other similar set-up were either lacking some sort of reverse proxy and were rather exposing ports or were poorly documented. This guide aims for the quickest way to set-up a secure and production ready media center set-up.

### Services
- Jellyfin
- Transmission
- Radarr
- Sonarr
- Prowlarr
- Jellyseerr

# 🧰 Getting Started
This guide assumes you have a basic knowledge of linux and Docker / Docker Compose. 

### Requirements
- Domain
- [Docker](https://docs.docker.com/engine/install/ubuntu/)
- Docker Compose (Docker Engine now comes with compose)

# 🏗️ Installation

## Preparations
First we want configure enviroment variables and setup domain records before installing anything. 

<b>Enviroment variables</b><br />
Navigate to `dmc/compose/.env/` and edit these variables.

| Variable | Default | Description |
|---|---|---|
| COMPOSE_PROJECT_NAME | dmc | The prefix for all of containers when started from the compose file |
| DOMAIN | domain.com | The domain that's going to be used to access Jellyfin and rest of the other services |
| DATA_DIR | ../data | The location where all of configs, media and downloads are going to be stored. By default it's stored in the data folder |
| TIMEZONE | Europe/Oslo | Your timezone, you can find all of the valid timezones here: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List |
| ENV_PUID | 1000 | Your user ID, used for file permissions. You can find it by running command: id |
| ENV_PGUID | 1000 | Your group ID, used for file permissions. You can find it by running command: id |                                          |
| SUB_DOMAIN_X | - | You can leave these as is unless you want to use different subdomains to access the services. If you change a subdomain, make sure the subdomains match when setting up domain DNS records. |      


<b>Domain DNS records</b><br />
Setting up subdomains for every service, you are free to use whatever subdomain you want as long as it matches the subdomain specified in `dmc/compose/.env/` file. The example below shows the default values.

If you're using Cloudflare, make sure to enable the proxying by enabling the cloud icon. For full end-to-end encryption, you can also enable "Full" under the SSL/TLS section in the Cloudflare panel.


| Sub domain | Record | Target |
|---|---|---|
| dmc.domain.com | A | Your server IP |
| jellyfin.domain.com | CNAME | dmc.domain.com |
| transmission.domain.com | CNAME | dmc.domain.com |
| jellyseer.domain.com | CNAME | dmc.domain.com |
| radarr.domain.com | CNAME | dmc.domain.com |
| sonarr.domain.com | CNAME | dmc.domain.com |
| prowlarr.domain.com | CNAME | dmc.domain.com |

## Setting up Traefik
<b>Clone repository</b><br />
```
git clone https://github.com/EdyTheCow/docker-media-center.git
```

<b>Set correct acme.json permissions</b><br />

Navigate to `_base/data/traefik/` and run
```
sudo chmod 600 acme.json
```

<b>Basic auth</b><br />
Navigate to `_base/data/traefik/.htpasswd` and paste your generated user/pass in MD5 format. This will be your basic auth user/pass for most services we're going to set up.

<b>Start docker compose</b><br />
Inside of `_base/compose` run
 ```
docker-compose up -d
 ```

## Jellyfin
Inside of `dmc/compose` run
 ```
docker-compose up -d jellyfin
 ```
<b>Configuration</b><br />
Navigate to `jellyfin.domain.com` in your browser and follow the instructions. When selecting library folder follow these paths:

| Library | Path |
|---|---|
| Movies | /data/media/local/movies |
| TV Shows | /data/media/local/tvshows |

Jellyfin only has one volume pointing at `/data/media` this allows us to add whatever media type we want without having to define extra volumes in docker. The reason why it's under `local` directory is in case we wanted to mount google drive or similar in the future, we could add media under `gdrive` instead of `local` for example.

## Transmission
Start service
 ```
docker-compose up -d transmission
 ```
<b>Configuration</b><br />
Navigate to `transmission.domain.com` in your browser, you should be asked to login using the credentials for basic auth you set-up earlier. Make sure everything works, other than that there's no further configuration.

## Prowlarr

## Radarr
Start service
 ```
docker-compose up -d radarr
 ```

 <b>Configuration</b><br />
In the panel, navigate to `Media Management` section and add root folder by simply selecting `/movies` directory.

## Sonarr
Start service
 ```
docker-compose up -d sonarr
 ```

 <b>Configuration</b><br />
In the panel, navigate to `Media Management` section and add root folder by simply selecting `/tv` directory.


## Jellyseerr
Start service
 ```
docker-compose up -d jellyseerr
 ```

 <b>Configuration</b><br />
 Select option to use `Jellyfin account` and proceed by providing url and account details for your jellyfin installation. Scan and enable libraries.

 <b>Adding radarr / sonarr</b><br />
Most importantly, do not use the actual url or IP of radarr / sonarr. Simply use `radarr` or `sonarr` for the hostname. This will resolve the local IP of the container rather than the public one. Since we're running everything on the same server, it's more secure and convenient to connect these services locally. Make sure to uncheck `Use SSL` option too for the local connection to work.

Below are the important settings you should edit, the instructions for sonarr are exactly the same. Just make sure to replace `radarr` with `sonarr` and you should be good to go.

| Setting | Value |
|---|---|
| Default Server | Checked |
| Server Name | Radarr (can be something else too) |
| Hostname or IP Address | radarr |
| Use SSL | Unchecked |
| API Key | Can be found under General section in radarr panel |


# 📋 TODO
- Add an option for rclone for ability to mount gdrive, etc.
- Add Plex as an option
- Add VPN option for Transmission torrent client
- Explore the hype surrounding https://real-debrid.com
