```markdown
# Parcial 2 - Servicios Telemáticos

**Estudiantes:** Juan Camilo Gonzalez Rodas (2226090), Roiman Urrego Zuñiga (2231385) y Santiago Valencia Sandoval 2225348
                 
**Entorno:** macOS / Vagrant (Ubuntu 22.04 LTS)

---

## 🌐 Topología de Red

* **`srv1` (Firewall / Router / NAT):**
  * IP externa: `192.168.60.1`
  * IP interna: `192.168.50.1`
  * Reenvío de puertos NAT (DNAT puerto `2222` -> `192.168.50.2:22`).
* **`srv2` (Servidor de Servicios):**
  * IP: `192.168.50.2`
  * Servicios: FTPS Explícito (`vsftpd`) y SFTP Restringido (`OpenSSH`).
* **`client` (Cliente de Pruebas):**
  * IP: `192.168.60.2`
  * Cliente DNS con **DNS over TLS (DoT)** configurado en `systemd-resolved`.

---

## 🔐 Servicios Implementados y Verificación

### 1. FTPS Explícito (TLSv1.3) — `vsftpd` en `srv2`
* Configurado detrás de NAT con rango de puertos pasivos `50000-50010` y `pasv_address=192.168.60.1`.
* Certificado SSL emitido por CA propia (`CA-ServiciosTelematicos`).

**Comandos de verificación desde `client`:**
```bash
# Validar Handshake TLSv1.3
openssl s_client -connect 192.168.60.1:21 -starttls ftp -CAfile ca.crt

# Subir archivo en modo pasivo cifrado
curl --cacert ca.crt --ssl-reqd -u usuarioftp:furbysanti10 -T 2225348.txt ftp://srv2/ --resolve srv2:21:192.168.60.1

# Descargar archivo cifrado
curl --cacert ca.crt --ssl-reqd -u usuarioftp:furbysanti10 -O ftp://srv2/2225348.txt --resolve srv2:21:192.168.60.1

```

---

### 2. DNS over TLS (DoT) — `systemd-resolved` en `client`

* Configurado en `/etc/systemd/resolved.conf` utilizando resolvers públicos (`1.1.1.1#cloudflare-dns.com` y `8.8.8.8#dns.google`).
* Modo estricto habilitado (`DNSOverTLS=yes`) para garantizar la privacidad total en el transporte de consultas DNS por el puerto TCP 853.
* `/etc/resolv.conf` enlazado al stub resolver local (`127.0.0.53`).

**Comandos de verificación:**

```bash
# Verificar estado del servicio y DoT activo
resolvectl status

# Consultas de prueba (evitando la caché local)
sudo resolvectl flush-caches
resolvectl query uao.edu.co
resolvectl query debian.org
resolvectl query github.com

```

---

### 3. SFTP Seguro con OpenSSH y NAT — `OpenSSH` en `srv2` + `ufw/iptables` en `srv1`

* **Servidor 2 (`srv2`):** Usuario exclusivo `sftp_2225348` (o usuario designado) enjaulado en su directorio mediante `ChrootDirectory /var/sftp`, `ForceCommand internal-sftp` y sin acceso a terminal interactivo shell (`PermitTTY no`).
* **Servidor 1 (`srv1`):** Reenvío de puerto perimetral (DNAT en puerto `2222` a `192.168.50.2:22`) y reglas de enrutamiento en UFW (`ufw route allow proto tcp to 192.168.50.2 port 22`).

**Comandos de verificación desde `client`:**

```bash
# Probar que el acceso shell interactivo es rechazado
ssh -p 2222 sftp_2225348@192.168.60.1

# Sesión interactiva SFTP
sftp -P 2222 sftp_2225348@192.168.60.1

# Comandos dentro de la consola SFTP:
sftp> cd uploads
sftp> ls
sftp> put 2225348.txt
sftp> get 2225348.txt
sftp> bye

```

---

## 📊 Tabla Comparativa: FTPS vs. SFTP

| Criterio | FTPS (FTP over TLS) | SFTP (SSH File Transfer Protocol) |
| --- | --- | --- |
| **Protocolo Base** | FTP extendido con TLS/SSL. | SSH (Secure Shell). |
| **Conexiones y Puertos** | **Múltiples:** Control en TCP 21 + Rango de puertos pasivos (ej. 50000-50010). | **Único:** 1 sola conexión TCP (puerto 2222/22). |
| **Autenticación del Servidor** | Certificados digitales X.509. | Claves de Host SSH (*Host Keys*). |
| **Inicio del Cifrado** | Tras comando explícito `AUTH TLS`. | Desde el intercambio inicial de llaves SSH. |
| **Atravesar Firewalls / NAT** | **Complejo:** Requiere abrir rangos dinámicos y configurar `pasv_address`. | **Simple:** Solo requiere reenviar un único puerto TCP. |
| **Configuración** | Alta complejidad (gestión de CA, certificados y puertos pasivos). | Baja complejidad (integrado nativamente en OpenSSH). |

**Conclusión:** **SFTP** es la solución ideal para entornos con restricciones estrictas de firewall debido a que encapsula autenticación, comandos y transferencia de datos en un **único canal TCP multiplexado**, simplificando significativamente las políticas de red e inspección.


```

```
