# 02 — Conceptos generales

> Duración del curso: 1h 19m · 10 clases
> Fuente: temario.md del curso

## Objetivos de aprendizaje
- Definir qué es una red, sus representaciones, tipos (LAN/WAN/WLAN...) y tipos de conexión a Internet.
- Identificar los cuatro puntos que hacen confiable una red (tolerancia a fallos, escalabilidad, calidad de servicio, seguridad).
- Describir tendencias actuales (BYOD, colaboración en línea, cloud, IoT) y los principios básicos de seguridad de red.
- Convertir números entre binario y decimal y comprender el sistema binario aplicado a IPs.

## Teoría

### 2.1 Qué es una Red? (Clase 2.1)
Una **red** es un conjunto de dispositivos finales (end devices: PCs, teléfonos, servidores, impresoras) y de dispositivos intermedios (switches, routers, APs) interconectados por medios (cobre, fibra, aire) que intercambian datos siguiendo reglas comunes (protocolos).

- **Dispositivo final (end device):** origen o destino del tráfico (PC, laptop, móvil, servidor).
- **Dispositivo intermedio (intermediary device):** interconecta (hub, switch, router, firewall, AP).
- **Medio de red (media):** cobre (UTP), fibra óptica, inalámbrico (Wi‑Fi).

### 2.2 Representaciones de una Red (Clase 2.2)
Dos formas habituales de representar una red:
- **Diagrama físico (physical topology):** muestra ubicación real de equipos, cableado y conexiones.
- **Diagrama lógico (logical topology):** muestra el flujo de datos, direccionamiento IP, VLANs y cómo se comunican los hosts, sin importar la ubicación física.

Iconografía habitual Cisco: router = cilindro con flechas en cruz; switch = rectángulo con flechas bidireccionales; nube = Internet/WAN; PC = monitor; servidor = caja con líneas. En Packet Tracer se usa la misma simbología.

### 2.3 y 2.4 Tipos de Redes (Clases 2.3, 2.4)

| Tipo | Alcance | Ejemplos |
|------|---------|----------|
| **PAN** (Personal Area Network) | Unos metros | Bluetooth, USB |
| **LAN** (Local Area Network) | Edificio/campus pequeño | Red de oficina, casa |
| **WLAN** | LAN por Wi‑Fi | 802.11 |
| **MAN** (Metropolitan Area Network) | Ciudad | Operador metropolitano |
| **WAN** (Wide Area Network) | Países/continentes | Internet, MPLS de operadora |
| **SAN** (Storage Area Network) | Centro de datos | Fibre Channel |

Otras clasificaciones útiles:
- **Intranet:** red privada interna de una organización.
- **Extranet:** acceso controlado a parte de la intranet para socios externos.
- **Internet:** red pública global interconectada.
- **Peer-to-peer (P2P):** los hosts actúan como clientes y servidores a la vez (BitTorrent).
- **Cliente-servidor:** clientes consumen servicios de servidores dedicados.

Criterios de diseño según alcance geográfico y número de nodos: cuanto mayor el área y el número de usuarios, más jerárquico y segmentado debe ser el diseño.

### 2.5 Tipos de Conexiones a Internet (Clase 2.5)

| Tecnología | Medio | Velocidad típica | Notas |
|------------|-------|------------------|-------|
| **Cable (HFC)** | Coaxial | 100 Mbps–1 Gbps+ | Asimétrica, compartida por vecindario (DOCSIS) |
| **DSL** | Par telefónico | 1–100 Mbps | ADSL asimétrico; distancia al nodo limita velocidad |
| **Fibra (FTTH)** | Fibra óptica | 100 Mbps–10 Gbps | GPON/EPON; mejor latencia y simetría |
| **Celular (3G/4G/5G)** | Radio | 10 Mbps–1 Gbps | Movilidad |
| **Satelital** | Radio (GEO/LEO) | 25–150 Mbps | Alta latencia (GEO ~600 ms); LEO tipo Starlink menor |
| **Dial-up** | Telefónica conmutada | ≤ 56 kbps | Histórico |
| **Línea dedicada (leased line)** | Fibra/cobre | Hasta 10+ Gbps | SLA empresarial, CIR garantizado |
| **Metro Ethernet** | Fibra | 10 Mbps–10 Gbps | Ethernet extendido en la ciudad |
| **Wireless ISP (WISP)** | Radio | 10–100+ Mbps | Zonas rurales |

Conceptos: **ancho de banda (bandwidth)** = capacidad teórica; **throughput** = rendimiento real; **latencia** = retardo extremo a extremo; **jitter** = variación de la latencia; **ancho de banda asimétrico** (bajada ≠ subida) típico en cable/DSL de hogar.

### 2.6 Puntos para Hacer una Red Confiable (Clase 2.6)
Los cuatro pilares de una red fiable según Cisco:

1. **Tolerancia a fallos (fault tolerance):** redundancia de enlaces/equipos para que un fallo no tumbe el servicio (p. ej., enlaces duplicados, fuentes redundantes). Incluye conmutación por error (failover).
2. **Escalabilidad (scalability):** la red debe crecer sin rediseño completo (diseño jerárquico: capa de acceso, distribución y núcleo).
3. **Calidad de servicio (QoS, Quality of Service):** priorizar tráfico sensible (voz, vídeo) sobre tráfico tolerante (descarga de ficheros) mediante clasificación y colas.
4. **Seguridad (security):** confidencialidad, integridad y disponibilidad; segmentación, control de acceso, cifrado, monitorización.

### 2.7 Principales Tendencias Hoy en Día (Clase 2.7)
- **BYOD** (Bring Your Own Device): el empleado usa su propio dispositivo; exige políticas de seguridad y segmentación.
- **Colaboración en línea:** videoconferencia, equipos distribuidos, herramientas cloud (Teams, Meet).
- **Computación en la nube (cloud):** SaaS, PaaS e IaaS; los datos y servicios viven fuera de la empresa.
- **Comunicación audio/vídeo integrada** en la red corporativa (VoIP, videoconferencia).
- **Internet of Things (IoT):** sensores y actuadores conectados; enorme crecimiento de dispositivos y datos; nuevas superficies de ataque.
- **Redes definidas por software (SDN)** e infraestructura como código: fuera de alcance profundo en este curso, pero es la dirección del sector.

### 2.8 Seguridad de la Red (Clase 2.8)
Principios clave:
- **Confidencialidad:** solo quien debe, ve los datos (cifrado).
- **Integridad:** los datos no se alteran en tránsito (hashing, firmas).
- **Disponibilidad:** la red sigue operativa (redundancia, protección DDoS).

Amenazas habituales: virus, gusanos (worms), troyanos, ransomware, phishing, ataques de denegación de servicio (DoS/DDoS), man-in-the-middle. Medidas: políticas de seguridad, contraseñas robustas, actualizaciones, segmentación, firewalls, formación de usuarios. (Desarrollado en la sección 17.)

### 2.9 Introducción a IPs y Sistema Binario (Clase 2.9)
- Una **dirección IP** (IPv4) tiene 32 bits, escrita como 4 octetos decimales separados por puntos: `192.168.1.10`.
- Cada octeto representa 8 bits, valor 0–255.
- **Sistema binario:** potencias de 2. De izquierda a derecha los pesos son: 128, 64, 32, 16, 8, 4, 2, 1.

Conversión decimal → binario: restar los pesos que caben. Ejemplo: `192` → 128+64 = `11000000`. `168` → 128+32+8 = `10101000`. `1` → `00000001`. `10` → `00001010`. Resultado: `192.168.1.10` = `11000000.10101000.00000001.00001010`.

Conversión binario → decimal: sumar los pesos donde hay un 1. `11000000` = 128+64 = 192.

| Decimal | Binario | Decimal | Binario |
|---------|---------|---------|---------|
| 0 | 00000000 | 128 | 10000000 |
| 1 | 00000001 | 192 | 11000000 |
| 10 | 00001010 | 224 | 11100000 |
| 172 | 10101100 | 240 | 11110000 |
| 192 | 11000000 | 255 | 11111111 |

Aplicación: la máscara `/24` = `255.255.255.0` = `11111111.11111111.11111111.00000000`. AND lógico entre IP y máscara determina la red (lo verás en profundidad en la sección 12).

### 2.10 Preguntas del Módulo
Quiz de autoevaluación de la sección. Intenta responder las preguntas de autoevaluación al final antes de mirar las soluciones.

## Ejemplos

### Ejemplo 1 — Diagrama físico vs lógico
Caso: oficina con router, switch, 3 PCs y AP Wi‑Fi.
- Físico: "R1 conectado al switch SW1 por el puerto Gi0/1; PCs por Fa0/1-3; AP en Fa0/4".
- Lógico: "Red 192.168.10.0/24; gateway 192.168.10.1; DHCP reparte .100–.199; VLAN 10 para datos y VLAN 20 para invitados".
Mismo sistema, dos vistas: el físico sirve para cablear, el lógico para documentar y depurar.

### Ejemplo 2 — Elección de conexión a Internet
Una pyme con 25 empleados y videoconferencia diaria necesita subida simétrica: la mejor opción es **fibra (FTTH)**. Un hogar con streaming puede vivir con **cable HFC**. Una casa rural sin fibra ni cable puede usar **WISP** o **5G**; el **dial-up** está obsoleto. Un banco con sede y sucursales críticas contrata **línea dedicada/Metro Ethernet** con SLA.

### Ejemplo 3 — Conversión binario (paso a paso)
Convertir `172.16.5.200` a binario:
- 172 = 128+32+8+4 → `10101100`
- 16 = 16 → `00010000`
- 5 = 4+1 → `00000101`
- 200 = 128+64+8 → `11001000`
Resultado: `10101100.00010000.00000101.11001000`. Verificación inversa: suma los pesos y comprueba 172, 16, 5, 200. ✔

### Ejemplo 4 — Red confiable en un colegio
- **Tolerancia a fallos:** doble enlace entre switch de núcleo y distribución.
- **Escalabilidad:** diseño en capas; añadir un switch de acceso no toca el núcleo.
- **QoS:** priorizar VoIP del aula virtual sobre descargas de alumnos.
- **Seguridad:** VLAN de invitados separada + firewall con salida controlada.

## Ejercicios

1. **(Recordar)** Define red, dispositivo final, dispositivo intermedio y medio de red.
2. **(Recordar)** Enumera los 4 puntos de una red confiable.
3. **(Aplicar)** Convierte a binario: (a) 10, (b) 172, (c) 192, (d) 255. Convierte a decimal: (a) `00001010`, (b) `11000000`, (c) `11111111`.
4. **(Aplicar)** Para cada escenario, indica la conexión a Internet más adecuada: (a) hogar urbano con fibra disponible, (b) aldea sin infraestructura cableada, (c) banco con sucursales y SLA, (d) teletrabajador en movilidad.
5. **(Comprender)** Explica con un ejemplo la diferencia entre ancho de banda y throughput.
6. **(Analizar)** Una red empresarial sin redundancia sufre una caída de 4 h por un único enlace roto. ¿Qué pilar de red confiable faltó? ¿Qué dos medidas propones?
7. **(Analizar)** ¿Por qué BYOD aumenta el riesgo de seguridad y qué dos controles básicos mitigarían el problema?

<details>
<summary>Soluciones</summary>

1. Red: conjunto de dispositivos interconectados que intercambian datos con protocolos comunes. Dispositivo final: origen/destino (PC, servidor). Intermedio: interconecta y dirige (switch, router, AP, firewall). Medio: canal físico (cobre, fibra, aire).
2. Tolerancia a fallos, escalabilidad, calidad de servicio (QoS), seguridad.
3. Binario: 10 = `00001010`; 172 = `10101100`; 192 = `11000000`; 255 = `11111111`. Decimal: `00001010`=10; `11000000`=192; `11111111`=255.
4. (a) Fibra FTTH; (b) WISP o 5G (satelital como respaldo); (c) línea dedicada/Metro Ethernet con SLA; (d) celular 4G/5G.
5. Ancho de banda = capacidad máxima teórica del enlace (p. ej., 100 Mbps). Throughput = velocidad efectiva que obtienes (p. ej., 70 Mbps por congestión). Jitter = variación del retardo; latencia = retardo en sí.
6. Falto tolerancia a fallos. Medidas: enlace redundante (con failover) y equipos/fuentes redundantes (o rutas alternativas de salida a Internet). También documentar y supervisar.
7. Dispositivos no gestionados pueden traer malware, estar desactualizados o perderse/robarse. Controles: segmentación (VLAN de invitados/BYOD + ACL) y política de acceso (NAC/802.1X, contraseñas y cifrado, MDM).
</details>

## Tips
- **Pesos binarios:** memoriza `128 64 32 16 8 4 2 1`. Truco: cada peso es el doble del siguiente (128 es 2⁷... y 1 es 2⁰).
- El examen pregunta comparaciones de tipos de red (LAN vs WAN vs MAN) y pilares de fiabilidad casi siempre en formato test.
- DSL/Cable son **asimétricos**; fibra y línea dedicada pueden ser **simétricas**.
- Intranet = interna; extranet = socios; Internet = pública.
- No confundas PAN (metros) con LAN (edificio) ni SAN (almacenamiento, no "red de área de almacenamiento" para usuarios).

## Datos curiosos
- La primera red operacional de conmutación de paquetes, **ARPANET** (1969), conectó UCLA, el Stanford Research Institute, UC Santa Barbara y la Universidad de Utah; el primer mensaje transmitido fue "LO" (se cayó el sistema al intentar escribir "LOGIN").
- El término **Internet** deriva de "inter-networking": interconectar redes heterogéneas mediante un protocolo común (TCP/IP), diseñado en los años 70 por Vint Cerf y Bob Kahn, considerados "padres de Internet".
- IPv4 se estandarizó en 1981 (RFC 791) con solo ~4 300 millones de direcciones; el agotamiento de direcciones IPv4 motivó IPv6 (RFC 8200, antes 2460).

## Para tu libreta de apuntes
- Red = finales + intermedios + medios + protocolos.
- Pilares: tolerancia a fallos · escalabilidad · QoS · seguridad.
- Conexiones: fibra > cable ≈ DSL > satélite/celular en latencia; dedicada = SLA.
- Binario: pesos 128-64-32-16-8-4-2-1; AND con máscara → red (sección 12).
- Tendencias: BYOD, cloud, IoT, colaboración, SDN.
- Seguridad CIA: Confidencialidad, Integridad, Disponibilidad.

## Preguntas de autoevaluación

1. Una red que abarca una ciudad se denomina… **Respuesta:** MAN (Metropolitan Area Network).
2. ¿Qué diferencia hay entre intranet y extranet? **Respuesta:** La intranet es privada e interna; la extranet da acceso controlado a terceros (socios/proveedores) a parte de ella.
3. El pilar que garantiza que un fallo de hardware no tumbe la red es… **Respuesta:** Tolerancia a fallos.
4. Convierte `11001000` a decimal. **Respuesta:** 200.
5. Nombra los tres principios de seguridad de la información. **Respuesta:** Confidencialidad, Integridad y Disponibilidad (CIA).
