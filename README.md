# Local Gateway

> Local Gateway provide is all in one DNS server with TLS Nginx proxy
configuration packaged with docker compose.

## Table of Contents

- [Abstract](#abstract)
- [Requirements](#requirements)
- [Setup](#setup)
- [Usage](#usage)
    - [Proxy to application running on the host](#proxy-to-application-running-on-the-host)
    - [Proxy to application running on docker container](#proxy-to-application-running-on-docker-container)
    - [Proxy with TLS](#proxy-with-tls)
    - [Register other TLD](#register-other-tld)
    - [gRPC](#grpc)
- [Roadmap](#roadmap)
- [Project status](#project-status)
- [Contributing](#contributing)
- [Maintainers](#maintainers)
- [License](#license)

## Abstract

TLS is a must in the networked development.

Working locally without TLS requires dev specific configurations (e.g. HSTS,
CSP, secure cookie, etc.) and prevents some iteration with external server (e.g.
SSO, webhook, etc).
Also, configuring each application to boot with TLS mode is
hard.

More, working with multiple applications or with an application
that deal with subdomain, having a DNS server is a must too.

The goal of this project is to provide a simple DNS server with TLS proxy
without impacting development applications and reducing the different between local
environment and production environment.

## Requirements

Using Local Gateway requires:

- [Docker](https://www.docker.com/) 18.09+
- [Docker Compose](https://docs.docker.com/compose/) 1.23+

## Setup

Ensure you have the required Docker and Docker compose version.

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

Build NGINX and DNSMasq images with docker compose.

```sh
docker-compose build
```

Start the DNSMasq and NGINX servers in daemon mode.
```sh
docker-compose up -d
```

Configure your operating system to send all `.dev` DNS queries to Local Gateway
DNS server. To achieve this, create a new file named `dev` in the `/etc/resolver/` directory and add the
nameserver to it.

```sh
# Create resolver folder when the folder does not exist.
sudo mkdir -p /etc/resolver

# Create the dev resolve file
sudo tee /etc/resolver/dev >/dev/null <<EOF
nameserver 127.0.0.1
EOF
```

To test your new configuration, use `ping` in order to check that you can now resolve some DNS names on
your new top-level domain.

```sh
# Make sure you haven't broken your DNS.
ping -c 1 www.github.com

# Check that .dev tld work
ping -c 1 this.is.a.test.dev
ping -c 1 acme.dev
```

You should see results that mention the IP 127.0.0.1 like this:

```
PING this.is.a.test.dev (127.0.0.1): 56 data bytes
```

## Usage

This section describe different ways to use the Local Gateway.

### Proxy to application running on the host

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

### Proxy to application running on docker container

TODO

- explain how bind container to a existing network
- explain how resolve container by dns
- example of nginx confguration

### Proxy with TLS

- explain mkcert or auto generate cert or others certs...
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

### Register other TLD

TODO

- how update dnsmasq config


### gRPC

TODO

## Roadmap

### v1

- [ ] full documentation
- [ ] tested on linux
- [ ] make target to generate nginx conf easly

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
