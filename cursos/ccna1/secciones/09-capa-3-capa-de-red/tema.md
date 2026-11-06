# 09 — Capa 3: Capa de Red

> Duración del curso: 57m · 5 clases
> Fuente: temario.md del curso

## Objetivos de aprendizaje
- Explicar las funciones de la capa 3: direccionamiento lógico, encapsulamiento, enrutamiento y fragmentación.
- Comparar los encabezados IPv4 e IPv6 y sus campos principales.
- Describir cómo un host enruta hacia redes remotas (default gateway).
- Explicar el papel del router, la tabla de enrutamiento y las rutas estáticas/dinámicas.

## Teoría

### 9.1 Características (Clase 9.1)
La **capa de red** (OSI capa 3) transporta datos entre redes distintas (no dentro del mismo segmento; eso es capa 2). Funciones:

- **Direccionamiento lógico:** direcciones IP (IPv4: 32 bits; IPv6: 128 bits), jerárquicas (red + host) frente a las MAC planas de capa 2.
- **Encapsulamiento:** añade cabecera IP al segmento → **paquete**.
- **Enrutamiento (routing):** elección del siguiente salto hacia el destino usando la tabla de enrutamiento.
- **Desencapsulamiento** en destino y entrega a capa 4.
- **Fragmentación/reensamblado:** IP permite fragmentar datagramas mayores que el MTU (IPv4; IPv6 solo fragmenta el origen).
- Entrega **no garantizada** (best effort): sin acuse ni retransmisión (de eso se encarga TCP arriba).

Protocolos clave: **IP** (IPv4/IPv6), **ICMP** (control/errores, sección 14), **OSPF/EIGRP/BGP** (enrutamiento dinámico, parte 2 del curso), **IPsec** (cifrado).

### 9.2 Paquetes IPv4 e IPv6 (Clase 9.2)
**Cabecera IPv4 (mínima 20 B, con opciones hasta 60 B):**

```
0                   1                   2                   3
| Ver(4) IHL(4) | DSCP/ECN(8) | Total Length(16)          |
| Identification(16)           | Flags(3) FragOffset(13)   |
| TTL(8)        | Protocol(8) | Header Checksum(16)       |
| Source Address (32)                              |
| Destination Address (32)                         |
| Options (0–40 B, opcional) + Padding             |
```
Campos significativos:
- **Version** = 4; **IHL** = longitud de cabecera en palabras de 32 bits (mínimo 5 → 20 B).
- **Total Length:** paquete completo (máx 65535 B).
- **Identification/Flags/Fragment Offset:** fragmentación (DF = Don't Fragment; MF = More Fragments).
- **TTL (Time To Live):** 8 bits; cada router decrementa 1; si llega a 0 → ICMP Time Exceeded (evita bucles eternos).
- **Protocol:** protocolo de la capa superior: **6 = TCP, 17 = UDP, 1 = ICMP**, 89 = OSPF.
- **Header Checksum:** solo cubre la cabecera (se recalcula en cada salto porque TTL cambia).
- Direcciones origen/destino: 32 bits.

**Cabecera IPv6 (fija 40 B):**
```
| Version(4) | Traffic Class(8) | Flow Label(20) |
| Payload Length(16) | Next Header(8) | Hop Limit(8)  |
| Source Address (128) ...                       |
| Destination Address (128) ...                  |
```
- **Next Header** sustituye a Protocol (TCP=6, UDP=17, ICMPv6=58, o cabeceras de extensión).
- **Hop Limit** sustituye a TTL.
- **Sin checksum** de cabecera (se delega en capas inferiores/superiores → routers más rápidos).
- **Sin fragmentación en routers**: solo el origen fragmenta (Path MTU Discovery).
- **Flow Label** permite trato QoS por flujo.
- Cabeceras de extensión (Hop-by-Hop, Routing, Fragment...): fuera de alcance CCNA salvo identificación.

| Característica | IPv4 | IPv6 |
|---|---|---|
| Bits de dirección | 32 | 128 |
| Cabecera | 20–60 B | 40 B fija |
| Checksum | Sí | No |
| Fragmentación en routers | Sí | No (solo origen) |
| Broadcast | Sí | No (solo multicast) |
| ARP | Sí | No (NDP/ICMPv6) |
| Configuración | DHCP/estática | SLAAC/DHCPv6/estática |

### 9.3 Routing desde un Host (Clase 9.3)
Un host decide cómo enviar cada paquete según su **tabla de enrutamiento local**:
1. Si el destino está en su **misma subred** (IP destino AND máscara == red propia) → entrega directa por la LAN (resuelve MAC con ARP y envía trama).
2. Si no → envía al **default gateway** (router), resolviendo la MAC del gateway por ARP.
El host no necesita conocer la ruta completa: solo su gateway.
Tabla en Windows: `route print`; Linux: `ip route`.

### 9.4 Introducción al Routing (Clase 9.4)
El **router** es el dispositivo de capa 3:
- Tiene interfaces en **redes distintas**; cada interfaz conectada es una red directamente conectada en su tabla.
- **Tabla de enrutamiento:** destinos (red + máscara) → siguiente salto/interfaz, con métrica y "administrative distance" (AD) de origen: conectada (0), estática (1), OSPF (110), EIGRP interna (90), RIP (120)...
- Orden de preferencia ante varias rutas al mismo destino: menor AD → menor métrica.
- Rutas:
  - **Directamente conectadas** (C y L en `show ip route`: C = red, L = /32 del propio enlace).
  - **Estáticas:** configuradas a mano (`ip route 10.0.0.0 255.0.0.0 192.168.1.2`). Simples, sin overhead, pero no se adaptan a fallos.
  - **Dinámicas:** aprendidas por protocolos (OSPF, EIGRP, BGP...). Escalables y adaptables; más complejas (fuera de alcance en esta sección).
  - **Default route:** `0.0.0.0 0.0.0.0` ("gateway de último recurso").
- Longest-prefix-match: ante varias rutas que coinciden con el destino, gana la más específica (mayor prefijo).

Comandos clave:
```
R1# show ip route            ! tabla completa (C, L, S, O, D...)
R1# show ip route static
R1(config)# ip route 192.168.2.0 255.255.255.0 10.0.0.2   ! estática
R1(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.2             ! por defecto
```

### 9.5 Preguntas del Módulo
Quiz de autoevaluación.

## Ejemplos

### Ejemplo 1 — El host decide: ¿misma subred o no?
PC `192.168.1.10/24` quiere enviar a:
- `192.168.1.50` → misma red (192.168.1.0/24) → ARP directo a la MAC del destino, trama directa.
- `192.168.2.50` → red remota → trama destinada a la MAC del **gateway** (192.168.1.1); IP destino sigue siendo 192.168.2.50.

### Ejemplo 2 — Lectura de `show ip route`
```
R1# show ip route
Gateway of last resort is 10.0.0.2 to network 0.0.0.0

C    192.168.1.0/24 is directly connected, GigabitEthernet0/0
L    192.168.1.1/32 is directly connected, GigabitEthernet0/0
S    192.168.2.0/24 [1/0] via 10.0.0.2
S*   0.0.0.0/0 [1/0] via 10.0.0.2
```
`S` = estática; `[1/0]` = distancia administrativa 1 / métrica 0; `S*` = candidata a ruta por defecto.

### Ejemplo 3 — TTL en acción
`traceroute 8.8.8.8` envía sondas con TTL=1,2,3...: cada router devuelve ICMP Time Exceeded, revelando su IP. Cuando el TTL alcanza al destino, este responde (echo reply o port unreachable). Así se obtiene el camino salto a salto.

### Ejemplo 4 — Fragmentación IPv4
Paquete IP de 4000 B (20 B de cabecera + 3980 B de payload) sale de un PC hacia un enlace con MTU 1500: se fragmenta en 3 datagramas de 1500, 1500 y 1040 B (payloads 1480 + 1480 + 1020). Con DF activado y MTU insuficiente en la ruta: router descarta y devuelve **ICMP Fragmentation Needed (Type 3 Code 4)**; TCP reduce el MSS (Path MTU Discovery). IPv6: el origen hace PMTUD siempre.

## Ejercicios

1. **(Recordar)** Enumera las diferencias entre IPv4 e IPv6 en: bits, tamaño de cabecera, checksum, fragmentación en routers.
2. **(Recordar)** ¿Qué campo IPv4 evita bucles infinitos y qué ocurre cuando llega a 0?
3. **(Aplicar)** PC `10.1.1.5/8` quiere hablar con `10.2.2.5`. Gateway `10.1.1.1`. ¿Por qué interfaz/MAC sale el paquete? ¿Qué IP destino lleva?
4. **(Aplicar)** Escribe el comando para crear una ruta estática a `172.16.0.0/16` vía `192.168.1.254` y una ruta por defecto vía el mismo salto.
5. **(Comprender)** ¿Por qué IP es "best effort" y quién se encarga de la fiabilidad?
6. **(Analizar)** Ante dos rutas hacia `10.0.0.0/8`: OSPF (AD 110, métrica 20) y estática (AD 1). ¿Cuál instala el router y por qué?
7. **(Analizar)** ¿En qué protocolo IP va un ping? ¿Y OSPF? Indica los números de protocolo de capa 4-superior en IP.
8. **(Aplicar)** En Packet Tracer, monta 2 routers con enlaces a 2 LANs, crea rutas estáticas en ambos sentidos y verifica `ping` entre PCs remotos y `show ip route`.

<details>
<summary>Soluciones</summary>

1. 32 vs 128 bits; 20–60 B vs 40 B fijos; IPv4 tiene header checksum, IPv6 no; IPv4 fragmenta en routers, IPv6 solo en origen (PMTUD).
2. TTL. Al llegar a 0, el router descarta el paquete y envía **ICMP Time Exceeded** (Type 11) al origen.
3. Sale por la NIC a la MAC del gateway (ARP resuelve 10.1.1.1); el paquete lleva IP origen 10.1.1.5 → IP destino 10.2.2.5 (sin cambios).
4. `ip route 172.16.0.0 255.255.0.0 192.168.1.254` ; `ip route 0.0.0.0 0.0.0.0 192.168.1.254`.
5. IP no garantiza entrega, orden ni ausencia de duplicados; TCP (capa 4) añade fiabilidad (ACKs, retransmisión, orden). UDP no.
6. Instala la **estática** (AD 1 < 110): IOS prefiere origen más fiable antes que métrica.
7. Ping → ICMP (protocolo 1). OSPF → IP protocolo 89. TCP = 6, UDP = 17.
8. Verificar que `C`/`L` aparecen para las LANs, `S` para las remotas; ping PC→PC exitoso; borrar una estática reproduce la pérdida de conectividad (y `tracert` muestra solo hasta el router local).
</details>

## Tips
- **Protocol number** en IPv4 (1 ICMP / 6 TCP / 17 UDP / 89 OSPF) y **Next Header** en IPv6: pregunta fija de examen.
- TTL/Hop Limit es el motivo de que `tracert` funcione; ambos parten de 128/64/255 según SO (Windows 128, Linux 64, algunos routers 255).
- Longest prefix match gana siempre a AD.
- `S*` = ruta estática por defecto; `Gateway of last resort` aparece encabezando `show ip route`.
- En IPv6 no existe broadcast ni ARP: solo multicast y NDP.
- No confundir **MAC (capa 2, local)** con **IP (capa 3, global)** ni con **puerto (capa 4, aplicación)**.

## Datos curiosos
- IPv4 fue desplegado en 1983 con ~4 300 millones de direcciones; en **2011** la IANA agotó el pool global de /8 (el último /8 se asignó el 3 de febrero de 2011). Los RIRs repartieron sus últimos bloques poco después.
- IPv6 se definió en RFC 2460 (1998, actualizado por RFC 8200 en 2017) con 340 undecillones de direcciones (3.4×10³⁸): hay más IPv6 por grano de arena que IPv4 en total.
- El TTL original venía pensado como "segundos de vida"; en la práctica cada router decrementa 1 por salto, no por tiempo.
- IP nunca se diseñó para ser seguro: **IPsec** se añadió después (RFC 1825, 1995) y opera sobre IP como protocolo 50/51 o en UDP 500/4500 (NAT-T).

## Para tu libreta de apuntes
- Capa 3: IP lógica jerárquica · routing · fragmentación · best effort.
- IPv4: 32b · 20–60 B · checksum · TTL · Protocol (1/6/17/89) · fragmenta en ruta.
- IPv6: 128b · 40 B fija · Next Header · Hop Limit · sin checksum · solo origen fragmenta.
- Host → misma subred: directo · otra: gateway. Tabla: `route print` / `ip route`.
- Router: C/L conectadas · S estáticas · S* default · AD: conectada 0 < estática 1 < OSPF 110 < RIP 120.

## Preguntas de autoevaluación

1. ¿Qué campo IPv4 se decrementa en cada router y con qué valor inicial típico en Windows? **Respuesta:** TTL; Windows inicia en 128.
2. ¿Qué protocolo IP (número) usa ICMP? **Respuesta:** 1. (TCP 6, UDP 17.)
3. Un paquete IPv6 mayor que el MTU del enlace: ¿quién lo fragmenta? **Respuesta:** El nodo origen (los routers no fragmentan en IPv6).
4. ¿Qué ruta gana ante 10.0.0.0/8 OSPF vs 10.0.0.0/16 estática para destino 10.1.2.3? **Respuesta:** La /16 (longest prefix match), independientemente de la AD.
5. ¿Qué comando muestra la tabla de enrutamiento en un router Cisco? **Respuesta:** `show ip route`.
