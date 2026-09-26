# Dominio agroyauri.com → Cloudflare (web) + cPanel naxhosting (correo)

Dominio y hosting cPanel en **naxhosting** (cliente.naxhosting.com). La web vive en **Cloudflare Pages** (proyecto `agroyauri`); el cPanel se usa solo para el **correo**.
Método elegido (igual que cmiperu.pe): **nameservers a Cloudflare**.

## DNS original (hosting naxhosting, 26-09-2026) — respaldo
Servidor del hosting: **213.136.74.139**. NS originales: `ns1.docenteactivo.com`, `ns2.docenteactivo.com`.

| Tipo | Nombre | Valor |
|---|---|---|
| A | agroyauri.com | 213.136.74.139 |
| CNAME | www | agroyauri.com |
| CNAME | mail | agroyauri.com |
| A | webmail, cpanel, whm, webdisk, ftp, autoconfig, autodiscover, cpcontacts, cpcalendars | 213.136.74.139 |
| MX | agroyauri.com | 0 agroyauri.com |
| TXT | agroyauri.com | `v=spf1 ip4:213.136.74.139 +a +mx +ip4:57.128.101.139 ~all` |
| TXT | _dmarc | `v=DMARC1; p=none;` |
| TXT | default._domainkey | `v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA1uRq262rVY6p61P/l1SkAp7f4mmaIIw4cxoN6yo8W64ZAJL+d+bY5WEXyIvn1b33zsov+eQI8COKJSrJfBd4Q/WBX/HaWmsCXOMFijlBHSGFs0ylCCjZOll9MTq+n5VZWGSzFD5MyPVSaYy4T5UWmZJWzx5PxBnMr+WxGx2Uzp2Q0jt20sG/ccwogTusUOiRUyPXHNkElQI9dosaWRGxeF/1TNw4796b7jpDYKcnNXT0/qN2meHMOWFDeEIWrX7FYdiuRRt+1+FieupGMIHMqtT5vHLev1gf5W7kEoJwiCA+O8RsSz8JqK8V8hNRWrSKYaoRFK3x6FSiAnh3OMfEvQIDAQAB;` |
| SRV | _autodiscover._tcp | 0 0 443 cpanelemaildiscovery.cpanel.net |

## ⚠️ Trampa del correo
El MX apunta a **agroyauri.com** y `mail` es CNAME de **agroyauri.com**. Cuando `agroyauri.com` pase a la web (Cloudflare), el correo se perdería.
**Por eso en Cloudflare**:
- `mail` debe ser **A → 213.136.74.139** (no CNAME).
- **MX** de `agroyauri.com` → **`mail.agroyauri.com`** (prioridad 0).

## DNS objetivo en Cloudflare (todo con nube GRIS = "DNS only", salvo la web)
| Tipo | Nombre | Valor | Proxy |
|---|---|---|---|
| A | mail | 213.136.74.139 | gris |
| A | webmail, cpanel, whm, webdisk, ftp, autoconfig, autodiscover, cpcontacts, cpcalendars | 213.136.74.139 | gris |
| MX | @ | mail.agroyauri.com (0) | — |
| TXT | @ | SPF (igual al original) | — |
| TXT | _dmarc | igual | — |
| TXT | default._domainkey | DKIM igual (completo) | — |
| SRV | _autodiscover._tcp | 0 0 443 cpanelemaildiscovery.cpanel.net | — |
| (web) | agroyauri.com y www | los crea Cloudflare al agregar los Custom domains en Pages | naranja |

Se BORRAN en Cloudflare: A `agroyauri.com` → 213.136.74.139 y CNAME `www` → agroyauri.com (los reemplazan los Custom domains de Pages).

## Después del cambio
- cPanel: **https://cpanel.agroyauri.com** (agroyauri.com/cpanel deja de funcionar). Webmail: **https://webmail.agroyauri.com**.
- Cloudflare Pages → Settings → Variables: `PUBLIC_SITE_URL=https://agroyauri.com` → redeploy. Supabase → Auth → Site URL.
- Verificar: `dig +short agroyauri.com MX` → `0 mail.agroyauri.com.`; `dig +short mail.agroyauri.com` → 213.136.74.139; enviar y recibir un correo de prueba.
