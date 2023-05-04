# Traefik Proxy

___

## Setup

1. Clone this repository locally.

    ```sh
    git clone https://github.com/design-group/traefik-proxy.git traefik-proxy && cd traefik-proxy
    ```

2. Pull any changes to the docker image and start the container.
      
    On Mac:
    
	```sh
    docker compose pull && docker compose up -d
    ```
    
	On WSL
    
	```sh
    docker-compose pull && docker-compose up -d
    ```

3. Generate the certificates for the `localtest.me` domain
   
	```sh
	./generate-certs.sh
	```

4. Trust the root ca certificate generated by the script based on your OS.

	- For Windows: https://learn.microsoft.com/en-us/skype-sdk/sdn/articles/installing-the-trusted-root-certificate
	- For MacOS: https://support.apple.com/guide/keychain-access/add-certificates-to-a-keychain-kyca2431/mac
	- For Linux: https://www.techrepublic.com/article/how-to-install-ca-certificates-in-ubuntu-server/

___

## Usage

To add an Ignition Gateway or other container to the proxy, add the following labels and environment variables to the container:

```yaml
labels:
  traefik.enable: "true"
  traefik.hostname: <desired-address>
environment:
  GATEWAY_SYSTEM_NAME: <desired-address>
  GATEWAY_PUBLIC_HTTP_PORT: 80
  GATEWAY_PUBLIC_HTTPS_PORT: 443
  GATEWAY_PUBLIC_ADDRESS: <desired-address>.localtest.me
```

For example, to add an Ignition Gateway to the proxy, add the following labels and environment variables to the container:

```yaml
labels:
  traefik.enable: "true"
  traefik.hostname: ignition
environment:
  GATEWAY_SYSTEM_NAME: ignition
  GATEWAY_PUBLIC_HTTP_PORT: 80
  GATEWAY_PUBLIC_HTTPS_PORT: 443
  GATEWAY_PUBLIC_ADDRESS: ignition.localtest.me
```

The traefik proxy is configured to use the `proxy` network. If the container is not on the `proxy` network, add the following to the container:

```yaml
networks:
  - default
  - proxy
```

Then add the following to the `docker-compose.yml` file:

```yaml
networks:
  default:
  proxy:
    external: true
    name: proxy
```

After adding the labels and environment variables, restart the container. Once the container restarts, it should be accessible at `https://<container-name>.localtest.me/`.

___

## Projects with pre-existing docker-compose.yml files

If the project already has a `docker-compose.yml` file, you can add a file named `docker-compose.traefik.yml` to the project. This file will be used to add the labels and environment variables to the container. In order to get your local environment to use this file, add the following to the `.env` file:

```sh
COMPOSE_PATH_SEPARATOR=:
COMPOSE_FILE=docker-compose.yml:docker-compose.traefik.yml
```

Then, add the labels and environment variables to the `docker-compose.traefik.yml` file. For example, here is the pre-existing `docker-compose.yml` file for the Ignition Gateway:

```yaml
services:
  ignition:
    image: bwdesigngroup/ignition-docker:8.1.22
    ports:
      - 8088:8088
    volumes:
      - ignition-data:/workdir
```

Here is the `docker-compose.traefik.yml` file for the Ignition Gateway:

```yaml
services:
  ignition:
    labels:
      traefik.enable: "true"
      traefik.hostname: ignition
    environment:
      GATEWAY_SYSTEM_NAME: ignition
      GATEWAY_PUBLIC_HTTP_PORT: 80
      GATEWAY_PUBLIC_HTTPS_PORT: 443
      GATEWAY_PUBLIC_ADDRESS: ignition.localtest.me
    networks:
      - default
      - proxy

networks:
  default:
  proxy:
    external: true
    name: proxy
```

After adding the labels and environment variables, re-up the container. Once the container restarts, it should be accessible at `https://ignition.localtest.me/`.

```sh
docker-compose up -d
```

---

## Enabling TLS for a secure connection over TCP

If you want to enable TLS for a secure connection over TCP, add the following labels to the container:

```yaml
traefik.tcp.routers.<router-name>.tls: "true"
traefik.tcp.routers.<router-name>.rule: "HostSNI(`<desired-address>.localtest.me`)"
traefik.tcp.routers.<router-name>.service: "<router-name>"
traefik.tcp.services.<router-name>.loadbalancer.server.port: "<port>"
```

For example, to enable TLS for a secure connection over TCP for a Neo4J Graph Database, add the following labels to the container:

```yaml
traefik.tcp.routers.neo4j.tls: "true"
traefik.tcp.routers.neo4j.rule: "HostSNI(`neo4j.localtest.me`)"
traefik.tcp.routers.neo4j.service: "neo4j"
traefik.tcp.services.neo4j.loadbalancer.server.port: "7687"
```