# 07 — Capa 2: Capa de Enlace de Datos

> Duración del curso: 33m · 6 clases
> Fuente: temario.md del curso

## Objetivos de aprendizaje
- Explicar la función de la capa 2 (IEEE 802, subcapas LLC/MAC, acceso al medio).
- Describir las topologías físicas y lógicas (bus, anillo, estrella, híbridas/malla).
- Desglosar la trama de capa 2: campos, FCS, direcciones MAC.
- Comparar el acceso al medio en medios compartidos (CSMA/CD) vs punto a punto.

## Teoría

### 7.1 Función de la Capa 2 (Clase 7.1)
La **capa de enlace de datos** (Data Link) hace que el transporte por el medio físico sea fiable y ordenado. Sus funciones:
- **Entramado (framing):** envolver los paquetes de capa 3 en **tramas** con cabecera y tráiler.
- **Direccionamiento físico:** direcciones **MAC** (48 bits, p. ej. `00:1A:2B:3C:4D:5E`).
- **Control de acceso al medio:** quién puede transmitir y cuándo (CSMA/CD, turnos, token).
- **Detección de errores:** **FCS** (Frame Check Sequence, CRC-32 en Ethernet); si no cuadra, la trama se descarta.
- **Control de flujo** (en algunos protocolos) y sincronización de trama.

La capa 2 se divide en dos subcapas (IEEE 802):
- **LLC** (Logical Link Control, 802.2): interfaz con la capa superior; identifica el protocolo de red (Type/Length).
- **MAC** (Media Access Control): acceso al medio, direccionamiento físico y encapsulación.

### 7.2 Topología de Bus (Clase 7.2)
Un **bus** es un único cable troncal compartido (coaxial en Ethernet original 10BASE2/10BASE5) con terminadores en los extremos (50 Ω). Todos los nodos ven todas las tramas.
- Ventajas: simple, barato, sin switches.
- Problemas: un corte tumba toda la red; colisiones frecuentes; difícil de diagnosticar y escalar; terminadores obligatorios.
- Método de acceso: **CSMA/CD** (Ethernet compartida). Obsoleta hoy; Ethernet moderno usa estrellas con switches en full dúplex (sin colisiones).

### 7.3 Topología de Anillo (Clase 7.3)
Cada nodo se conecta al siguiente formando un anillo cerrado. El tráfico circula en un sentido (simple) o dos (dual).
- **Token Ring (IEEE 802.5)** y **FDDI** (fibra, anillos duales): un testigo (token) otorga el derecho a transmitir; sin colisiones.
- **Anillo simple vs doble:** el doble (FDDI) sobrevive a la rotura de un enlace/nodo, aislando el fallo.
- Problemas: un fallo aísla el anillo (en simple); latencia creciente con los nodos; rara vez se ve hoy salvo en redes industriales (p. ej., algunos buses de campo) y fibra metropolitana histórica (SONET/SDH usa anillos).

### 7.4 Topología de Estrella (Clase 7.4)
Todos los nodos cuelgan de un punto central (hub o switch). Es la topología dominante:
- **Estrella física / lógica:** lógicamente es un bus compartido si el centro es un hub (colisiones, CSMA/CD); con un switch, cada enlace es un punto a punto full dúplex.
- Ventajas: un cable por nodo; un fallo de un enlace no afecta al resto; fácil de ampliar y diagnosticar. Punto único de fallo: el dispositivo central (se mitiga con redundancia).
- Variantes: **estrella extendida** (varias estrellas enlazadas), **árbol (jerárquico)**: capa de acceso–distribución–núcleo (diseño recomendado por Cisco), **malla** (parcial/completa: enlaces redundantes entre nodos, usado en núcleo/WAN).

Comparativa de topologías:

| Topología | Medios | Fallo de un nodo | Colisiones | Uso actual |
|-----------|--------|------------------|------------|------------|
| Bus | Coaxial compartido | Tumba toda la red | Sí | Obsoleta |
| Anillo | Anillo | Tumba el anillo (simple) | No (token) | Heredada/industrial |
| Estrella (switch) | Cobre/fibra por nodo | Solo ese enlace cae | No (full dúplex) | Universal |
| Malla | Múltiples enlaces | Transparente (redundancia) | No | Núcleo/WAN/ISP |

### 7.5 Tramas de Capa 2 (Clase 7.5)
Estructura básica de una trama Ethernet II:

```
| Preámbulo+SFD (8 B) | MAC destino (6 B) | MAC origen (6 B) | Tipo/Length (2 B) | Datos + relleno (46–1500 B) | FCS (4 B) |
```
- **Preámbulo + SFD**: sincronización y marca de inicio.
- **MAC destino/origen**: 6 bytes; unicast, multicast (bit I/G = 1, p. ej. 01:00:5E:xx para IPv4 multicast) o broadcast (FF:FF:FF:FF:FF:FF).
- **Tipo** (Ethernet II): código del protocolo de capa 3 (0x0800 IPv4, 0x86DD IPv6, 0x0806 ARP). En tramas **802.3** el campo es Length (longitud) y LLC lleva el tipo (802.2 + SNAP).
- **Datos/relleno:** MTU mínimo 64 B de trama total → relleno si el payload < 46 B. MTU máximo 1500 B (jumbo frames ~9000 B, fuera de CCNA).
- **FCS**: CRC-32; el receptor lo recalcula y descarta si no coincide.

Trama IEEE 802.11 (Wi‑Fi): más compleja (duración, secuencia, direcciones 1–4 según AP/bridge). Mención suficiente para el examen: Wi‑Fi también usa tramas con MACs.

Errores de trama: trama **runt** (<64 B), **gigante/jumbo** (>1518 B), **con CRC malo**, **colisiones tardías** (late collisions) — suelen indicar problemas de cableado o duplex mismatch.

### 7.6 Preguntas del Módulo
Quiz de autoevaluación.

## Ejemplos

### Ejemplo 1 — Dominio de colisión vs difusión
8 PCs en un hub: un único dominio de colisión y uno de difusión. Las mismas 8 PCs en un switch: 8 dominios de colisión (uno por puerto) pero 1 de difusión. Con un router entre dos grupos de 4: 2 dominios de difusión. Los switches separan colisiones; los routers (o VLANs) separan difusiones.

### Ejemplo 2 — FCS en acción en Packet Tracer
En modo Simulation, envía un ping entre dos PCs conectados a un switch y fuerza un cable dañado (o un error) para observar el descarte por FCS en las estadísticas del puerto (`show interfaces` → `CRC` errors incrementándose). En IOS: `show interfaces fa0/1` → línea `... ... CRC`, `Runts`, `Giants`.

### Ejemplo 3 — Trama ARP desglosada
`FF:FF:FF:FF:FF:FF | AA:BB:CC:11:22:33 | 0x0806 | ...ARP... | FCS`
- Destino broadcast (todos la procesan), origen MAC del emisor, tipo 0x0806 (ARP), relleno si hace falta para llegar a 64 B, FCS válido.

### Ejemplo 4 — De hub a switch
Oficina 1998: 10BASE2 en bus coaxial, un corte en el cable tumba toda la planta. Migración a estrella con switches 2960: cada mesa con su latiguillo; un cable cortado solo afecta a ese usuario; el switch aprende MACs y conmuta en línea retrás. Diagnóstico: `show mac address-table`, `show interfaces status`.

## Ejercicios

1. **(Recordar)** ¿Qué hace la capa 2 con los paquetes de capa 3?
2. **(Recordar)** ¿Qué significa FCS y qué detecta?
3. **(Aplicar)** Escribe los campos de una trama Ethernet II en orden, con tamaños.
4. **(Aplicar)** Clasifica las topologías: bus, anillo simple, estrella con switch, malla completa — en "falla de un enlace" indicando cuántas estaciones quedan aisladas.
5. **(Comprender)** ¿Por qué una estrella con hub sigue teniendo colisiones y una con switch no?
6. **(Analizar)** `show interfaces` muestra 500 CRC y 300 runts en un puerto. ¿Qué sospechas? Cita dos causas.
7. **(Analizar)** ¿Qué significan los bits I/G y G/L de una MAC? Pon un ejemplo de MAC multicast y una broadcast.
8. **(Aplicar)** En Packet Tracer, conecta 4 PCs a un switch y envíales un broadcast (ping a una IP inexistente provoca ARP broadcast); observa en Simulation cómo se replica y cómo responden solo los interesados.

<details>
<summary>Soluciones</summary>

1. Los encapsula en tramas (cabecera+tráiler), añadiendo direcciones MAC origen/destino y FCS, y controla el acceso al medio.
2. Frame Check Sequence: CRC-32 que cubre la trama; detecta tramas corruptas (bit flips) en la transmisión; la trama se descarta silenciosamente.
3. Preámbulo+SFD 8 · MAC destino 6 · MAC origen 6 · Tipo 2 · Datos 46–1500 · FCS 4.
4. Bus: un corte aísla el bus completo (todos en el lado afectado). Anillo simple: un fallo rompe el anillo y puede aislar a todos. Estrella (switch): solo el nodo de ese enlace cae. Malla completa: nadie queda aislado (hay enlaces alternativos).
5. El hub repite la señal por todos sus puertos: medio compartido half-duplex → colisiones y CSMA/CD. El switch crea enlaces punto a punto full-duplex por puerto: sin medio compartido, sin colisiones.
6. Sospecha de capa física/1-2: cable dañado o de mala calidad, conector mal ponchado, duplex/speed mismatch, interferencias (EMI), distancia excesiva.
7. I/G (bit menos significativo del primer byte): 0 = unicast, 1 = multicast (group). G/L (o U/L): 0 = universal (OUI del fabricante), 1 = local/administrada. Broadcast `FF:FF:FF:FF:FF:FF`; multicast IPv4 típica `01:00:5E:xx:xx:xx`.
8. En Simulation se ve la ARP request replicada a todos los puertos del switch y el ARP reply del único destino; filtrar por ARP/ICMP para seguir el flujo.
</details>

## Tips
- **Tamaño mínimo de trama Ethernet = 64 B, máximo = 1518 B** (sin VLAN tag; con tag 802.1Q, 1522 B). El padding existe para que las colisiones sean detectables (slot time 512 bits).
- FCS = 4 bytes CRC-32; si falla, descarta sin avisar (capa 2 no retransmite: de eso se ocupan TCP o la aplicación).
- Broadcast no cruza routers; las VLANs también segmentan difusión.
- El examen cae: subcapas LLC/MAC, FCS, topologías, "¿qué pasa si el cable del bus se corta?", MAC multicast `01:00:5E`.
- Recordatorio: un switch separa dominios de colisión (uno por puerto), no de difusión; un router separa ambos.

## Datos curiosos
- Las direcciones MAC las asigna el IEEE mediante el **OUI** (Organizationally Unique Identifier, los 3 primeros bytes) registrado por cada fabricante; los 3 últimos los pone el fabricante de forma única. Por eso se puede averiguar la marca de una tarjeta de red por su MAC.
- **FF:FF:FF:FF:FF:FF** es un destinatario especial: "todos en este segmento". Las NICs la aceptan siempre; una IP broadcast típica es 192.168.1.255.
- Token Ring de IBM (1985) tenía prioridades en el anillo; FDDI a 100 Mbps sobre fibra fue el "backbone" empresarial de los 90 antes de Gigabit Ethernet.
- CSMA/CD es históricamente hermano de ALOHA (red hawaiana de radio de los 70), del que deriva: "si no puedes oír al emisor central, transmite; si hay colisión, reintenta tras un tiempo aleatorio (backoff exponencial binario)".

## Para tu libreta de apuntes
- Capa 2: framing · MAC · acceso al medio · CRC/FCS · LLC/MAC (802.2/802.3).
- Trama II: preámbulo8 | dst 6 | src 6 | tipo 2 | datos 46–1500 | FCS 4 (mín 64 B).
- FCS malo → descartar. Capa 2 NO retransmite.
- Topologías: bus < anillo < estrella (switch) < malla (redundancia).
- Hub = 1 dominio colisión; switch = N dominios colisión + 1 difusión; router = difusión separada.

## Preguntas de autoevaluación

1. ¿Qué subcapas componen la capa 2 según IEEE 802 y qué hace cada una? **Respuesta:** LLC (interfaz con capa 3, identifica protocolo) y MAC (acceso al medio, direccionamiento físico).
2. ¿De cuántos bytes es una dirección MAC y qué representa el primer byte? **Respuesta:** 6 bytes (48 bits); el primer byte lleva el bit I/G (individual/group) y U/L (universal/local).
3. ¿Qué ocurre con una trama cuyo FCS no coincide? **Respuesta:** Se descarta sin retransmisión en capa 2.
4. En una estrella con switch, ¿cuántos dominios de colisión hay por puerto? **Respuesta:** Uno por puerto (enlaces punto a punto full dúplex, sin colisiones).
5. Nombra dos topologías redundantes usadas en redes núcleo/WAN. **Respuesta:** Malla (parcial o completa) y anillo dual (FDDI/SONET); estrella extendida/jerárquica con redundancia.
