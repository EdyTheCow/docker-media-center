<p align="center">
  <img width="500" src="https://raw.githubusercontent.com/BeefBytes/Assets/master/Other/container_illustration/v2/dmc.png">
</p>


# 📚 About
Docker Media Center is a fully automated media set-up using Jellyfin and a bunch of other services to achieve full automation. All of the services listed below are running behind Traefik reverse proxy with proper certificates and basic auth. Most other similar setups were either lacking a reverse proxy and exposing ports or were poorly documented. This guide aims for the quickest way to set-up a secure and production ready media center.

Furthermore, the guide is set up with volume paths in a way to allow hardlinks of media files rather than making copies. This means once a media file is downloaded, it is hardlinked to the Jellyfin media directory rather than copied. Such a method is faster and saves storage on your server.

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

<b>Clone repository</b><br />
```
git clone https://github.com/EdyTheCow/docker-media-center.git
```

<b>Enviroment variables</b><br />
Navigate to `dmc/compose/.env/` and edit these variables.

| Variable | Default | Description |
|---|---|---|
| COMPOSE_PROJECT_NAME | dmc | The prefix for all of containers when started from the compose file |
| DOMAIN | domain.com | The domain that's going to be used to access Jellyfin and rest of the other services |
| DATA_DIR | ../data | The location where all of media and downloads are going to be stored |
| SERVICES_DIR | ../data | The location where all of services configs are stored |
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

## Traefik
<b>Set correct acme.json permissions</b><br />

Navigate to `_base/data/traefik/` and run
```
sudo chmod 600 acme.json
```

<b>Basic auth</b><br />
Navigate to `_base/data/traefik/.htpasswd` and paste your generated user/pass in MD5 format. This will be your basic auth user/pass for most services we're going to set up. If unsure, google how to generate htpasswd.

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
| Movies | /data/media/movies |
| TV Shows | /data/media/tvshows |

Jellyfin only has one volume pointing at `/data/media` this allows us to add whatever media type we want without having to define extra volumes in docker for future media types. 

## Transmission
Inside of `dmc/compose` run
 ```
docker-compose up -d transmission
 ```
<b>Configuration</b><br />
Navigate to `transmission.domain.com` in your browser, you should be asked to login using the credentials for basic auth you set-up earlier. Under `Preferences -> Torrents` make sure your download paths look like this: `/data/downloads/complete` and `/data/downloads/incomplete`. This will be important later on for hardlinks.

## Radarr
Inside of `dmc/compose` run

 ```
docker-compose up -d radarr
 ```
<b>Configuration</b><br />
Navigate to `radarr.domain.com` in your browser, in the panel under `Media Management` section and add the root folder by simply selecting `/data/media/movies` directory.

Under `Download Clients` add a new client by selecting Transmission. Change these settings:

| Setting | Value |
|---|---|
| Name | Transmission |
| Host | transmission |
| Category | movies |

Host `transmission` will resolve the local IP of the container, do not use a domain or public IP. It's more convenient and secure to connect services locally. Since the connection is local, you do not need to insert any other credentials. Click `Test` to make sure it works and add the client.

## Sonarr
Inside of `dmc/compose` run
 ```
docker-compose up -d sonarr
 ```

 <b>Configuration</b><br />
Navigate to `sonarr.domain.com` in your browser, in the panel under `Media Management` section and add the root folder by simply selecting `/data/media/tvshows` directory.

Under `Download Clients` add new client by selecting Transmission. Change these settings:

| Setting | Value |
|---|---|
| Name | Transmission |
| Host | transmission |
| Category | tvshows |

Host `transmission` will resolve the local IP of the container, do not use a domain or public IP. It's more convenient and secure to connect services locally. Since the connection is local, you do not need to insert any other credentials. Click `Test` to make sure it works and add the client.

## Prowlarr
Inside of `dmc/compose` run
 ```
docker-compose up -d prowlarr
 ```
<b>Configuration</b><br />
Navigate to `prowlarr.domain.com` in your browser, under `Settings -> Apps` add Sonarr and Radarr. 

| Setting | Value |
|---|---|
| Name | Radarr |
| Sync Level | Full Sync |
| Prowlarr Server | http://prowlarr:9696 |
| Radarr Server | http://radarr:7878 |

Instructions for adding Sonarr are exactly the same, just change the name to `Sonarr` and use `http://sonarr:8989` for `Sonarr Server`. You'll find API keys for both under `Settings -> General` in their respective panels.

Navigate to `Indexers` and click `Add Indexer` to add public or private indexers. Once added, these will automatically sync with Sonarr and Radarr. 

## Jellyseerr
Inside of `dmc/compose` run
 ```
docker-compose up -d jellyseerr
 ```

 <b>Configuration</b><br />
 Navigate to `jellyseerr.domain.com` in your browser, select option to use `Jellyfin account` and proceed by providing url and account details for your jellyfin installation. Scan and enable libraries.

 <b>Adding radarr / sonarr</b><br />
Most importantly, do not use the actual url or IP of radarr / sonarr. Simply use `radarr` or `sonarr` for the hostname. This will resolve the local IP of the container rather than the public one. Since we're running everything on the same server, it's more secure and convenient to connect these services locally. Make sure to uncheck `Use SSL` option too for the local connection to work.

Below are the important settings you should edit, the instructions for sonarr are exactly the same. Just make sure to replace `radarr` with `sonarr` and you should be good to go.

| Setting | Value |
|---|---|
| Default Server | Checked |
| Server Name | Radarr |
| Hostname or IP Address | radarr |
| Use SSL | Unchecked |
| API Key | Can be found under General section in radarr / sonarr panel |


# 📋 TODO
- Add an option for rclone for ability to mount gdrive, etc.
- Add Plex as an option
- Add VPN option for Transmission torrent client
- Explore the hype surrounding https://real-debrid.com
