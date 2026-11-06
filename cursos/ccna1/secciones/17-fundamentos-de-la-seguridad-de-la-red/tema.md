# 17 — Fundamentos de la Seguridad de la Red

> Duración del curso: 1h 18m · 4 clases
> Fuente: temario.md del curso

## Objetivos de aprendizaje
- Identificar las amenazas y ataques más frecuentes contra redes (malware, DoS/DDoS, MITM, ingeniería social).
- Describir los mecanismos de mitigación: políticas, segmentación, controles de acceso, actualizaciones.
- Aplicar medidas de seguridad básicas en dispositivos de red: contraseñas robustas, SSH en lugar de Telnet, banners, puertos físicos, gestión fuera de banda.
- Valorar el principio de defensa en profundidad (múltiples capas de control).

## Teoría

### 17.1 Amenazas de Seguridad y Ataques de la Red (Clase 17.1)
**Clasificación por objetivo:**
- **Confidencialidad:** robo de datos (sniffing, escucha en medios compartidos, contraseñas débiles).
- **Integridad:** alteración de datos en tránsito/reposo (MITM, inyección).
- **Disponibilidad:** hacer inaccesible el servicio (DoS/DDoS, ransomware que cifra datos).

**Amenazas que caen en el CCNA:**
- **Malware:** virus (se replican con acción humana), **gusanos** (worm, propagación automática por red: Slammer, Conficker), **troyanos** (disfrazados de software legítimo), **ransomware** (cifra y pide rescate: WannaCry, 2017 — explotó SMBv1 con ETERNALBLUE).
- **Ingeniería social:** phishing (correos falsos), spear phishing (dirigido), vishing (voz), smishing (SMS), tailgating/piggybacking (entrar físicamente detrás de alguien), pretexting.
- **Ataques de red:**
  - **DoS/DDoS:** saturar recursos con tráfico masivo o peticiones (SYN flood → agota la tabla de conexiones; amplificación DNS/NTP; botnets).
  - **Man-in-the-middle (MITM):** interceptar la comunicación (ARP spoofing en LAN, DNS poisoning, evil twin Wi‑Fi).
  - **Password attacks:** fuerza bruta, diccionario, credential stuffing (reutilizar credenciales filtradas).
  - **Inyección/explotación:** SQL injection, XSS, buffer overflow, explotación de sistemas sin parchear.
  - **MAC/port attacks:** MAC flooding (desbordar tabla CAM → sniffing), DHCP spoofing (servidor falso → gateway falso).
  - **Sniffing:** capturar tráfico en segmentos (hubs, SPAN, Wi‑Fi abierto).
- **Amenazas internas:** empleados descuidados o maliciosos (ver CIA: la gente es el vector nº 1).
- **Amenazas físicas:** acceso al CPD, USBs maliciosos, consolas sin contraseña.

### 17.2 Mitigar los Ataques a la Red (Clase 17.2)
**Defensa en profundidad** = capas de controles; ningún control único es suficiente:
- **Políticas de seguridad:** uso aceptable, contraseñas, BYOD, respuesta a incidentes; lo que no está escrito no se cumple.
- **Segmentación:** VLANs + ACLs; separar invitados/IoT/servidores; microsegmentación.
- **Control de acceso:** autenticación fuerte (contraseñas robustas, MFA), "mínimo privilegio" y necesidad de conocer (need-to-know); NAC (802.1X) para admitir solo dispositivos autorizados.
- **Port security (switches):** limitar MACs por puerto, sticky, violación shutdown/restrict (fuera de este curso, parte 2).
- **DHCP snooping + Dynamic ARP Inspection:** frenar DHCP/ARP spoofing.
- **Firewall:** filtrado de paquetes/estado (NAT, ACLs, NGFW con DPI); ubicado en el perímetro.
- **IDS/IPS:** detección (IDS, alerta) y prevención (IPS, bloquea inline) de intrusiones.
- **Antivirus/EDR** endpoints; **parches/actualizaciones** (la mayoría de brechas explotan vulnerabilidades con parche disponible).
- **Cifrado:** VPN (IPsec/SSL) para acceso remoto y sitios interconectados; TLS para web/correo; SSH para gestión.
- **Copias de seguridad** (3-2-1) y plan de respuesta a incidentes: el ransomware se paga menos cuando puedes restaurar.
- **Monitorización y logs:** Syslog centralizado, NetFlow, SIEM; detección temprana.
- **Formación de usuarios:** anti-phishing, reporte de incidentes — la mayoría de brechas involucran al elemento humano (Verizon DBIR); el phishing es el vector inicial más común.

### 17.3 Seguridad de los Dispositivos (Clase 17.3)
Medidas concretas en routers/switches Cisco (útiles para labs y examen):

1. **Contraseñas robustas** (longitud > complejidad; frases de paso; únicas por equipo) y **nunca en texto plano**: `enable secret` (hash) > `enable password`; `service password-encryption` para el resto (ofuscación).
2. **SSH, no Telnet**: Telnet envía credenciales en claro (sniffing trivial). Configuración mínima:
```
R1(config)# ip domain-name empresa.local
R1(config)# crypto key generate rsa modulus 2048
R1(config)# username admin secret claveFuerte9!
R1(config)# line vty 0 4
R1(config-line)# transport input ssh
R1(config-line)# login local
R1(config-line)# exit
R1(config)# ip ssh version 2
R1(config)# end
R1# copy running-config startup-config
```
3. **Acceso físico**: el puerto Console da control total del equipo → racks cerrados/con llave, acceso solo para personal autorizado; desactivar puertos de switch sin usar (`shutdown`), etiquetado e inventario.
4. **Banner** legal (`banner motd`) y sin información de sistema (nada de versión/modelo).
5. **Mínimo privilegio y AAA** (concepto): autenticación (¿quién eres?), autorización (¿qué puedes?), accounting (¿qué hiciste?) — con servidor RADIUS/TACACS+; en labs, `enable secret` local.
6. **Desactivar servicios innecesarios**: HTTP server (`no ip http server`, sí `ip http secure-server` si se usa), CDP en puertos no confiados, paquetes small-servers.
7. **Gestión fuera de banda (OOB)**: red dedicada de administración separada de la red de usuarios.
8. **Copias de configuración**: `copy run start` + copias externas; versionado.
9. **Tiempo sincronizado** (NTP) y **logs** (`logging buffered`, `logging host <ip>`) para correlación de eventos.
10. **Actualizar IOS** ante boletines de seguridad de Cisco (PSIRT) y cambiar credenciales por defecto (¡siempre!).

### 17.4 Preguntas del Módulo
Quiz de autoevaluación.

## Ejemplos

### Ejemplo 1 — Due diligence de un router recién sacado de la caja
1. `show version` → comprueba versión de IOS contra boletines Cisco.
2. Cambia todas las contraseñas por defecto; `enable secret` fuerte.
3. `no ip domain-lookup` (operación), `crypto key generate rsa`, vty solo SSH, `transport input ssh`.
4. `no ip http server`; `service password-encryption`; banner legal.
5. Cierra puertos sin uso; guarda en startup-config; verifica con `show running-config`.

### Ejemplo 2 — Simulación de incidente: phishing + ransomware
Un empleado abre un adjunto → macro → descarga → cifra la unidad de red en minutos. Respuesta: aislar el segmento (shutdown de puertos/VLAN), restaurar desde backup (3-2-1), forzar cambio de credenciales, análisis de logs (Syslog/NetFlow) para alcance, revisar parches SMBv1/RDP, formación al empleado y notificación según política. Lección: el vector inicial fue humano; la contención, segmentación + backups.

### Ejemplo 3 — SSH en lugar de Telnet (demo de "sniffing")
En Packet Tracer/Wireshark: con Telnet, el login (usuario/clave) aparece en claro en la captura. Repítelo con SSH: el tráfico va cifrado e ilegible. Conclusión directa: gestión de red **siempre** por SSH/HTTPS.

### Ejemplo 4 — Política mínima viable de contraseñas (empresa pequeña)
≥ 12 caracteres, mezcla, únicas, gestor de contraseñas corporativo, MFA en correo/VPN/cloud, caducidad solo tras incidente (NIST 800-63 desaconseja caducidad periódica forzosa sin evidencia), bloqueo tras N intentos, aviso de credenciales filtradas (haveibeenpwned).

## Ejercicios

1. **(Recordar)** Define confidentiality, integrity y availability con un ejemplo de ataque a cada una.
2. **(Recordar)** Diferencia virus, gusano y troyano.
3. **(Aplicar)** Escribe los comandos para habilitar SSH v2 en R1: dominio, clave RSA 2048, usuario local y vty restringidas a SSH.
4. **(Aplicar)** Clasifica como amenaza: (a) SYN flood, (b) phishing, (c) ARP spoofing, (d) WannaCry, (e) tailgating, (f) contraseña "1234".
5. **(Comprender)** ¿Qué es la defensa en profundidad? Da 4 capas concretas para proteger un servidor web interno.
6. **(Analizar)** Un atacante intercepta el tráfico entre PC y switch usando MAC flooding. Explica el proceso y 2 mitigaciones.
7. **(Analizar)** ¿Por qué bloquear todo el ICMP es mala idea? (Enlace con la sección 14.)
8. **(Aplicar)** Redacta la política de seguridad en 10 líneas para una oficina de 20 personas con Wi‑Fi de invitados y 2 servidores.

<details>
<summary>Soluciones</summary>

1. Confidencialidad: solo acceden los autorizados (ataque: sniffing/robo de credenciales). Integridad: los datos no se alteran sin autorización (ataque: MITM modificando transacción bancaria). Disponibilidad: el servicio está accesible (ataque: DDoS, ransomware).
2. Virus: requiere acción humana (abrir/ejecutar); gusano: se propaga solo por la red; troyano: se disfraza de legítimo y abre la puerta (backdoor).
3. `ip domain-name empresa.local` → `crypto key generate rsa modulus 2048` → `username admin secret ...` → `line vty 0 4` → `transport input ssh` → `login local` → `exit` → `ip ssh version 2`.
4. (a) DDoS/disponibilidad · (b) ingeniería social/phishing · (c) MITM/capa 2 (integridad-confidencialidad) · (d) ransomware/malware · (e) ingeniería social física · (f) contraseña débil (vulnerabilidad/autenticación).
5. Múltiples controles independientes y solapados. Ejemplo servidor web: (1) segmentación VLAN + ACL, (2) firewall/NGFW con WAF, (3) parches + hardening del SO (mínimos servicios), (4) TLS + autenticación + monitorización (logs/SIEM/IDS). (Añadir backup y respuesta a incidentes.)
6. Envía miles de MACs falsas → llena la CAM del switch → el switch inunda como hub → el atacante captura tráfico ajeno con un sniffer. Mitigaciones: **port-security** (máximo de MACs por puerto + sticky/shutdown) y **802.1X/NAC** (solo puertos autenticados transmiten). Además DHCP snooping/DAI contra ataques relacionados y segmentación.
7. Porque rompe PMTUD (3/4) — HTTPS se cuelga — y diagnóstico (ping/traceroute) incluyendo monitorización de red. Filtrar por tipos (permitir 8/0 eco y 3/4 PMTUD) en lugar de "todo ICMP fuera".
8. Ejemplo: política de contraseñas robustas + MFA en correo/VPN · mínimos servicios en servidores, parches mensuales · VLAN separada para invitados sin acceso a LAN · WPA2/WPA3-Enterprise o PSK robusto con rotación · firewall con solo puertos necesarios (443/25/993) · backups 3-2-1 y restauración probada trimestralmente · antivirus/EDR en endpoints · logs centralizados y revisión semanal · formación anti-phishing y canal de reporte · gestión por SSH con cambio de credenciales por defecto · acceso físico al rack restringido y USBs controlados.
</details>

## Tips
- CIA = Confidencialidad, Integridad, Disponibilidad. Memorízalo.
- Telnet = texto plano = JAMÁS en producción; SSH v2 o nada.
- "Parchea antes que hackean": la mayoría de brechas usan CVEs con parche disponible (EternalBlue tenía parche 2 meses antes de WannaCry).
- Examen: "¿Qué ataque desborda la tabla MAC del switch?" → **MAC flooding**; "servidor DHCP falso" → **DHCP spoofing**; "ARP con MACs ajenas" → **ARP spoofing/poisoning**.
- La defensa en profundidad es la respuesta a casi cualquier pregunta de "¿cómo proteger X?": varias capas.
- Nunca hagas pentesting fuera de entornos autorizados (legalidad; y tu laboratorio es Packet Tracer/GNS3).

## Datos curiosos
- **WannaCry** (12 de mayo de 2017) infectó ~230.000 equipos en 150 países en un solo día explotando SMBv1 (ETERNALBLUE, filtrado de la NSA); Boeing, Telefónica y el NHS británico estuvieron entre las víctimas. El kill-switch accidental de Marcus Hutchins (un dominio sin registrar) detuvo la propagación.
- El ataque **Morris Worm** (1988, 1.º gusano famoso de Internet) tiró ~10% de la red incipiente; su autor casi es detenido; hoy el término "virus de red" no existía, se le llamó "gusano".
- Según los informes Verizon DBIR sucesivos, la mayoría de brechas involucran al elemento humano y el phishing es el vector inicial más común; la seguridad humana (formación + MFA) es el control más coste-efectivo que existe.
- Cisco publica sus boletines de seguridad en **PSIRT** (Cisco Security Advisories); los administradores la revisan como parte del ciclo de mantenimiento.

## Para tu libreta de apuntes
- CIA: Confidencialidad · Integridad · Disponibilidad.
- Amenazas: malware (virus/worm/troyano/ransomware) · DoS/DDoS · MITM (ARP spoof) · social (phishing) · password · MAC flooding · sniffing.
- Mitigar: segmentar (VLAN/ACL) · parchear · firewalls/IDS/IPS · VPN/TLS/SSH · backups 3-2-1 · logs/SIEM · formación.
- Dispositivo: SSH no Telnet · enable secret · no http server · puertos cerrados · OOB · NTP · Syslog.

## Preguntas de autoevaluación

1. Nombra los tres principios de seguridad de la información. **Respuesta:** Confidencialidad, Integridad, Disponibilidad (CIA).
2. ¿Qué comando evita que la gestión remota de un dispositivo use texto plano? **Respuesta:** `transport input ssh` en las líneas vty (con claves RSA y SSH v2 habilitado).
3. Un ataque que llena la tabla CAM del switch para capturar tráfico ajeno es… **Respuesta:** MAC flooding (mitigación: port-security).
4. ¿Por qué la política de contraseñas exige MFA además de complejidad? **Respuesta:** Una contraseña robada/filtrada no basta para acceder; el segundo factor (token/app/biometría) bloquea el login con credenciales válidas comprometidas.
5. Enumera las capas mínimas de defensa en profundidad para una LAN de oficina. **Respuesta abierta:** segmentación VLAN/ACL, firewall, parches/hardening, antivirus/EDR, autenticación fuerte/MFA, backups, monitorización y formación.
