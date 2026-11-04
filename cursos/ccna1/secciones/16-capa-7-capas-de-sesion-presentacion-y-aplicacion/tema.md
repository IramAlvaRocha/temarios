# 16 — Capa 7: Capas de Sesión, Presentación y Aplicación

> Duración del curso: 1h 00m · 7 clases
> Fuente: temario.md del curso

## Objetivos de aprendizaje
- Explicar el papel de las capas de sesión (establecimiento/gestión de diálogos), presentación (formato/cifrado/compresión) y aplicación (servicios al usuario).
- Describir arquitecturas P2P y cliente-servidor.
- Identificar los protocolos de servicios web, correo y transferencia de archivos y sus puertos.
- Explicar los servicios de direccionamiento: DNS y DHCP (proceso DORA), y su importancia en la red.

## Teoría

### 16.1 Capa de Sesión (Clase 16.1)
**OSI capa 5** — establece, mantiene, sincroniza y termina **sesiones** de diálogo entre aplicaciones:
- Control de diálogo: **simplex** (una dirección: TV/teclado→PC... en la práctica, sensores), **half-duplex** (alternado: walkie-talkie), **full-duplex** (simultáneo: teléfono/VoIP).
- Sincronización y puntos de restauración (checkpoints): recuperación tras corte sin rehacer toda la transferencia (p. ej., reanudar una descarga grande).
- Ejemplos: RPC, NetBIOS/NetBEUI (legado), SQL sessions, sesiones de red en Windows, control de diálogo en SMB.
- En el modelo TCP/IP las tres capas superiores se fusionan en "Aplicación".

### 16.2 Capa de Presentación (Clase 16.2)
**OSI capa 6** — traduce y transforma los datos para que sean "legibles" por la aplicación:
- **Formato/codificación:** ASCII ↔ EBCDIC (histórico), Unicode/UTF-8, serialización (JSON, XML, ASN.1), codificación binaria.
- **Compresión:** gzip/deflate, JPEG, MP3/4 — reduce bytes en tránsito/almacenamiento.
- **Cifrado/descifrado:** TLS/SSL (cifrado de sesión HTTPS), IPsec no (capa 3). La capa 6 no crea la clave: la app/protocolo (TLS handshake) la negocia.
- **Traducción:** poco endian vs big endian; resoluciones.
- Ejemplos: TLS sobre HTTP→HTTPS, codecs de vídeo (H.264), certificados digitales (X.509), PKCS.
- Pregunta de examen: "¿HTTPS en qué capa?" → HTTP aplicación + **TLS presentación** (modelo por capas), aunque en TCP/IP todo es "Aplicación".

### 16.3 Capa de Aplicación (Clase 16.3)
**OSI capa 7** — servicios de red directamente para el software del usuario:
- No es "la aplicación" (el navegador no es capa 7): es el **protocolo** que habla con la aplicación (HTTP en el navegador).
- Protocolos: HTTP/HTTPS, DNS, DHCP, FTP/TFTP/SFTP, SMTP/POP3/IMAP, SSH/Telnet, SNMP, NTP, SIP, LDAP, SMB.
- Modelo de servicios: el cliente solicita, el servidor responde (cliente-servidor) o ambos roles (P2P).

### 16.4 Redes P2P (Punto a Punto) (Clase 16.4)
**Peer-to-peer**: cada nodo actúa a la vez como cliente y servidor; no hay servidor central (o es opcional, p. ej., localización de tracker/semillas).
- Ejemplos: **BitTorrent** (descarga distribuida), compartición de archivos Windows (SMB/CIFS en grupos de trabajo), llamadas VoIP P2P, blockchain.
- Ventajas: sin cuello de botella central, escalabilidad natural, reparto de costes/ancho de banda.
- Problemas: seguridad distribuida difícil (no hay perímetro central), legalidad (descargas), piratería; difícil de controlar con firewalls; mal prioritario de red (mucho tráfico).
- Cliente-servidor vs P2P: CS = control central, backup y seguridad simples, coste concentrado; P2P = resiliencia y coste distribuido, gestión compleja. En la práctica muchos servicios son híbridos (BitTorrent con trackers; mensajería tipo Signal/WhatsApp con servidores de rendezvous).

### 16.5 Protocolos de Servicios Web, Email y Transferencia de Archivos (Clase 16.5)
**Web:**
- **HTTP** (RFC 9110, sucesor de 7230/2616): protocolo petición/respuesta, sin estado; métodos: GET (leer), POST (crear), PUT (actualizar), DELETE, HEAD, PATCH, OPTIONS. Códigos: 2xx éxito (200 OK, 201 creado), 3xx redirección (301/302), 4xx cliente (400, 401, 403, 404), 5xx servidor (500, 503). HTTP/1.1 conexiones persistentes; HTTP/2 multiplexado; HTTP/3 QUIC.
- **HTTPS** = HTTP sobre **TLS** (puerto 443): cifra y autentica (certificados X.509, CA en la que el SO/navegador confía). No confundir con HTTP seguro en capa 7: es cifrado en presentación.
- Servidores web: Nginx, Apache, IIS. Proxies/reverse proxies (balanceo, caché, TLS termination).

**Email (tres protocolos típicos):**
| Protocolo | Puerto | Papel |
|-----------|--------|-------|
| **SMTP** (Simple Mail Transfer Protocol) | 25 (587 sumisión con TLS, 465 SMTPS legado) | Envío/reenvío entre servidores y sumisión de cliente |
| **POP3** (Post Office Protocol v3) | 110 (995 TLS) | Descarga y típicamente borra en servidor |
| **IMAP** (Internet Message Access Protocol) | 143 (993 TLS) | Acceso en servidor; sincroniza carpetas multi-dispositivo |
- Flujo real: cliente → SMTP → servidor del remitente → (SMTP) → servidor del destinatario → IMAP/POP3 → cliente destino. SPF/DKIM/DMARC combaten suplantación (mención).

**Transferencia de archivos:**
- **FTP** (RFC 959): 20 (datos, activo) / 21 (control). Dos conexiones; modo **pasivo (PASV)** recomendado tras NAT/firewalls. Credenciales en claro.
- **SFTP**: sobre **SSH** (22): cifrado completo, recomendado.
- **FTPS**: FTP + TLS (990/control, 989/datos a veces).
- **TFTP** (RFC 1350): UDP 69, sin autenticación; para transferir configs/IOS en labs y arranque por red (PXE). Limitaciones: ficheros de hasta ~32 MB con bloques de 512 B y contador de 16 bits (RFC 1350); las extensiones RFC 2348 (blksize) amplían ese límite.
- **SCP**: copia sobre SSH (legado de SFTP, aún común).

### 16.6 Servicios de Direccionamiento (Clase 16.6)
**DNS** (Domain Name System, RFC 1034/1035):
- Traduce nombres → IPs (y viceversa): `www.cisco.com` → IP.
- Jerarquía distribuida: **root (".")** → TLD (`.com`) → autoritativo del dominio. Caché en resolver/SO (TTL).
- Tipos de registro: **A** (IPv4), **AAAA** (IPv6), **CNAME** (alias), **MX** (correo), **PTR** (inversa), **NS** (servidores del dominio), **TXT** (verificación/SPF), **SOA** (autoridad).
- **Puerto 53/UDP** (consultas) y **53/TCP** (respuestas grandes, zone transfer).
- En hosts: `nslookup nombre`, `dig nombre`. Si DNS falla: la web "no carga" aunque haya IP (clásico).
- Conceptos: iterativo (el resolver salta de root→TLD→autoritativo) vs recursivo (el resolver hace todo el trabajo al cliente); caché y TTL; DNSSEC (firmas, mención fuera de alcance).

**DHCP** (Dynamic Host Configuration Protocol, RFC 2131/3315 para DHCPv6):
- Asigna IP, máscara, gateway, DNS automáticamente a clientes.
- Proceso **DORA**:
  1. **Discover** (cliente, broadcast): "¿hay servidor DHCP?"
  2. **Offer** (servidor): "te ofrezco 192.168.1.100/24, gateway .1, DNS .10, lease 24h".
  3. **Request** (cliente, broadcast): "acepto la oferta del servidor X" (informa a otros servidores que declinen).
  4. **Ack** (servidor): "confirmado, es tuya por el lease".
- **Lease**: concesión temporal (horas-días); el cliente renueva en T1 (50%)/T2 (87.5%) del lease; si expira sin renovar, vuelve a DORA.
- Tipos: manual (MAC→IP fija), automática (fija pero entre rango), dinámica (pool + lease). **Reserva DHCP** = IP fija por MAC con toda la config automática.
- En Cisco: `ip dhcp pool LAN` + `network`, `default-router`, `dns-server`, `excluded-address` (parte 2); en hosts, `ipconfig /release` y `/renew`.
- DHCPv6: con o sin estado (con RA flags O/M; ver sección 13); DUID identifica al cliente; rapid commit.

### 16.7 Preguntas del Módulo
Quiz de autoevaluación.

## Ejemplos

### Ejemplo 1 — Navegación HTTPS completa en capas
1. DNS resuelve `www.cisco.com` → 23.x.x.x (UDP/53) — aplicación.
2. TCP handshake al 443 — transporte.
3. TLS handshake: certificados, cifrado simétrico negociado — presentación.
4. HTTP `GET /` — aplicación; Sesión: la propia sesión TLS/HTTP; respuesta comprimida (gzip) — presentación.

### Ejemplo 2 — Diagnóstico: web no carga, ping por IP sí
`ping 8.8.8.8` OK · `ping www.google.com` falla → problema **DNS**: `nslookup www.google.com` muestra timeout → revisar servidor DNS configurado (`ipconfig /all`), probar con 8.8.8.8 como DNS alternativo o `ipconfig /flushdns`.

### Ejemplo 3 — DORA en Packet Tracer
PC sin IP → link en modo Simulation: paquete **DHCP Discover** broadcast → **Offer** del servidor (router/DHCP) → **Request** broadcast → **Ack**. Resultado en `ipconfig /all`: IP + lease. Si falta el servidor: APIPA `169.254.x.x`.

### Ejemplo 4 — Cliente de correo corporativo
Diego configura Outlook: envío `smtp.empresa.com:587` (TLS) y recepción `imap.empresa.com:993` (TLS) con autenticación por contraseña/token; SPF/DKIM evitan que sus correos vayan a spam; móvil Sync vía IMAP mantiene carpetas/enviados sincronizados multi-dispositivo (vs POP3 que baja y borra).

## Ejercicios

1. **(Recordar)** ¿En qué capas OSI están HTTP, TLS y TCP? Asigna cada una.
2. **(Recordar)** Enumera los 4 pasos de DORA y el puerto de DHCP servidor.
3. **(Aplicar)** Empareja: Puerto ↔ servicio: 80, 443, 25, 110, 143, 21, 22, 53, 67, 69, 3389.
4. **(Aplicar)** Flujo de correo: describe servidores y puertos desde que Ana pulsa "Enviar" hasta que Bruno lo lee en 3 dispositivos.
5. **(Comprender)** ¿Por qué HTTPS cifra en capa 6 y no en capa 7? ¿Qué protege y qué no?
6. **(Analizar)** Un usuario descarga a 10 MB/s de 50 semillas BitTorrent frente a 1 MB/s de un solo servidor: ¿qué arquitectura lo explica y qué inconvenientes tiene?
7. **(Analizar)** "La IP es válida pero no cargan webs ni correo; otras PCs sí cargan". Diagnóstico paso a paso con comandos.
8. **(Comprender)** Diferencia entre ARP, DNS y DHCP: qué traduce cada uno y a qué capa pertenecen.

<details>
<summary>Soluciones</summary>

1. HTTP → capa 7 (aplicación) · TLS → capa 6 (presentación) · TCP → capa 4 (transporte).
2. Discover → Offer → Request → Ack; servidor DHCP escucha en UDP/67, cliente envía desde UDP/68.
3. 80 HTTP · 443 HTTPS · 25 SMTP · 110 POP3 · 143 IMAP · 21 FTP-control · 22 SSH/SFTP · 53 DNS · 67 DHCP (servidor) · 69 TFTP · 3389 RDP.
4. Ana → SMTP sumisión (587, TLS) → servidor SMTP saliente → DNS MX del dominio de Bruno (53) → SMTP entre servidores (25) → maildir → Bruno lee vía IMAP 993 en PC y móvil (sincronizado) y/o POP3 110 (descarga y borra en un solo equipo).
5. TLS negocia claves y cifra el flujo HTTP: es asunto de formato/cifrado de datos (presentación). Protege contenido en tránsito, integridad y autenticación del servidor (certificado). No protege: endpoints comprometidos, metadatos (IP/puerto destino, tamaño, timing parcialmente) ni el contenido en cliente/servidor.
6. **P2P**: cada semilla sube fragmentos; más pares = más ancho de banda agregado. Inconvenientes: seguridad distribuida difícil, sin SLA, control de contenido imposible central, problemas legales, imprevisibilidad del rendimiento.
7. `ipconfig /all` → ¿IP estática o DHCP (¿169.254?)? `ping 127.0.0.1` → `ping gateway` → `ping 8.8.8.8` → `ping www.google.com` y `nslookup`: si los tres primeros OK y nslookup falla → DNS; probar `ipconfig /flushdns`, cambiar DNS a público, `ipconfig /release`+`/renew`.
8. **ARP**: IP→MAC, capa 2/3 límite (operación de enlace). **DNS**: nombre→IP, capa 7 (aplicación, sobre UDP/TCP 53). **DHCP**: asigna config IP automáticamente (IP/máscara/gateway/DNS), capa 7 (sobre UDP 67/68).
</details>

## Tips
- HTTPS = HTTP + TLS (443); SFTP = SSH (22); FTPS = FTP+TLS; no mezcles siglas.
- SMTP se usa en ambos sentidos: sumisión 587 (cliente→servidor) y servidor→servidor 25.
- DNS = UDP 53 (TCP para grandes); el examen pregunta "¿qué consultar para verificar resolución?" → `nslookup`.
- DORA: el broadcast del Discover garantiza que cualquier servidor de la subred lo vea.
- Los códigos HTTP 2xx/3xx/4xx/5xx caen en el examen.
- APIPA 169.254 = DHCP inalcanzable: `ipconfig /release`+`/renew` o arregla el servidor DHCP/red.

## Datos curiosos
- El primer correo electrónico (Ray Tomlinson, 1971) se envió entre dos PDP-10 vecinas y eligió la **@** para separar nombre de máquina de persona: aún sobrevivimos con ella.
- **ARPANET** funcionaba sin DNS: existía un fichero HOSTS.TXT mantenido a mano en el NIC; el crecimiento lo hizo ingerible: en 1983 Paul Mockapetris inventó el **DNS** (RFCs 882/883).
- El primer dominio registrado jamás fue **symbolics.com** (15 de marzo de 1985); le siguieron bbn.com, think.com, mcc.com.
- HTTP/1.0 (1996) exigía una conexión TCP por petición; HTTP/1.1 (1997) trajo keep-alive, y HTTP/2 (2015) multiplexó streams; HTTP/3 (2022, RFC 9114) usa QUIC sobre UDP para eliminar el head-of-line blocking.

## Para tu libreta de apuntes
- Capa 5 sesión (diálogos) · capa 6 presentación (formato/cifrado/compresión) · capa 7 aplicación (protocolos usuario).
- CS vs P2P: servidor central vs pares igualitarios.
- Web: HTTP 80 · HTTPS 443 (TLS). Correo: SMTP 25/587 · POP3 110/995 · IMAP 143/993. Archivos: FTP 20/21 · SFTP 22 · TFTP 69 UDP.
- DNS: 53 UDP · registros A/AAAA/MX/CNAME/PTR/TXT · nslookup.
- DHCP: 67/68 UDP · DORA (Discover, Offer, Request, Ack) · lease + T1/T2 · 169.254=APIPA.

## Preguntas de autoevaluación

1. ¿Qué capa OSI cifra los datos en HTTPS? **Respuesta:** Capa 6 (presentación, con TLS). (En TCP/IP: aplicación.)
2. Ordena los pasos DORA. **Respuesta:** Discover → Offer → Request → Ack.
3. ¿Qué protocolo y puerto se usa para recibir correo manteniendo sincronización entre dispositivos? **Respuesta:** IMAP, puerto 143 (993 con TLS).
4. Un PC muestra 169.254.1.33: ¿qué falló y qué revisas? **Respuesta:** No obtuvo DHCP (APIPA); revisar conectividad, servidor DHCP encendido/pool, cables/switch.
5. Diferencia entre HTTP y HTTPS (protocolo, puerto, seguridad). **Respuesta:** HTTP = texto plano en 80; HTTPS = HTTP sobre TLS en 443, cifrado e integridad, con certificado de servidor.
