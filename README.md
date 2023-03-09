# OPNSense Firewall with traefik-wp-redis Docker compose site
Docker compose for Traefik front Wordpress server with redis accelerator

## Purpose
This project is intended as an exercise in network management and is not production ready. Most of these instructions are directly applicable to production site install with some hardening required.
## Site logical layout

Project is designed to be executed in any virtualization environment that allows for multiple VMs and nested virtualization. 
Image below describes premise of this exercise. 
![project server layout](/assets/images/fw-traefik-wp-redis.png "fw-traefik-wp-redis")

Project application part is designed to be hosted as docker compose stack. This will setup Wordpress with empty site and additional Redis server as accelerator. This will not do WP config required to use Redis. 

## Setting up docker compose

docker-compose.yml contains necessary config to spin up WP site but it needs some modification depending on your setup. 

Traefik proxy needs to know hostname or IP of the host fronting the site. On image above setup is made to be port forwarded to WAN through OPNSense. This means that hostname argument should be set to OPNSense wan ip or domain name. 
```yaml
      - "traefik.http.routers.wordpress.rule=Host(`hostname`)"
      - "traefik.http.routers.wordpress.entrypoints=web"
      - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"
      - "traefik.http.routers.wordpress.middlewares=redirect-to-https@docker"
      - "traefik.http.routers.wordpress-secured.rule=Host(`hostname`)"
      - "traefik.http.routers.wordpress-secured.entrypoints=websecure"

```

## Setting up Wordpress

Compose file sets up traefik with port 8080 as traefik dashboard and http (tcp 80) and https (tcp 443) as wordpress front. There is also middleware to redirect http to https. Https certificate is done with acme account but can be easily change to using referenced tls cert. Using tls with traefik is outside of this exercise but this is explained at traefik site **[here](https://doc.traefik.io/traefik/https/overview/)**.

After cloning repo, you can start up wordpress with:
```
docker compose up -d
```
With linux host, preferred way is to install docker engine instead of docker desktop. For lab environment this can be done with convenience script. Script is provided by Docker with caveat of non production usage. 

Docker Convenience Script can be downloaded and ran with:
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```
After this Wordpres can be accessed with http://hostname that should redirect connection to https site. 

## OPNSense portforward 

Firewall settings

Firewall -> Settings -> Advanced:

```
- Reflection for port forwards: Enabled
- Reflection for 1:1: Disabled
- Automatic outbound NAT for Reflection: Enabled
```
Save.

Port Forwarding:

- You have a host with IP 192.168.1.200, with port 80 open TCP.

- You want to port forward from the outside http to inside http.

Step 1: Set up aliases

Too simple explanation: Aliases are friendly names to IP addresses. If you're managing a bunch of IPs to forward, it's best to give the IP address a label.

Under firewall > aliases > add a new alias

```
- name: A short friendly name for the IP address you're aliasing. Like 'WebServer' or 'Wordpress'
- type: Host(s)
- Aliases: Input 192.168.1.200
```
Save.

Step 2: Register the port forward

Go to:
Firewall > NAT > Port forward > add

```
- Interface: WAN
- TCP/IP Version: IPv4
- Protocol: TCP
```
Under Source > Advanced:
```
- Source / Invert: Unchecked
- Source: Any
- Source Port Range: any to any

- Destination / Invert: Unchecked
- Destination: WAN address
- Destination Port range: http to http

- Redirect target IP: Alias "wordpress"
- Redirect target Port: http

- Pool Options: Default
- NAT reflection: Enable
- Filter rule association: Rule NAT
```
Save, and you now should be able to forward an incoming http to traefik http port.