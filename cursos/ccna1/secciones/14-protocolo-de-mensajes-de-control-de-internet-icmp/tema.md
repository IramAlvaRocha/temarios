# 14 — Protocolo de Mensajes de Control de Internet (ICMP)

> Duración del curso: 13m · 2 clases
> Fuente: temario.md del curso

## Objetivos de aprendizaje
- Explicar qué es ICMP y su papel: mensajes de error, diagnóstico y control (sin garantía de entrega).
- Identificar los mensajes ICMP más importantes (echo request/reply, unreachable, time exceeded, redirect) y su uso por `ping` y `traceroute`.
- Diferenciar ICMPv4 de ICMPv6 y conocer NDP como extensión de ICMPv6.

## Teoría

### 14.1 Características del Mensaje ICMP (Clase 14.1)
**ICMP** (Internet Control Message Protocol, RFC 792 para IPv4; RFC 4443 para IPv6) complementa a IP: comunica **errores y condiciones de diagnóstico** entre nodos de red. No transporta datos de aplicación; viaja encapsulado en IP (número de protocolo **1** en IPv4; ICMPv6 es **58**).

Estructura: cabecera ICMP con **Tipo** (8 bits), **Code** (8 bits, subtipo), **Checksum** (16 bits, cubre todo el mensaje ICMP) + datos (normalmente incluye la cabecera IP original y 8 bytes del datagrama que causó el error — suficiente para identificar la conexión en TCP/UDP).

**Mensajes ICMPv4 clave:**

| Tipo | Nombre | Uso |
|------|--------|-----|
| 0 | Echo Reply | Respuesta de `ping` |
| 3 | Destination Unreachable | Destinatario/puerto/red inalcanzable. Codes: 0 red, 1 host, 2 protocolo, 3 puerto, 4 fragmentación necesaria (DF), 13 comunicación prohibida (admin) |
| 5 | Redirect | Hay una ruta mejor por otro gateway |
| 8 | Echo Request | Solicitud de `ping` |
| 11 | Time Exceeded | TTL=0 en tránsito (Code 0) o reensamblado agotado (Code 1). Base de `traceroute` |
| 4 | Source Quench (obsoleto) | Control de congestión antiguo; en desuso (RFC 6633) |
| 12 | Parameter Problem | Cabecera IP inválida |
| 13/14 | Timestamp | Medida de retardo (poco usado) |

Reglas fundamentales:
- Los mensajes ICMP **de error** (tipos 3, 11, 12, 5) nunca se generan en respuesta a: otro ICMP de error, un broadcast/multicast, ni a tramas de broadcast. (Evita tormentas de errores.)
- ICMP no garantiza entrega ni autenticación: los ataques con ICMP falsificado son posibles → los firewalls suelen limitar/filtrar ICMP (permitiendo lo esencial: el tipo 8/0 para ping, 3/4 para PMTUD).

**Herramientas que usan ICMP:**
- `ping`: envía ICMP Echo Request (tipo 8) y espera Echo Reply (tipo 0). Mide reachability y RTT (tiempo de ida y vuelta). Salida típica: bytes, TTL, tiempo en ms.
- `traceroute` / `tracert`: envía paquetes (UDP con ports altos en Unix, ICMP echo en Windows) con TTL incremental 1,2,3… Cada router decrementa TTL; al llegar a 0 responde ICMP Time Exceeded mostrando su IP → registro del camino. El destino final responde (port unreachable/echo reply o time exceeded según implementación).
- Descubrimiento de MTU (Path MTU Discovery): IP DF + ICMP tipo 3 code 4.

**ICMPv6 (RFC 4443):** obligatorio en IPv6 y mucho más rico:
- Tipo 1 Destination Unreachable · Tipo 2 Packet Too Big (PMTUD en IPv6; IPv4 usa 3/4) · Tipo 3 Time Exceeded · Tipo 4 Parameter Problem.
- Informativos: 128 Echo Request · 129 Echo Reply.
- NDP se implementa sobre ICMPv6: 133 RS, 134 RA, 135 NS, 136 NA, 137 Redirect (ver sección 10).
- PMTUD obligatorio en IPv6 (los routers no fragmentan).

**ICMP en IPv4 vs IPv6 (comparación rápida):**

| | ICMPv4 | ICMPv6 |
|---|--------|--------|
| Protocolo IP | 1 | 58 |
| Echo | 8/0 | 128/129 |
| Time exceeded | 11 | 3 |
| Unreachable | 3 | 1 |
| Fragmentación solicitada | 3/4 | 2 (Packet Too Big) |
| NDP | No aplica | 133–137 |

### 14.2 Preguntas del Módulo
Quiz de autoevaluación.

## Ejemplos

### Ejemplo 1 — Sesión `ping` en IOS
```
R1# ping 192.168.2.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.2.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms
```
Cada `!` = respuesta; `.` = timeout; `U` = unreachable recibido.

### Ejemplo 2 — `traceroute` desglosado
```
R1# traceroute 8.8.8.8
 1  192.168.1.1  1 ms  1 ms  1 ms
 2  10.0.0.1     8 ms  7 ms  9 ms
 3  * * *                              <- salto con ICMP bloqueado
 4  203.0.113.5  15 ms  14 ms  15 ms
```
Un `* * *` no implica rotura: el salto intermedio simplemente no genera Time Exceeded (filtrado ICMP o TTL ignorado por política).

### Ejemplo 3 — PMTUD con ICMP
PC envía paquetes DF; un enlace intermedio con MTU 1400 devuelve ICMP 3/4 "fragmentation needed, MTU=1400". El host reduce su MSS a 1360 y continúa: por eso bloquear ICMP rompe conexiones "que pingan pero no cargan webs".

### Ejemplo 4 — Errores no generados (regla de seguridad/estabilidad)
Red con tormenta de broadcasts L2: ningún router responde ICMP por cada trama broadcast descifrada, ni contesta a ICMP de error con ICMP de error: ambos casos están prohibidos por RFC para evitar amplificación/colapso.

## Ejercicios

1. **(Recordar)** ¿Qué número de protocolo IP usa ICMP en IPv4 e ICMPv6 en IPv6?
2. **(Recordar)** Enumera 3 tipos ICMP y su significado.
3. **(Aplicar)** Anota la salida esperada de `ping`: paquete llega al destino pero la respuesta se pierde en el camino de vuelta vs el destino no existe (2 casos).
4. **(Comprender)** Explica cómo `traceroute` usa TTL + ICMP Time Exceeded para mapear la ruta.
5. **(Comprender)** ¿Por qué no se envían mensajes ICMP de error en respuesta a broadcast?
6. **(Analizar)** Una web carga parcialmente / conexiones HTTPS se cuelgan pero `ping` funciona: ¿qué mensaje ICMP puede estar bloqueado y qué mecanismo depende de él?
7. **(Analizar)** Compara ICMPv4 e ICMPv6: echo, time exceeded, fragmentación: mapa de números de tipo.

<details>
<summary>Soluciones</summary>

1. ICMP = protocolo 1 en IPv4; ICMPv6 = 58.
2. 0 Echo Reply (respuesta ping), 3 Destination Unreachable (red/host/puerto no alcanzable; 4 = PMTUD), 8 Echo Request (ping), 11 Time Exceeded (TTL=0 / reensamblado).
3. Si la respuesta se pierde: verás `.`/timeout aunque el destino exista (no distinguible de caída total por ping simple). Si el destino/red no existe: los routers intermedios o el gateway devuelven ICMP 3 (unreachable) — en Windows verás "Destination host unreachable" (generado localmente si no hay ruta) o "Reply from ... Destination net unreachable" — el host puede mostrar U en IOS.
4. Sondas con TTL=1,2,3…; cada router decrementa y, con TTL=0, descarta y origina Time Exceeded (11) con su IP como origen: la secuencia de respuestas revela los saltos en orden.
5. Para evitar tormentas de respuestas/amplificación (un broadcast podría provocar miles de ICMP); además un error sobre un broadcast no es útil (el emisor puede no ser identificable de forma fiable y la respuesta iría a todos).
6. El tipo **3 code 4** ("Fragmentation needed") / en IPv6 el tipo **2** (Packet Too Big): **Path MTU Discovery** depende de él; si el firewall lo filtra, TCP no puede adaptar su MSS y las sesiones con paquetes grandes (HTTPS/TLS) se cuelgan.
7. Echo req/reply: 8/0 → 128/129. Time exceeded: 11 → 3. Packet too big (PMTUD): 3/4 → 2. Redirect: 5 → 137. Unreachable: 3 → 1.
</details>

## Tips
- `ping` = ICMP 8 → 0; `traceroute` = TTL + ICMP 11 (+ 3/4 para terminar).
- Nunca culpes a ICMP de "no ser TCP": ICMP **no** retransmite; solo informa.
- En el examen, "¿Qué protocolo usa ping?" → **ICMP** (nunca TCP/UDP).
- Windows tracert usa ICMP echo; Linux traceroute usa UDP a puertos altos por defecto (`traceroute -I` fuerza ICMP).
- Bloquear todo ICMP en firewalls rompe PMTUD y diagnóstico: filtra por tipos, no "todo o nada".

## Datos curiosos
- El ICMP original (RFC 792, 1981) cabe en media página de cabecera y explica por qué "el ping funciona" fue la prueba de fuego de los primeros días de Internet; el famoso `ping` fue creado por Mike Muuss en 1983 (y su nombre viene del sonar: el eco submarino).
- En 2001-2003 el gusano **Slammer** y los ataques Smurf (ICMP echo amplificado a broadcast) obligaron a cambiar las políticas ICMP de los operadores: la lección fue "ICMP no es el enemigo, el filtrado ciego sí".
- ICMPv6 integra NDP, autoconfig y PMTUD; sin ICMPv6, IPv6 simplemente no funciona — por eso los firewalls IPv6 deben permitir ICMPv6 esencial sí o sí (RFC 4890 incluso lo recomienda).

## Para tu libreta de apuntes
- ICMP = errores/control (no datos). En IP: protocolo 1 (v4) / 58 (v6).
- ping: Echo Request 8 → Echo Reply 0. · TTL→0 ⇒ Time Exceeded 11.
- Unreachable 3 (codes: 0 red,1 host,3 puerto,4 PMTUD,13 admin) · Redirect 5.
- No: respuestas a broadcast ni a errores ICMP.
- IPv6: Echo 128/129 · Packet Too Big 2 · Time Exceeded 3 · NDP en ICMPv6 (133–137).

## Preguntas de autoevaluación

1. ¿Qué tipo ICMP responde un host a un ping? **Respuesta:** Echo Reply (tipo 0).
2. ¿Qué mensaje permite a `traceroute` listar los routers del camino? **Respuesta:** Time Exceeded (tipo 11 en v4 / tipo 3 en v6).
3. ¿Qué código de Unreachable informa de MTU insuficiente? **Respuesta:** Tipo 3 Code 4 (Packet Too Big = tipo 2 en IPv6).
4. Verdadero/Falso: "ICMP garantiza la entrega de los datos IP". **Respuesta:** Falso; es un protocolo de control/errores sin garantía.
5. Nombra los tres mecanismos ICMPv6 que IPv4 no tiene (o no de la misma forma) derivados de NDP. **Respuesta:** RS/RA (autoconfig de routers y prefijos), NS/NA (resolución de vecinos = ARP) y DAD (detección de duplicados).
