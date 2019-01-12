# Local Gateway

_Local Gateway_ provides an all-in-one DNS server with TLS Nginx proxy
configuration packaged with `docker-compose`.

## Table of Contents

- [Abstract](#abstract)
- [Requirements](#requirements)
- [Setup](#setup)
- [Usage](#usage)
    - [Proxy to an application running on the host](#proxy-to-an-application-running-on-the-host)
    - [Proxy to an application running in a docker container](#proxy-to-an-application-running-in-a-docker-container)
    - [Proxy with TLS](#proxy-with-tls)
    - [Register other TLDs](#register-other-tlds)
    - [gRPC](#grpc)
- [Roadmap](#roadmap)
- [Project status](#project-status)
- [Contributing](#contributing)
- [Maintainers](#maintainers)
- [License](#license)

## Abstract

In web application development TLS (Transport Layer Security) communications
are a must.

Indeed, working locally without TLS adds dev specific configurations
which prevents testing some security features (e.g. HSTS,
CSP, secure cookie, etc.) and forbids some interactions with external server (e.g.
SSO, webhook, etc). Configuring each application to do TLS termination is
hard.

Furthermore when you work with multiple applications or with an application
that deals with subdomain having a DNS server is a must too (_prevents you
from editing your local `/etc/hosts` file_).

So the goal of this project is to provide a simple DNS server with TLS proxy
without changing your development application and to reduce the differences
between local environment and production environment.

## Requirements

_Local Gateway_ requires the following software to run on your machine:

- [Docker](https://www.docker.com/) 18.09+
- [Docker Compose](https://docs.docker.com/compose/) 1.23+

## Setup

Ensure you have required `docker` and `docker-compose` version.

```sh
docker --version
# Docker version 18.09.0, build 4d60db4

docker-compose --version
# docker-compose version 1.23.1, build b02f1306
```

Clone the repository on your workstation.

```
git clone https://github.com/gearnode/local-gateway.git && cd local-gateway
```

Build NGINX and DNSMasq image with docker compose.

```sh
docker-compose build
```

Start the DNSMasq and NGINX server in daemon mode.
```sh
docker-compose up -d
```

Configure your operating system to send all `*.dev` DNS queries to your
_Local Gateway_ DNS server. To do this, Create a new file named `dev` in
the `/etc/resolver/` directory and add the nameserver to it.

```sh
# Create resolver folder when the folder does not exist.
sudo mkdir -p /etc/resolver

# Create the dev resolve file
sudo tee /etc/resolver/dev >/dev/null <<EOF
nameserver 127.0.0.1
EOF
```

Test your new configuration by performing DNS lookup.
Use `host` (or `dig`) software to check that you can now resolve
some DNS names in your new top-level domain.

```sh
# Make sure you haven't broken your DNS.
> host -t a github.com
github.com has address 140.82.118.3
github.com has address 140.82.118.4

# Check that .dev tld works
> host this.is.a.test.dev
this.is.a.test.dev has address 127.0.0.1

> host acme.dev
acme.dev has address 127.0.0.1

```

You should see results that mention the IP 127.0.0.1 as shown above.

## Usage

This section describes different ways to use the _Local Gateway_.

### Proxy to an application running on the host

- explain the host.docker.internal dns
- example of nginx configuration

```nginx
upstream backend {
  server host.docker.internal:3000;
}

server {
  listen 80;

  server_name www.acme.dev;

  location / {
    proxy_pass http://backend;

    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $remote_addr;
  }
}
```

### Proxy to an application running in a docker container

TODO

- explain how to bind containers to a existing network
- explain how to resolve containers by dns
- example of nginx configuration

### Proxy with TLS

- explain [mkcert](https://github.com/FiloSottile/mkcert)
or auto generate cert or others certs...
- configure nginx + e.g.

```nginx
upstream backend {
  server host.docker.internal:3000;
}

server {
  listen 443 ssl;

  server_name www.acme.dev;

  ssl_certificate /etc/certs/www-acme-dev.pem;
  ssl_certificate_key /etc/certs/www-acme-dev-key.pem;

  ssl_ciphers EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH;
  ssl_protocols TLSv1.1 TLSv1.2;

  location / {
    proxy_pass http://backend;

    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $remote_addr;
  }
}
```

### Register other TLDs

TODO

- how to update your dnsmasq config


### gRPC

TODO

## Roadmap

### v1

- [ ] full documentation
- [ ] tested on linux
- [ ] make target to generate nginx conf easily

## Project status

TODO

## Contributing

Pull requests are welcome. For major changes, please open an issue first to
discuss what you would like to change.

## Maintainers

- [Bryan Frimin](https://github.com/gearnode)

See also the list of [contributors](https://github.com/gearnode/local-gateway/contributors) who participated in this project.

## License

This project is licensed under the Apache License Version 2.0 - see the [LICENSE](LICENSE) file for details
