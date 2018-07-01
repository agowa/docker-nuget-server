# docker-nuget-server

[![Docker Build Status](https://img.shields.io/docker/build/agowa338/docker-nuget-server.svg)](https://hub.docker.com/r/agowa338/docker-nuget-server/)
[![Docker Pulls](https://img.shields.io/docker/pulls/agowa338/docker-nuget-server.svg)](https://hub.docker.com/r/agowa338/docker-nuget-server/)
[![Docker Automated build](https://img.shields.io/docker/automated/agowa338/docker-nuget-server.svg)](https://hub.docker.com/r/agowa338/docker-nuget-server/)
[![ImageLayers Size](https://img.shields.io/imagelayers/image-size/agowa338/docker-nuget-server/latest.svg)](https://hub.docker.com/r/agowa338/docker-nuget-server/)
[![ImageLayers Layers](https://img.shields.io/imagelayers/layers/agowa338/docker-nuget-server/latest.svg)](https://hub.docker.com/r/agowa338/docker-nuget-server/)



Auto build docker [image](https://hub.docker.com/r/agowa338/docker-nuget-server/) for [simple-nuget-server](https://github.com/agowa338/simple-nuget-server)

## docker command

### https

``` shell
api_key = `openssl rand -base64 32`

docker run -d --name nuget-server -p 443:443 -e HTTPS_ENABLE=true -e NUGET_API_KEY=$api_key agowa338/docker-nuget-server
```

### http

``` shell
api_key = `openssl rand -base64 32`
docker run -d --name nuget-server -p 80:80 -e NUGET_API_KEY=$api_key agowa338/docker-nuget-server
```

### docker-compose

Make sure your Host feed is available on either port `80` or `443`.
If you use https, port `80` redirects to `443` with tls enabled.

#### https

``` yaml
version: '2'
services:
  nuget-server:
    container_name: nuget-server
    image: agowa338/docker-nuget-server:latest
    ports:
      - "443:443"
      - "80:80"
    restart: always
    environment:
      NUGET_API_KEY: "2bYS9RO2DYBc4CbPRuE3AoTlDVoiLM24pbkhPk1D83w="
      UPLOAD_MAX_FILESIZE: "40M"
#      SERVER_NAME: "nuget.example.com" # Also ip addresses can be used.
      WORKER_PROCESSES: "4"
      WORKER_CONNECTIONS: "65535"
    volumes:
      - nuget-db:/var/www/simple-nuget-server/db:rw
      - nuget-packagefiles:/var/www/simple-nuget-server/packagefiles:rw
      - nuget-tls:/etc/nginx/tls:ro
    ulimits:
      nproc: 65535
      nofile:
        soft: 20000
        hard: 40000
volumes:
  nuget-db:/srv/nuget/db
  nuget-packagefiles:/srv/nuget/packagefiles
  nuget-tls:/srv/nuget/tls
```

#### http

``` yaml
version: '2'
services:
  nuget-server:
    container_name: nuget-server
    image: agowa338/docker-nuget-server:latest
    ports:
      - "80:80"
    restart: always
    environment:
      NUGET_API_KEY: "2bYS9RO2DYBc4CbPRuE3AoTlDVoiLM24pbkhPk1D83w="
      UPLOAD_MAX_FILESIZE: "40M"
#      SERVER_NAME: "nuget.example.com" # Also ip addresses can be used.
      WORKER_PROCESSES: "4"
      WORKER_CONNECTIONS: "65535"
    volumes:
      - nuget-db:/var/www/simple-nuget-server/db:rw
      - nuget-packagefiles:/var/www/simple-nuget-server/packagefiles:rw
      - nuget-tls:/etc/nginx/tls:ro
    ulimits:
      nproc: 65535
      nofile:
        soft: 20000
        hard: 40000
volumes:
  nuget-db:/srv/nuget/db
  nuget-packagefiles:/srv/nuget/packagefiles
  nuget-tls:
```

#### Environment configuration

* `NUGET_API_KEY`: set nuget api key. By default the key is generated randomly

* `UPLOAD_MAX_FILESIZE`: set the maximum size of an uploaded nuget package file. Default size: `20M`

* `WORKER_PROCESSES`: set nginx worker processes.Default: `1`

* `WORKER_CONNECTIONS`: set nginx worker connections. Default: `65535`

* `SERVER_NAME`: set server domain name,must set value with your server name or ip. Default: `localhost`

* `HTTPS_ENABLE`: If set to any value (even if set to false!), nginx will be started by using the tls configuration. It cannot be changed after the first launch, you'll have to start a new instance.

  **Note:** If use `host` network mode,you can set `SERVER_PORT` value  to change nuget server port.

#### Volumes
* `/var/www/simple-nuget-server/db` Path with SQLite database.
* `/var/www/simple-nuget-server/packagefiles` Path with nuget packages save.
* `/etc/nginx/tls` Path with nginx tls configuration. If you want to use https, please mount this path, generate cert/key and dhparameter to support https.


## Test

For testing, you can either use
* the [nuget commandline tool](https://www.nuget.org/downloads) or
* [PowerShell](https://www.github.com/PowerShell/PowershellCore)

### Push nuget package:

``` shell
nuget push xxx.nupkg -source SERVER_NAME -apikey NUGET_API_KEY
```

### Download nuget package:

``` shell
nuget install xxx -source SERVER_NAME -packagesavemode nupkg
```

### Initialize for PowerShell Packages

``` powershell
$api_key = "NUGET_API_KEY"
Register-PSRepository -Name MyRepository -SourceLocation https://nuget.example.com/ -PublishLocation https://nuget.example.com/ -InstallationPolicy Trusted
Publish-Module -NuGetApiKey $api_key -Name PackageManagement -Repository MyRepository
Publish-Module -NuGetApiKey $api_key -Name PowerShellGet -Repository MyRepository
```

