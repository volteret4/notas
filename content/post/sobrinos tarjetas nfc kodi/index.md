---
title : "sobrinos, tarjetas NFC para poner música y películas en Kodi"
slug : "sobrinos-tarjetas-nfc-kodi"
date : "2026-10-31"
image : ""
tags : ["nfc", "raspberry", "kodi", "python", "tmdb", "sops", "acr122u"]
categories : ["scripts"]
description : "Una Raspberry con un lector NFC que, al acercar una tarjeta impresa, reproduce en Kodi un disco, un artista, una película o una serie. Con portadas imprimibles en A4 y una web por álbum."
---

# Qué es

Repo: `~/gits/pollo/sobrinos` (`git@github.com:volteret4/sobrinos.git`). Documentación detallada en `instalacion.md` y `uso.md` del propio repo.

- **Lector**: ACR122U en una Raspberry Pi (usuario `mito`), con `pcscd`. Tarjetas MIFARE Classic y, para las nuevas, **NTAG213/215**.
- **Reproductor**: Kodi, por JSON-RPC (`http://IP_KODI:8080/jsonrpc`, con usuario y contraseña del control remoto).
- **Contenido**: el programa indexa películas, series, álbumes y artistas, y al asociar una tarjeta pregunta primero la categoría y luego el elemento.
- **Portadas**: tarjetas de 52×82 mm, delante y detrás, con un único QR a la web del álbum en GitHub Pages. Las de **artista** llevan su foto delante (Last.fm, Wikipedia o MusicBrainz; si no la encuentra, pide una ruta) y detrás un collage de sus discos. Las de **cine** salen de TMDB.
- **Impresión**: `imprimir_tarjetas.py` las junta en A4 (9 por página), con modo dúplex y marcas de corte.
- **Android como lector** (tarjetas NTAG): una página Flask servida por nginx en `raspberrypi.local`, en el mismo proceso que lee las tarjetas.
- **Secretos**: `secrets.enc.yaml` cifrado con sops + age.

# Instalación (resumen)

```bash
sudo apt-get install -y libacsccid1 pcscd pcsc-tools nginx avahi-daemon fonts-dejavu-core && sudo systemctl enable --now pcscd.socket avahi-daemon
```

Desactivar los módulos NFC del kernel, que se pelean con `pcscd`, en `/etc/modprobe.d/blacklist.conf`:

```
install nfc /bin/false
install pn533 /bin/false
```

```bash
sudo usermod -aG plugdev $USER
```

```bash
python3 -m venv ~/Scripts/python_venv && ~/Scripts/python_venv/bin/pip install -r requirements.txt
```

sops para arm64 (`.deb` de github.com/getsops/sops) y la clave age copiada de la máquina principal a `~/.config/sops/age/keys.txt`. Luego:

```bash
sops edit secrets.enc.yaml
```

# Uso

| Script | Para qué |
|---|---|
| `orquestar.py` | flujo completo: portada → asociar tarjeta → actualizar web |
| `nfc/nfc_detect_uid.py` | leer el UID de una tarjeta |
| `nfc/nfc_config_gen.py` | asociar UID ↔ contenido |
| `nfc/nfc_reaccionar.py` | el monitor que reproduce al leer |
| `nfc/nfc_generar_portada.py`, `nfc_generar_portada_artista.py`, `nfc_tarjetas_cine.py` | portadas |
| `nfc/imprimir_tarjetas.py --entrada docs/portadas/musica` | A4 |
| `album_web_generator.py` | web del álbum (GitHub Pages, carpeta `docs/`) |

Servicio en la Raspberry, `/etc/systemd/system/sobrinos.service`:

```ini
[Unit]
Description=Sobrinos NFC Monitor
After=network.target

[Service]
User=mito
WorkingDirectory=/home/mito/gits/pollo/sobrinos
ExecStart=/home/mito/Scripts/python_venv/bin/python nfc/nfc_reaccionar.py
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Errores que salieron: con `sudo` no se usaba el venv del usuario (por eso el servicio usa la ruta absoluta del intérprete), el bucle de "No smart card inserted" llenaba el log (silenciado), y la información de los discos se guardaba en la carpeta de la música (ahora va dentro del proyecto).

# Revertir

```bash
sudo systemctl disable --now sobrinos && sudo rm /etc/systemd/system/sobrinos.service /etc/nginx/sites-enabled/sobrinos
```

Quita las dos líneas de `blacklist.conf` si quieres volver a usar los módulos NFC del kernel.
