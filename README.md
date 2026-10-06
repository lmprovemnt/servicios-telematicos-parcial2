# Parcial 2 - Servicios Telemáticos

**Estudiante:** Código 2225348  
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

cat << 'EOF' > .gitignore
.vagrant/
.DS_Store
*.key
*.pem
