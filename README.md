# Caddy-Cloudflare-Crowdsec

[![GitHub License](https://img.shields.io/github/license/RubenLou/caddy-cloudflare-crowdsec)](https://github.com/RubenLou/caddy-cloudflare-crowdsec/blob/main/LICENSE)
[![GitHub Actions Workflow Status](https://img.shields.io/github/actions/workflow/status/RubenLou/caddy-cloudflare-crowdsec/.github%2Fworkflows%2Fpipeline.yml)](https://github.com/RubenLou/caddy-cloudflare-crowdsec/actions/workflows/pipeline.yml)
[![Docker Image Pull Count](https://img.shields.io/docker/pulls/monstr0/caddy-cloudflare-crowdsec)](https://hub.docker.com/r/monstr0/caddy-cloudflare-crowdsec)

Caddy image with integrated support for Cloudflare DNS and Crowdsec.

## Images

No image is built manually. Everyday at `00:00h`, an action is ran to check for updates on any of the dependencies. If an update is found, a new image is built and updated with the `:latest` and `:YYYYMMDDHHMMSS` tags.

## Usage

**docker cli**
```
docker run -d \
  --name=caddy \
  --restart unless-stopped \
  -e CF_API_TOKEN=REPLACE_ME \
  -e CROWDSEC_LOCAL_API_KEY=REPLACE_ME \
  -p 80:80 \
  -p 443:443 \
  -v ~/path/caddy/Caddyfile:/etc/caddy/Caddyfile \
  -v ~/path/data/caddy:/dat \
  monstr0/caddy-cloudflare-crowdsec:latest
```

**docker compose**
```
services:
  caddy:
    image: monstr0/caddy-cloudflare-crowdsec:latest
    container_name: caddy
    restart: unless-stopped
    ports:
      - 80:80
      - 443:443
    environment:
      - CF_API_TOKEN=REPLACE_ME
      - CROWDSEC_LOCAL_API_KEY=REPLACE_ME
    volumes:
      - ~/path/caddy/Caddyfile:/etc/caddy/Caddyfile
      - ~/path/data/caddy:/data
```

**Caddyfile**
```
{
        order crowdsec first

        crowdsec {
                api_url http://localhost:8080
                api_key {$CROWDSEC_LOCAL_API_KEY}
        }

        # set the correct client ip header https://github.com/hslatman/caddy-crowdsec-bouncer#client-ip
        servers {
                trusted_proxies cloudflare
                client_ip_headers Cf-Connecting-Ip
        }
}

# Log accesses for crowdsec log analysis
(access_log) {
	log {
		output file /data/log/caddy.log
	}
}

# Set SSL/TLS encription mode to Full (strict) https://dash.cloudflare.com/24215ba4f9e7d7664f1c6acbb31e0ec3/{$MY_DOMAIN}/ssl-tls
(cloudflare) {
	tls {
		dns cloudflare {$CLOUDFLARE_API_TOKEN}
		resolvers 1.1.1.1
	}
}

website.com {
        crowdsec
        reverse_proxy localhost:1234
        import cloudflare
        import access_log
}
```

## Notes

If you wish to check a working demo, please refer to this [docker-compose demo](demo/README.md)

Check this page for more configurations: https://github.com/serfriz/caddy-custom-builds/pkgs/container/caddy-cloudflare-crowdsec#configuration