# About

!!! note

    Just wanna get started and don't care about any of this yapping? Head straight to [Installation](/docker-media-center/installation) section!

This documentation consists of a few different ways to set-up and run fully automated media server. 
All of pre-configured docker compose files have been fully tested, secured and are production ready. 
Anything that needs to be manually configured is documented step by step. 
Sources for all of services and other documentation are linked in the guides.

Use these configurations and guides however you see fit, all credit goes to people who have developed all of these services making this possible. 
Documentation and testing of all of this has taken me many hours. 
If you found any of this useful, I would appreciate if you consider leaving a star :star: :)

## Why use this over X?
When I started looking at existing guides and provided docker compose files, I was surprised at the lack of reverse proxies and/or attempts of securing the services. Most of "ready to use" examples have open ports with no security in mind. Running services that way locally is probably fine, this guide is meant for anyone who wants to run them publicly in a secure way. 

None of services in any of configurations are exposed to internet directly. The few ones that are use Traefik as a reverse proxy to securely expose service's web UI with proper certificates. On top of that, services are put behind basic authentication enforced by Traefik. The rest of the services run and communicate locally on the server. Everything was pre-configured as much as possible for a quick and secure installation.

## Choosing configuration
!!! note

    Not familiar with services that are mentioned? Check out [References ](/docker-media-center/references ) section.
    
The recommended and fastest way of setting up automated media server is to use Riven with a debrid service.
While Riven is still quite new and is under active development, it replaces majority of Arr services compared to a more traditional setup.
Meaning there is way less things to install and maintain. Here's a quick overview:

### Riven
**Services:** Jellyfin, Jellyseerr, Riven, Zurg, Rclone

Riven replaces sonarr, radarr, prowlarr (optional), download client (torrent / debrid client).
However there's a small subscription cost for a debrid service which provides the unlimited shared storage.

### Arr stack
**Services:** Jellyfin, Jellyseerr, Sonarr, Radarr, Prowlarr, qBittorrent, Gluetun

The traditional setup using arr services can provide more granular control. 
Debrid can be also used with arr stack leveraging the unlimited storage.
However, installation and maintenance requires more effort.