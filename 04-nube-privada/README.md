# ☁️ Nube privada tipo Google Drive por menos de 3 USD

Guía del video: monta tu propia nube con **ownCloud Infinite Scale (oCIS)** en un VPS barato y publícala con un **túnel de Cloudflare** (sin abrir puertos ni instalar certificados).

**Redes:** @remoto360 en TikTok, Instagram, Facebook y YouTube · [remoto360.com](https://www.remoto360.com)

> Cambia en los comandos: `TU_IP` por la IP de tu VPS y `TUDOMINIO.COM` por tu dominio.

---

## 1. Compra el VPS

En **InterServer** contrata un VPS Linux con **Ubuntu** (el plan más barato alcanza). Te llegan por correo la IP y la contraseña de `root`.

## 2. Conéctate por SSH

```bash
ssh root@TU_IP
```

## 3. Crea el usuario y descarga ownCloud (oCIS)

```bash
useradd --system --home /var/lib/ocis --shell /usr/sbin/nologin ocis
mkdir -p /etc/ocis /var/lib/ocis && chown -R ocis:ocis /etc/ocis /var/lib/ocis
curl -L -o /usr/local/bin/ocis https://download.owncloud.com/ocis/ocis/stable/8.0.1/ocis-8.0.1-linux-amd64
chmod +x /usr/local/bin/ocis
```

> Si ya hay una versión más nueva, cambia `8.0.1` por esa en la URL y en el nombre del archivo.

## 4. Inicializa (aquí se genera la clave del admin, guárdala)

```bash
OCIS_CONFIG_DIR=/etc/ocis ocis init --insecure no
chown -R ocis:ocis /etc/ocis
```

## 5. Configura el dominio y el servicio

Crea `/etc/ocis/ocis.env`:

```ini
OCIS_URL=https://nube.TUDOMINIO.COM
PROXY_HTTP_ADDR=127.0.0.1:9200
PROXY_TLS=false
OCIS_CONFIG_DIR=/etc/ocis
OCIS_BASE_DATA_PATH=/var/lib/ocis
```

Crea `/etc/systemd/system/ocis.service`:

```ini
[Unit]
Description=ownCloud Infinite Scale
After=network.target

[Service]
User=ocis
EnvironmentFile=/etc/ocis/ocis.env
ExecStart=/usr/local/bin/ocis server
Restart=always

[Install]
WantedBy=multi-user.target
```

Arráncalo:

```bash
systemctl daemon-reload && systemctl enable --now ocis
```

## 6. Tu dominio en Cloudflare

Cloudflare es gratis. Dos opciones:

- **Ya tienes dominio:** agrégalo en Cloudflare y cambia los nameservers donde lo compraste.
- **No tienes:** cómpralo directo en Cloudflare.

## 7. Publícala con un túnel

1. Cloudflare → **Zero Trust → Networks → Tunnels → Create tunnel** (Cloudflared).
2. Copia el comando de instalación que te da (trae tu token) y pégalo en el VPS.
3. En **Published application**: subdominio `nube.TUDOMINIO.COM` → **HTTP** → `localhost:9200`.

## 8. Entra

Abre `https://nube.TUDOMINIO.COM`

- Usuario: `admin`
- Clave: la que salió en el paso 4

Instala la app **ownCloud** en el celular y activa la **subida automática de fotos**.

---

## ⚠️ Notas

- **Límite del plan gratis de Cloudflare:** 100 MB por petición; los archivos muy grandes subidos desde el navegador pueden fallar.
- **Seguridad:** esta guía deja la nube funcionando. Para proteger el servidor (llaves SSH, firewall, CrowdSec) mira el video anterior de la serie.
- Si algún video no se reproduce en el navegador (solo audio), suele ser formato HEVC del iPhone: en Ajustes → Cámara → Formatos elige **Más compatible**.
