# Demo with docker-compose

## Background

This example shows how to use CrowdSec with Caddy with docker-compose and a common network topology (similar to [this](https://www.crowdsec.net/blog/secure-docker-compose-stacks-with-crowdsec)):

```
┌────────────┐   ┌──────────┐    ┌───────────┐
│ Cloudflare ├──►│  Caddy   ├───►│    App    │
└────────────┘   └─────┬────┘    └───────────┘
                       │               
                 ┌─────┴────┐   
                 │ Crowdsec │   
                 └──────────┘    
```

This setups considers the following:

- Cloudflare manages the DNS records of your domain, redirecting (and obfuscating with the orange cloud) the traffic to your target ip where the caddy instance lives
- SSL certificates are automatically managed by caddy with the [caddy-dns/cloudflare](github.com/caddy-dns/cloudflare) module - you need to create an API token Zone Read and DNS Edit access ((see here)[https://github.com/libdns/cloudflare])
- The [crowdsec bouncer](github.com/hslatman/caddy-crowdsec-bouncer/http) module acts as a firewall for those domains that have the `crowdsec` directive in the Caddyfile, inspecting visitors ip and allowing or denying access.
- To monitor all access for a domain, set a `log {}` directive in the Caddyfile. This log file is mounted on the crowdsec docker instance and inspected with the `acquis.d/caddy.yaml` config. Alternatively, if you are using services with specific [collections](https://app.crowdsec.net/hub/collections), you might decide to use that as opposed to enabling access logging.

## Setup

1. Update variables in the `.env` file. Edit all `~/path` as relevant to your setup
2. Add the Caddy bouncer, generating an API key with `docker-compose exec crowdsec cscli bouncers add caddy-bouncer`. Store that key in the `.env` file
3. Copy the following files to the folders:
   - `crowdsec/caddy.yml` file to the folder `~/path/crowdsec/acquis.d`
   - `caddy/Caddyfile` to the folder `~/path/caddy`
4. Run with `docker-compose up --build --force-recreate`

## Testing

1. Run the command `docker compose exec crowdsec cscli decisions add --ip YOUR_IP`
2. Wait a few seconds and verify that you get a **403** error trying to access your website
3. Run the command to remove the decision `docker compose exec crowdsec cscli decisions remove --ip YOUR_IP`

## Credits

Adapted from https://github.com/hslatman/caddy-crowdsec-bouncer-example/pull/2
