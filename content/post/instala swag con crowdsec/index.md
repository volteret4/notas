---
title : "Instala Swag Con Crowdsec"
date : "2026-03-23"
image : ""
tags : ["cloud", "crowdsec", "nginx", "notificaciones", "ssh"]
categories : ["Seguridad"]
description : "Configura tu proxy favorito añadiendo una capa extra de seguridad con crowdsec y notificaciones via telegram."
---



# Configuración
Usa un docker compose como este para arrancar los dos servicios, uso duckdns para mis dominios gratuitos, cámbialos según necesidad

```yaml
services:
  swag:
    image: lscr.io/linuxserver/swag
    container_name: swag
    cap_add:
      - NET_ADMIN
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
      - URL=$DOMAIN.DUCKDNS
      - SUBDOMAINS=wildcard
      - VALIDATION=dns
#      - CERTPROVIDER= #optional
      - DNSPLUGIN=duckdns #optional
      - EMAIL=frodobolson@disroot.org #optional
      - ONLY_SUBDOMAINS=false #optional
#      - EXTRA_DOMAINS=<extradomains> #optional
#      - STAGING=false #optional
      - DOCKER_MODS=ghcr.io/linuxserver/mods:swag-crowdsec #|linuxserver/mods:swag-dashboard
      - CROWDSEC_API_KEY=$API_KEY # la obtendremos luego con cscli
      - CROWDSEC_LAPI_URL=http://crowdsec:8080
    volumes:
      - $HOME/contenedores/herramientas/swag/config:/config
      - $HOME/contenedores/podcast-tts:/config/www/podcast-tts
      - $HOME/gits/pollo/dieta_sonora:/config/www/musica
    networks:
      - swag

    ports:
      - 81:81
      - 443:443
      - 80:80 #optional
    restart: unless-stopped


  crowdsec:
    image: crowdsecurity/crowdsec:latest
    container_name: crowdsec
    restart: unless-stopped
    environment:
      - GID=1000
      - CUSTOM_HOSTNAME=pepecono
      - COLLECTIONS= crowdsecurity/nginx crowdsecurity/whitelist-good-actors crowdsecurity/http-cve
#        crowdsecurity/appsec-generic-rules
#        crowdsecurity/appsec-virtual-patching
    ports:
      - "6060:6060"  # optional metrics
      - "8080:8080"  # local API for bouncers
    volumes:
      - $HOME/contenedores/herramientas/swag/crowdsec/config:/etc/crowdsec
      - $HOME/contenedores/herramientas/swag/crowdsec/data:/var/lib/crowdsec/data
      - $HOME/contenedores/herramientas/swag/log/nginx:/var/log/nginx:ro  # Mount your Nginx logs here
      - /var/log:/var/log/host:ro # aqui puedes montar los logs del host
    security_opt:
      - no-new-privileges=true
    networks:
      - swag
networks:
  swag:
    external: true
```

Una vez arrancados ambos contenedores debes usar `cscli` dentro del contenedor, sería cómodo usar un alias como 
```sh
alias cscli="docker exec -t crowdsec cscli"
```
Luego haremos para consultar si tenemos bouncers activos
```sh
cscli bouncers list
# en caso de no existir lo añadiremos para swag con 
cslcli bouncers add swag
```

Copiaremos la api-key que nos proporciona en el `docker-compose.yml` (¡¡sin comillas!!)y recompondremos el contenedor. Tras esto veremos que el primer commando muestra una tabla con ultima fecha de conexión. 

## Notificaciones

Para añadir notificaciones con telegram modificaremos el perfil que se encuentra en `crowdsec/config/profiles` descomentando las notificaciones http) quedando algo asi

```yaml
name: default_ip_remediation
#debug: true
filters:
 - Alert.Remediation == true && Alert.GetScope() == "Ip"
decisions:
 - type: ban
   duration: 4h
#duration_expr: Sprintf('%dh', (GetDecisionsCount(Alert.GetValue()) + 1) * 4)
notifications:
#   - slack_default  # Set the webhook in /etc/crowdsec/notifications/slack.yaml before enabling this.
#   - splunk_default # Set the splunk url and token in /etc/crowdsec/notifications/splunk.yaml before enabling this.
  - http_default   # Set the required http parameters in /etc/crowdsec/notifications/http.yaml before enabling this.
#   - email_default  # Set the required email parameters in /etc/crowdsec/notifications/email.yaml before enabling this.
on_success: break
---
name: default_range_remediation
#debug: true
filters:
 - Alert.Remediation == true && Alert.GetScope() == "Range"
decisions:
 - type: ban
   duration: 4h
#duration_expr: Sprintf('%dh', (GetDecisionsCount(Alert.GetValue()) + 1) * 4)
notifications:
#   - slack_default  # Set the webhook in /etc/crowdsec/notifications/slack.yaml before enabling this.
#   - splunk_default # Set the splunk url and token in /etc/crowdsec/notifications/splunk.yaml before enabling this.
  - http_default   # Set the required http parameters in /etc/crowdsec/notifications/http.yaml before enabling this.
#   - email_default  # Set the required email parameters in /etc/crowdsec/notifications/email.yaml before enabling this.
on_success: break
```

Y modificaremos la notificación http editando el archivo `crowdsec/config/notifications/http.yaml`
```yaml
type: http          # Don't change
name: http_default  # Must match the registered plugin in the profile

# One of "trace", "debug", "info", "warn", "error", "off"
log_level: info

# group_wait:         # Time to wait collecting alerts before relaying a message to this plugin, eg "30s"
# group_threshold:    # Amount of alerts that triggers a message before <group_wait> has expired, eg "10"
# max_retry:          # Number of attempts to relay messages to plugins in case of error
# timeout:            # Time to wait for response from the plugin before considering the attempt a failure, eg "10s"

#-------------------------
# plugin-specific options

# The following template receives a list of models.Alert objects
# The output goes in the http request body

# Replace XXXXXXXXX with your Telegram chat ID
format: |
  {
   "chat_id": "XXXXXXXX",
   "text": "
     {{range . -}}
     {{$alert := . -}}
     {{range .Decisions -}}
     {{.Value}} will get {{.Type}} for next {{.Duration}} for triggering {{.Scenario}}.
     {{end -}}
     {{end -}}
   ",
   "reply_markup": {
      "inline_keyboard": [
          {{ $arrLength := len . -}}
          {{ range $i, $value := . -}}
          {{ $V := $value.Source.Value -}}
          [
              {
                  "text": "See {{ $V }} on shodan.io",
                  "url": "https://www.shodan.io/host/{{ $V -}}"
              },
              {
                  "text": "See {{ $V }} on crowdsec.net",
                  "url": "https://app.crowdsec.net/cti/{{ $V -}}"
              }
          ]{{if lt $i ( sub $arrLength 1) }},{{end }}
      {{end -}}
      ]
  }

url: https://api.telegram.org/botXXX:YYY/sendMessage # Replace XXX:YYY with your API key

method: POST
headers:
  Content-Type: "application/json"
```