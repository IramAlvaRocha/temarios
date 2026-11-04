# 15 — Capa 4: Capa de Transporte

> Duración del curso: 39m · 6 clases
> Fuente: temario.md del curso

## Objetivos de aprendizaje
- Explicar la función de la capa 4: segmentación, multiplexación por puertos y control de flujo/errores.
- Comparar TCP y UDP: tabla completa (conexión, fiabilidad, orden, velocidad, cabeceras, usos, puertos típicos).
- Identificar puertos bien conocidos, registrados y dinámicos.
- Describir la comunicación TCP (3-way handshake, ventana, acuses, cierre) y la UDP (best effort).

## Teoría

### 15.1 Transporte de los Datos (Clase 15.1)
La **capa de transporte** (OSI 4) proporciona entrega **extremo a extremo** entre aplicaciones de hosts distintos (la red IP solo entrega entre hosts, no entre procesos). Funciones:
- **Segmentación y reensamblado:** divide los datos de aplicación en segmentos y los reconstruye en destino.
- **Multiplexación/demultiplexación:** con los **puertos**, un host mantiene muchas conversaciones a la vez (navegar + correo + chat).
- **Control de flujo:** el receptor marca cuánto puede recibir (ventana TCP).
- **Control de errores:** detección (checksum) + retransmisión (TCP).
- **Establecimiento y cierre de conexiones:** TCP orientado a conexión.
- **Fiabilidad (opcional):** TCP garantiza orden y entrega; UDP no.

### 15.2 Introducción a TCP y UDP (Clase 15.2)
**TCP** (Transmission Control Protocol, RFC 793 → RFC 9293):
- Orientado a conexión (handshake), fiable: números de secuencia, ACKs, retransmisión por timeout/3 ACKs duplicados, orden garantizado, control de congestión y de flujo.
- Cabecera: mínimo **20 B** (hasta 60 con opciones): puertos src/dst, seq, ack, flags (SYN, ACK, FIN, RST, PSH, URG), ventana, checksum, urgente.

**UDP** (User Datagram Protocol, RFC 768):
- Sin conexión, sin fiabilidad, sin orden garantizado, sin control de congestión: envía y olvida.
- Cabecera: **8 B** fijos: puertos src/dst, longitud, checksum.
- Ventaja: latencia y overhead mínimos; útil en streaming, voz/vídeo (RTP), DNS, DHCP, SNMP, juegos, VPNs.

**Tabla comparativa (pregunta estrella del examen):**

| Característica | TCP | UDP |
|---|---|---|
| Conexión | Orientado a conexión | Sin conexión |
| Fiabilidad | Garantizada (ACK/retransmisión) | No |
| Orden | Ordenado (seq numbers) | No garantizado |
| Control de flujo/congestión | Sí (ventana/AIMD) | No |
| Cabecera | ≥ 20 B | 8 B |
| Velocidad/overhead | Mayor overhead | Mínimo |
| Uso típico | HTTP(S), FTP, SSH, SMTP, IMAP | DNS, DHCP, VoIP/RTP, streaming, SNMP, TFTP |
| Multiplexación | Puertos | Puertos |
| MTU/PMTUD | Ajusta MSS con PMTUD | Debe respetar MTU (o IP fragmenta) |

### 15.3 Puertos de Red (Clase 15.3)
Puerto = identificador de 16 bits (0–65535) de una **aplicación/proceso**:
- **Bien conocidos (well-known), 0–1023**: asignados por IANA (HTTP 80, etc.).
- **Registrados (registered/user), 1024–49151**: asignados a aplicaciones concretas bajo registro IANA (1114 mSQL, 3306 MySQL, 3389 RDP, 8080 HTTP-alt).
- **Dinámicos/privados (ephemeral), 49152–65535** (Windows; Linux típico 32768+): origen de los clientes.

**Puertos que el CCNA exige saber:**

| Puerto | Protocolo | Servicio |
|--------|-----------|----------|
| 20/21 | TCP | FTP (datos/control) |
| 22 | TCP | SSH (y SFTP sobre SSH) |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 67/68 | UDP | DHCP (servidor/cliente) |
| 69 | UDP | TFTP |
| 80 | TCP | HTTP |
| 110 | TCP | POP3 |
| 123 | UDP | NTP |
| 143 | TCP | IMAP |
| 161/162 | UDP | SNMP (agente/trap) |
| 443 | TCP | HTTPS |
| 3389 | TCP | RDP |
| 514 | UDP | Syslog |

Formato de tupla de conexión TCP: `(src IP, src port, dst IP, dst port)`. Un servidor escucha en puerto fijo; los clientes usan ephemeral. Firewalls filtran por puerto/protocolo.

### 15.4 Comunicación TCP (Clase 15.4)
**3-way handshake (apertura):**
1. Cliente → `SYN` (seq=x): "quiero abrir conexión".
2. Servidor → `SYN+ACK` (seq=y, ack=x+1): "acepto y te propongo mis términos".
3. Cliente → `ACK` (ack=y+1): "confirmado". A partir de aquí fluyen datos.

**Transferencia fiable:** segmentos numerados (seq), receptor confirma con ACK acumulativo; si falta un segmento (timeout o 3 ACKs duplicados), se retransmite. **Ventana (window)**: cuántos bytes puede enviar el emisor sin ACK (control de flujo). El **checksum** detecta corrupción.

**Cierre (4 pasos):** `FIN` → `ACK` (del otro lado) → `FIN` → `ACK`. También existe cierre abrupto con `RST` (rechazo/reset).

**Estados** (resumen): LISTEN, SYN-SENT, SYN-RECEIVED, ESTABLISHED, FIN-WAIT, CLOSE-WAIT, TIME-WAIT, CLOSED.

### 15.5 Comunicación UDP (Clase 15.5)
- Sin handshake: envía datagrama y listo. Sin ACKs, sin retransmisión, sin orden.
- Ventajas: mínima latencia y overhead; ideal cuando perder es mejor que esperar (voz en tiempo real; streaming; consulta DNS puntual).
- La fiabilidad, si se necesita, se implementa **en la aplicación** (p. ej., QUIC sobre UDP: HTTP/3; SIP/RTP con sus propios mecanismos parciales).
- Riesgos: puede inundar la red sin control de congestión; se usa donde la pérdida es tolerable o recuperable por niveles superiores.

### 15.6 Preguntas del Módulo
Quiz de autoevaluación.

## Ejemplos

### Ejemplo 1 — Captura mental del handshake
PC (52341) → servidor (443):
```
PC → SYN seq=1000
Servidor → SYN-ACK seq=9000 ack=1001
PC → ACK ack=9001
PC → GET /index.html (PSH+ACK)
```
Cierre: `FIN,ACK` / `ACK` / `FIN,ACK` / `ACK`.

### Ejemplo 2 — Misma web por TCP y DNS por UDP
- Navegación HTTPS: TCP 52341→443; sesiones largas, orden y retransmisión.
- Resolución de nombres: `DNS query UDP 53210→53`; si no hay respuesta en X s, el cliente **repite la consulta** (timeout de aplicación). Respuesta típica <100 B cabe en un datagrama UDP.

### Ejemplo 3 — Multiplexación con netstat
```
C:\> netstat -an | find "443"
  TCP    192.168.1.10:52341    142.250.184.110:443    ESTABLISHED
```
`netstat` / `ss -tulpn` (Linux) lista puertos y conexiones: útil para ver qué app usa qué puerto.

### Ejemplo 4 — Streaming y pérdida
Videollamada (RTP/UDP): si se pierde un datagrama, el jugador muestra un micro-corte y sigue; con TCP la retransmisión llega tarde y desincronizaría el audio. Con DNS-over-HTTPS/QUIC (UDP 443) la fiabilidad la pone QUIC, no UDP.

## Ejercicios

1. **(Recordar)** Enumera 3 diferencias TCP vs UDP.
2. **(Recordar)** ¿De qué tamaño mínimo es la cabecera UDP y qué campos lleva?
3. **(Aplicar)** Asigna el puerto: SSH, HTTPS, DNS, DHCP servidor, FTP control, NTP, RDP.
4. **(Aplicar)** Dibuja el 3-way handshake y el cierre en 4 pasos con flags y seq/ack inventados.
5. **(Comprender)** ¿Por qué DNS usa UDP y HTTP usa TCP?
6. **(Analizar)** Un cliente abre 5.000 conexiones simultáneas: ¿cuántos puertos ephemeral consume y qué tuple las discrimina?
7. **(Analizar)** ¿Qué pasa si un segmento TCP se pierde? Describe dos mecanismos de detección/recuperación.
8. **(Comprender)** ¿Qué añade HTTP/3 (QUIC) a UDP para poder usarlo como transporte web fiable?

<details>
<summary>Soluciones</summary>

1. Conexión (sí/no), fiabilidad (ACK/retrans vs no), orden (sí/no), overhead (20+ vs 8 B), control de flujo (ventana sí/no).
2. 8 bytes: puerto origen (2), puerto destino (2), longitud (2), checksum (2).
3. SSH 22/TCP · HTTPS 443/TCP · DNS 53 UDP+TCP · DHCP servidor 67/UDP · FTP control 21/TCP · NTP 123/UDP · RDP 3389/TCP.
4. Handshake: SYN(seq=x) → SYN-ACK(seq=y, ack=x+1) → ACK(ack=y+1). Cierre: FIN → ACK → FIN → ACK. (Ver ejemplo 1.)
5. DNS: consultas cortas, respuesta pequeña, reintento barato; latencia importa → UDP. HTTP: documentos largos, orden y fiabilidad imprescindibles → TCP.
6. 5.000 puertos ephemeral (uno por conexión); los discrimina la tupla (IP local, puerto local, IP remota, puerto remoto); el servidor los distingue por esa combinación aunque todos vayan a su mismo puerto 443.
7. (a) **Timeout RTO**: si no llega ACK a tiempo, retransmite el segmento. (b) **3 ACKs duplicados** (fast retransmit): retransmite inmediatamente sin esperar timeout; luego fast recovery/AIMD reduce la ventana.
8. QUIC implementa sobre UDP: establecimiento de conexión cifrado (TLS 1.3 integrado), numeración de paquetes propia, ACKs y retransmisión, control de congestión, multiplexación de streams y cabeceras reducidas — fiabilidad sin handshake TCP adicional y sin head-of-line blocking entre streams.
</details>

## Tips
- Puertos imprescindibles: 20/21, 22, 23, 25, 53, 67/68, 69, 80, 110, 123, 143, 161/162, 443.
- TCP flags del handshake y cierre: SYN, SYN-ACK, ACK, FIN, RST.
- "TCP = teléfono: descolgar, hablar, colgar. UDP = carta: echas y no sabes si llega."
- Examen: "¿Qué protocolo usa DNS?" → **UDP/53 por defecto, TCP/53 para respuestas grandes y zone transfers**.
- FTP activo vs pasivo: 20 datos/21 control (pasivo es el usado hoy por NATs). SFTP ≠ FTPS: SFTP va sobre SSH (22).
- Navega siempre cifrado: 443 = HTTPS; 80 en claro.

## Datos curiosos
- **443/TCP (HTTPS)** nació con SSL en la era de Netscape (mediados de los 90); hoy el cifrado lo provee TLS 1.3 (RFC 8446), estándar IETF vivo.
- Los números de puerto son de **16 bits** (0–65535), límite histórico de los protocolos de transporte de la época de TCP/IP; aún así basta para la mayoría de aplicaciones.
- El 3-way handshake evita que conexiones "fantasma" de mensajes antiguos (reenvíos de red) abran sesiones falsas: el ISN (Initial Sequence Number) aleatorio hace inviable inyectar segmentos válidos sin él.
- UDP no tiene control de congestión por diseño: en 1986 y 1988 el colapso de Internet por congestión llevó a Jacobson y a Karn a definir el control de congestión de TCP de forma urgente.

## Para tu libreta de apuntes
- Capa 4 = extremo a extremo: puertos = procesos. Tupla TCP (IP:puerto x4).
- TCP: handshake SYN→SYN-ACK→ACK · seq/ACK · ventana · cierre FIN/ACK×2 · fiable.
- UDP: sin conexión · 8 B · best effort · bajo overhead.
- Bien conocidos ≤1023 · registrados 1024–49151 · dinámicos ≥49152.
- Puertos: 21/22/23/25/53/67-68/69/80/110/123/143/161/443/3389.

## Preguntas de autoevaluación

1. ¿Cuántos bytes tiene la cabecera UDP? **Respuesta:** 8 bytes.
2. Escribe el 3-way handshake completo. **Respuesta:** SYN → SYN-ACK → ACK.
3. Un servicio web cifrado escucha por defecto en el puerto… **Respuesta:** 443/TCP (HTTPS).
4. ¿Qué mecanismo TCP evita que un emisor rápido desborde al receptor? **Respuesta:** El control de flujo mediante el campo ventana (advertised window).
5. Nombra 3 protocolos/aplicaciones que usan UDP y explica por qué. **Respuesta:** DNS (consultas cortas), DHCP (broadcast sin conexión), VoIP/RTP o streaming (latencia > fiabilidad), TFTP (simplicidad), SNMP (consultas).
