# 05 — Protocolos de Red

> Duración del curso: 58m · 7 clases
> Fuente: temario.md del curso

## Objetivos de aprendizaje
- Explicar qué es un protocolo y las reglas que definen (formato, timing, codificación, entrega).
- Describir las suites de protocolos (TCP/IP) y los modelos de referencia OSI y TCP/IP, capa por capa.
- Explicar la segmentación y el encapsulamiento de datos (PDUs: datos, segmento, paquete, trama, bits).
- Identificar cómo los datos son direccionados (MAC de capa 2 e IP de capa 3).

## Teoría

### 5.1 Reglas de Protocolo (Clase 5.1)
Un **protocolo** es un conjunto de reglas acordadas que los dispositivos deben cumplir para comunicarse. Todo protocolo define:
- **Codificación (encoding):** cómo se representan los bits en el medio (niveles de tensión, pulsos de luz, ondas de radio) — p. ej., Manchester en 10BASE-T.
  - `100BASE-TX` = 4B/5B + MLT-3 sobre par trenzado.
  - `100BASE-FX` = 4B/5B + NRZI sobre fibra.
- **Formato y encapsulación del mensaje:** estructura del mensaje (cabeceras/campos) para que el receptor lo entienda.
- **Tamaño del mensaje:** límites (p. ej., MTU de 1500 bytes en Ethernet).
- **Sincronización (timing):** velocidad de transmisión (control de flujo) y orden de envío (entrega ordenada: "¿quién habla primero?").
- **Entrega (delivery):** unicast (1→1), multicast (1→grupo), broadcast (1→todos en el dominio).
- **Control de errores:** detección (FCS/CRC en Ethernet), corrección o retransmisión (TCP).

### 5.2 Protocolos de Red (Clase 5.2)
Protocolos según su función:
- **Comunicación:** HTTP/HTTPS, FTP, SSH, Telnet, SMTP, DNS, DHCP, SIP...
- **Gestión de red:** ICMP, SNMP, Syslog, CDP/LLDP.
- **Enrutamiento:** OSPF, EIGRP, BGP, RIP.
- **Transporte:** TCP, UDP.
- **Acceso al medio:** Ethernet (802.3), Wi‑Fi (802.11), PPP.
- **Seguridad:** IPsec, TLS/SSL, 802.1X, SSH.

Propiedades de un protocolo: abierto (estándar público, RFC) o propietario (desarrollado por un fabricante; p. ej., EIGRP era propietario de Cisco hasta 2013).

### 5.3 Suites de Protocolos (Clase 5.3)
Una **suite de protocolos** agrupa los protocolos que trabajan juntos. La dominante es **TCP/IP**:

- Desarrollada por el DoD de EE. UU. (ARPANET), estandarizada vía **RFC** (Request for Comments, documentos de la IETF; p. ej., RFC 791 IPv4, RFC 793 TCP, RFC 768 UDP).
- Incluye: IP (v4/v6), ICMP, TCP, UDP, ARP (sobre Ethernet), DNS, DHCP, HTTP, etc.
- Otros suites históricos: OSI (ISO), AppleTalk (extinto), IPX/SPX (Novell, extinto), NetBEUI/NetBIOS (Microsoft, extinto).

### 5.4 Modelos de Referencia (Clase 5.4)
**Modelo OSI (7 capas)** — referencia conceptual de ISO:

| # | Capa | PDU | Función | Ejemplos |
|---|------|-----|---------|----------|
| 7 | Aplicación | Datos | Interfaz con el software | HTTP, DNS, DHCP, SMTP |
| 6 | Presentación | Datos | Formato, cifrado, compresión | TLS, JPEG, ASCII/Unicode |
| 5 | Sesión | Datos | Establece/gestiona sesiones | NetBIOS, RPC, SQL sessions |
| 4 | Transporte | Segmento (TCP) / Datagrama (UDP) | Entrega extremo a extremo, puertos | TCP, UDP |
| 3 | Red | Paquete | Direccionamiento lógico y routing | IP, ICMP, OSPF |
| 2 | Enlace de datos | Trama | Direccionamiento físico (MAC), acceso al medio | Ethernet, Wi‑Fi, PPP, ARP |
| 1 | Física | Bits | Señales, cableado, conectores | UTP, fibra, radio |

**Modelo TCP/IP (4 capas, modelo IETF práctico):**

| Capa TCP/IP | Capas OSI equivalentes | Protocolos |
|-------------|----------------------|------------|
| Aplicación | 5+6+7 | HTTP, DNS, DHCP, SSH, SMTP |
| Transporte | 4 | TCP, UDP |
| Internet | 3 | IP, ICMP, IPsec, OSPF |
| Acceso a red (Link/Network Access) | 1+2 | Ethernet, Wi‑Fi, ARP, PPP |

Mnemotécnicos (¡memorízalos!):
- OSI de arriba abajo: **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing (Aplicación, Presentación, Sesión, Transporte, Red, Enlace/Data Link, Física).
- De abajo arriba: **P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way.

El examen usa ambos modelos: OSI es la referencia teórica; TCP/IP la implementación real.

### 5.5 Repaso de Segmentación (Clase 5.5)
**Segmentación** = dividir los mensajes grandes en piezas manejables antes de transmitirlos:
- La capa 4 (TCP) corta los datos de aplicación en segmentos y los reensambla en destino.
- Beneficios: multiplexación (varias conversaciones a la vez), intercalado (un mensaje grande no bloquea a los demás), control de errores y retransmisión solo del segmento perdido, mejor uso del enlace.
- Coste: cabeceras extra en cada segmento/paquete/trama (overhead).

### 5.6 Direccionando los Datos (Clase 5.6)
- **Encapsulamiento (de arriba abajo):** Datos → +cabecera TCP/UDP = **segmento** → +cabeceras IP (origen/destino lógicos) = **paquete** → +cabeceras/tráiler Ethernet (MAC origen/destino, FCS) = **trama** → señalización en medio = **bits**.
- **Desencapsulamiento (de abajo arriba):** cada capa quita su cabecera y pasa el resto a la capa superior.
- **Direccionamiento dual:** IP de capa 3 (lógica, extremo a extremo, no cambia en tránsito salvo NAT) y MAC de capa 2 (física, tramo a tramo, cambia en cada salto).
- Escenario: un paquete IP viaja del PC al servidor web pasando por el router; las MACs cambian en cada enlace, la IP origen/destino se mantiene.

PDU por capa (pregunta estrella del examen):
- Capa 7/6/5: **Datos** · Capa 4: **Segmento** (TCP) / **Datagrama** (UDP) · Capa 3: **Paquete** · Capa 2: **Trama** · Capa 1: **Bits**.

### 5.7 Preguntas del Módulo
Quiz de autoevaluación.

## Ejemplos

### Ejemplo 1 — Encapsulamiento de una petición web
`GET http://www.cisco.com/` (datos HTTP):
1. Capa 7/6/5: datos HTTP.
2. Capa 4: cabecera TCP (puertos: origen aleatorio 52341 → destino 80) → *segmento*.
3. Capa 3: cabecera IP (src 192.168.1.10 → dst del servidor, p. ej. 23.x.x.x) → *paquete*.
4. Capa 2: cabecera Ethernet (MAC del PC → MAC del gateway/router) + FCS → *trama*.
5. Capa 1: bits codificados en el cable. En destino, se desencapsula en orden inverso.

### Ejemplo 2 — Multiplexación en un cliente
El PC navega (TCP 52341→80) y descarga por FTP (TCP 52342→21) a la vez. Las cabeceras de capa 4 con puertos distintos permiten al stack reenviar cada flujo a la aplicación correcta: eso es multiplexación por puertos.

### Ejemplo 3 — Segmentación y MTU
Ethernet MTU = 1500 B. Un fichero de 120 KB se divide en ~80 segmentos/paquetes. Si una trama excede el MTU (p. ej. por túneles), IP puede fragmentar el paquete (campo *Identification/Flags/Fragment Offset*); en Ethernet estándar, la capa 4 ajusta el tamaño para no fragmentar (MSS en TCP).

### Ejemplo 4 — Cambian las MACs, no la IP
Trama sale del PC con MAC(PC)→MAC(R1-LAN). R1 la desencapsula, consulta su tabla de enrutamiento, y reenvía con nueva trama MAC(R1-WAN)→MAC(R2). La IP origen/destino nunca cambia (salvo NAT). Prueba: en Packet Tracer, modo Simulation, observa las cabeceras en cada salto.

## Ejercicios

1. **(Recordar)** Enumera las 7 capas OSI y las 4 del modelo TCP/IP.
2. **(Recordar)** ¿Qué PDU corresponde a cada capa OSI 4, 3, 2, 1?
3. **(Comprender)** Explica con tus palabras qué es la segmentación y dos ventajas.
4. **(Aplicar)** Dados los protocolos: HTTP, IP, Ethernet, TCP, DNS, MAC/ARP — asígnalos a su capa OSI y TCP/IP.
5. **(Aplicar)** Un emisor envía datos a un servidor a través de un router. ¿Qué direcciones cambian en cada salto y cuáles se mantienen de extremo a extremo?
6. **(Analizar)** ¿Por qué el modelo OSI sigue enseñándose si el implementado es TCP/IP? Da dos razones.
7. **(Analizar)** ¿En qué capa OSI opera un router? ¿Y un switch? ¿Y un cable de fibra?
8. **(Comprender)** ¿Qué es el FCS y en qué capa se usa?

<details>
<summary>Soluciones</summary>

1. OSI: Aplicación, Presentación, Sesión, Transporte, Red, Enlace de datos, Física. TCP/IP: Aplicación, Transporte, Internet, Acceso a red.
2. Capa 4: segmento (TCP)/datagrama (UDP); capa 3: paquete; capa 2: trama; capa 1: bits.
3. Segmentación: dividir los mensajes en unidades menores para transmitirlos y reensamblarlos. Ventajas: multiplexación de conversaciones, intercalado/no bloqueo, retransmisión selectiva, uso eficiente del enlace.
4. HTTP → L7 OSI / Aplicación TCP-IP; DNS → L7; TCP → L4 / Transporte; IP → L3 / Internet; Ethernet → L2 / Acceso a red; ARP → L2/L3 práctica (mapea IP↔MAC; en modelos se enseña en capa 2/3 límite, en TCP/IP "acceso a red").
5. Cambian: MACs (origen/destino de cada tramo). Se mantienen: IPs origen/destino (extremo a extremo, salvo NAT).
6. OSI es un marco común de referencia universal para diagnosticar ("¿en qué capa está el fallo?") y es vendor-neutral; TCP/IP es la implementación concreta. Además el vocabulario OSI (PDU, capas) se usa en toda la documentación y en el examen.
7. Router → capa 3 (Red). Switch → capa 2 (Enlace de datos; L3 si es multilayer). Fibra/cable → capa 1 (Física).
8. FCS (Frame Check Sequence): campo de 4 bytes con CRC para detectar errores en la trama; lo añade y verifica la capa 2 (Ethernet).
</details>

## Tips
- Memoriza las capas y los PDU a fuego: el examen cae sí o sí. Mnemónico: "**D**atos **S**egmento **P**aquete **T**rama **B**its" de arriba a abajo.
- Orden de encapsulación: App → TCP → IP → Ethernet → bits. Desencapsulación: inverso.
- La capa de sesión/presentación/aplicación OSI colapsan en "Aplicación" en TCP/IP.
- ARP vive en la frontera L2/L3 (resuelve IP→MAC); normalmente se estudia con la capa de enlace/red.
- Broadcast solo L2 (MAC FF:FF:FF:FF:FF:FF) dentro del dominio; no cruza routers. Multicast: un grupo (224.0.0.0/4 en IPv4).

## Datos curiosos
- El modelo OSI (ISO 7498, 1984) perdió la "guerra de protocolos" frente a TCP/IP por la velocidad de despliegue de Arpanet/Internet y porque OSI era complejo e incompleto para routers reales.
- Las **RFC** empezaron en 1969 como notas informales; el RFC 1 lo escribió Steve Crocker y proponía cómo documentar la red. Hoy hay miles de RFCs.
- "The Cathedral and the Bazaar" (Eric Raymond) popularizó la diferencia entre el desarrollo planificado (OSI) y el abierto/emergido (TCP/IP).

## Para tu libreta de apuntes
- Protocolo = reglas: encoding, formato, timing, entrega, errores.
- OSI: 7-Aplicación/Presentación/Sesión · 4-Transporte · 3-Red · 2-Enlace · 1-Física.
- TCP/IP: Aplicación / Transporte / Internet / Acceso a red.
- PDU: Datos → Segmento → Paquete → Trama → Bits.
- Encapsulado ↓ · desencapsulamiento ↑ · MACs cambian salto a salto · IP extremo a extremo.

## Preguntas de autoevaluación

1. ¿Qué PDU genera la capa de red? **Respuesta:** El paquete.
2. ¿Qué dos direcciones cambian en cada salto enrutado y cuál se mantiene? **Respuesta:** Cambian las MAC; se mantiene la IP origen/destino.
3. Nombra las capas del modelo TCP/IP en orden de arriba a abajo. **Respuesta:** Aplicación, Transporte, Internet, Acceso a red.
4. ¿Qué capa OSI se encarga de puertos y entrega extremo a extremo? **Respuesta:** Capa 4 (Transporte).
5. Un mensaje falla y vuelves a enviarlo completo aunque solo se corrompió un segmento: ¿qué mecanismo falta y en qué capa está? **Respuesta:** Control de errores/retransmisión con acuses y numeración de secuencia — capa 4 (TCP).
