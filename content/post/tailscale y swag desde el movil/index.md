---
title : "Una web local accesible desde el móvil con Tailscale y SWAG"
slug : "tailscale-y-swag-desde-el-movil"
date : "2026-10-31"
image : ""
tags : ["tailscale", "swag", "nginx", "pihole", "dns", "duckdns"]
categories : ["network"]
description : "Cómo llegar desde el móvil a una web de casa servida por SWAG, usando Tailscale y DNS, sin abrir más puertos."
---

# Contexto

La web de **bandcamp** (`bandcamp_imap_mails`, puerto 8765 en pepecono) se publica a través de **SWAG** (nginx con certificado wildcard de `*.pollete.duckdns.org`) en pepecono, con este bloque generado:

`~/contenedores/herramientas/swag/config/nginx/proxy-confs/bandcamp.subdomain.conf`:

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name bandcamp.*;

    include /config/nginx/ssl.conf;
    client_max_body_size 0;

    location / {
        include /config/nginx/proxy.conf;
        include /config/nginx/resolver.conf;

        set $upstream_app 192.168.1.133;
        set $upstream_port 8765;
        set $upstream_proto http;
        proxy_pass $upstream_proto://$upstream_app:$upstream_port;
    }
}
```

Desde el 21-07 este archivo lo genera `scripts/gen_proxy_confs.sh` de tumtumpa, así que los cambios a mano se pierden al regenerarlo.

En casa, **pi-hole** (en `dietpi`) resuelve cada `*.pollete.duckdns.org` que uso a la IP de pepecono (192.168.1.133).

# Desde el móvil, fuera de casa

SWAG escucha en el 443 de **todas** las interfaces de pepecono, incluida `tailscale0` (100.90.91.96). Con Tailscale en el móvil solo hace falta que el nombre resuelva a una IP alcanzable:

**Opción A, usar pi-hole como DNS de la tailnet.** En la consola de Tailscale → *DNS* → *Nameservers* → añadir la IP **de Tailscale** de `dietpi` (100.82.228.12) con *Override local DNS*. pi-hole tiene que aceptar consultas de la interfaz `tailscale0` (*Settings → DNS → Permit all origins*). El problema: pi-hole responde 192.168.1.133, que fuera de casa no existe salvo que pepecono anuncie la subred:

```bash
ssh pepecono 'sudo tailscale set --advertise-routes=192.168.1.0/24'
```

Y aprobar la ruta en la consola.

**Opción B, DNS split solo para el dominio.** Que `bandcamp.pollete.duckdns.org` resuelva a la IP de Tailscale de pepecono (100.90.91.96): en pi-hole, un registro *Local DNS* para los clientes de la tailnet, o un servidor *split DNS* para `pollete.duckdns.org` en la consola de Tailscale. El certificado wildcard sigue valiendo, porque el nombre es el mismo.

Para comprobarlo desde el móvil con Tailscale activo:

```bash
curl -sI https://bandcamp.pollete.duckdns.org | head -1
```

> Nota: la sesión del 15-06 y la del 01-07 ya no se conservan, así que no sé cuál de las dos opciones quedó aplicada. Hoy pepecono tiene `CorpDNS: true` (usa el DNS de la tailnet), no anuncia rutas (`AdvertiseRoutes: null`) y `tailscale serve` no tiene nada configurado.

# Revertir

- Opción A: quita el nameserver en la consola de Tailscale y la ruta con `sudo tailscale set --advertise-routes=`.
- Opción B: borra el registro local de pi-hole o el split DNS.
