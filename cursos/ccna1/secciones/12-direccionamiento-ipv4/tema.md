# 12 — Direccionamiento IPv4

> Duración del curso: 42m · 5 clases
> Fuente: temario.md del curso

## Objetivos de aprendizaje
- Aplicar la operación AND lógica entre IP y máscara para obtener la dirección de red.
- Identificar rangos de direcciones: públicas/privadas, especiales, clases (históricas) y notación CIDR/máscara.
- Realizar subnetting y VLSM para diseñar esquemas de direccionamiento eficientes.
- Diseñar una estructura de direccionamiento jerárquica para redes pequeñas.

## Teoría

### 12.1 Máscara de Red y Operación Lógica AND (Clase 12.1)
- IPv4 = 32 bits: **bits de red (1s en la máscara) + bits de host (0s)**.
- La máscara indica qué parte identifica la red y cuál al host. Se expresa como dot-decimal (`255.255.255.0`) o prefijo **CIDR** (`/24`).
- **AND lógico** (bit a bit entre IP y máscara) → dirección de red. Ejemplo:

```
IP:      192.168.1.130   11000000.10101000.00000001.10000010
Máscara: 255.255.255.192 11111111.11111111.11111111.11000000
AND:     192.168.1.128   11000000.10101000.00000001.10000000
```
→ la red es `192.168.1.128/26`.

Regla: dos hosts están en la misma red sii comparten el resultado del AND con su máscara.

### 12.2 Repaso UMB y Tipos de Direcciones IP (Clase 12.2)
"UMB" = repaso de **dirección de Red, dirección de Broadcast y rango Útil (rango utilizable)** — los tres extremos clave de cualquier subred:
- **Network address** = host bits todos a 0 (en el AND) — identifica la red, no asignable.
- **Broadcast address** = host bits todos a 1 — entrega a todos los hosts de la subred, no asignable.
- **Rango utilizable** = desde network+1 hasta broadcast−1 → hosts = 2^(bits de host) − 2.
- **Wildcard mask** (inversa): 0s donde la máscara tiene 1s (p. ej. `0.0.0.255` para /24) — se usa en ACLs/OSPF (mención).

**Clases históricas** (classful, en desuso desde CIDR/1993):

| Clase | Rango primer octeto | Máscara por defecto | Uso |
|-------|--------------------|---------------------|-----|
| A | 1–126 | /8 (255.0.0.0) | Muy grandes (126 redes) |
| B | 128–191 | /16 (255.255.0.0) | Grandes/medianas |
| C | 192–223 | /24 (255.255.255.0) | Pequeñas (254 hosts) |
| D | 224–239 | — | Multicast |
| E | 240–255 | — | Experimental/reservada |

Excepciones de nota: `127.0.0.0/8` loopback (no es clase A); `0.0.0.0` "esta red"; `255.255.255.255` broadcast limitado.

**Bloques especiales:**
- **Privadas (RFC 1918):** `10.0.0.0/8`, `172.16.0.0/12` (172.16–172.31), `192.168.0.0/16` — no enrutables en Internet; NAT las traduce.
- **Públicas:** asignadas por RIRs; únicas globalmente.
- **Loopback:** `127.0.0.0/8` (típ. `127.0.0.1`).
- **Link-local/APIPA:** `169.254.0.0/16` (autoasignada si no hay DHCP; RFC 3927).
- **CGN/shared:** `100.64.0.0/10` (RFC 6598).
- **Multicast:** `224.0.0.0/4` (224–239; p. ej. 224.0.0.1 = todos los nodos de la subred, 224.0.0.2 = routers).
- **Documentación/prueba:** `192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24` (RFC 5737).
- 0.0.0.0/8 "esto" / ruta por defecto; 255.255.255.255 broadcast.

**Binario de máscaras frecuentes:** `/24`=255.255.255.0 · `/25`=.128 · `/26`=.192 · `/27`=.224 · `/28`=.240 · `/29`=.248 · `/30`=.252 · `/31`=.254 (RFC 3021, enlaces punto a punto) · `/32`=.255.

### 12.3 Subnetting General y con VLSM (Clase 12.3)
**Subnetting:** dividir una red en subredes tomando bits prestados de host.
Fórmulas: **subredes = 2^s** (s = bits tomados) · **hosts/subred = 2^h − 2** (h = bits de host restantes) · **salto/incremento (block size) = 256 − valor del octeto de interés**.

Ejemplo: `192.168.1.0/24` en 4 subredes → s=2 (2²=4) → máscara `/26` → hosts = 2⁶−2 = 62 · bloque = 256−192 = **64**:
- `.0/26` → red .0, utilizables .1–.62, broadcast .63
- `.64/26` → red .64, utilizables .65–.126, broadcast .127
- `.128/26` → red .128, utilizables .129–.190, broadcast .191
- `.192/26` → red .192, utilizables .193–.254, broadcast .255

**VLSM** (Variable Length Subnet Mask): subredes de distinto tamaño para no desperdiciar. Procedimiento: ordenar requisitos **de mayor a menor**, asignar la subred más grande primero.

Ejemplo: disponemos de `192.168.10.0/24`; Dpto. A 100 hosts, Dpto. B 50, Enlace WAN 2 hosts:
1. A: /25 (126 hosts) → `192.168.10.0/25` (.0–.127).
2. B: /26 (62) → `192.168.10.128/26` (.128–.191).
3. WAN: /30 (2) → `192.168.10.192/30` (.192–.195). Sobra .196–.255 para futuro.

### 12.4 Diseñar Siguiendo una Estructura (Clase 12.4)
Diseño jerárquico y direccionamiento escalable:
- Agrupar por función/ubicación (VLAN por departamento; subred por piso/edificio).
- Reservar direccionamiento para crecimiento (no saturar /24 si se espera crecer a 300 hosts: planifica /23 o varias /24 agregadas).
- Documentar: plan de direccionamiento (red, máscara, gateway, VLAN, uso).
- Regla práctica: red punto a punto = /30 (o /31); LAN típica de oficina = /24; enlaces agregables con VLSM.

### 12.5 Preguntas del Módulo
Quiz de autoevaluación.

## Ejemplos

### Ejemplo 1 — AND paso a paso
¿`172.16.33.77` con máscara `255.255.240.0`, ¿en qué red está?
- 172 → 10101100 AND 11111111 = 172
- 16 → 00010000 AND 11111111 = 16
- 33 → 00100001 AND 11110000 = 00100000 = 32
- 77 → sin efecto (máscara .0)
Red: `172.16.32.0/20`. Rango: .32.0–.47.255; broadcast .47.255; hosts .32.1–.47.254 (4094 hosts).

### Ejemplo 2 — Subnetting completo de /22 en oficinas
`10.10.0.0/22` necesita 3 subredes (ventas 200, ingeniería 500, RRHH 100):
1. Ingeniería (mayor): necesita ≥500 → /23 (510 hosts) → `10.10.0.0/23`.
2. Ventas: ≥200 → /24 (254) → `10.10.2.0/24`.
3. RRHH: ≥100 → /25 (126) → `10.10.3.0/25`. Libre: .10.3.128/25 en adelante.

### Ejemplo 3 — VLSM en examen
Con `192.168.50.0/24`: LAN1 60 hosts, LAN2 28, LAN3 12, WAN1 2.
Orden: LAN1 /26 (62) .0–.63 · LAN2 /27 (30) .64–.95 · LAN3 /28 (14) .96–.111 · WAN1 /30 .112–.115. Verificar con AND que ninguna se solapa.

### Ejemplo 4 — ¿Ruta pública o privada?
Una pyme detrás de NAT usa `192.168.0.0/16`; el ISP le asigna una pública `81.x.x.x`. El router con NAT traduce las privadas→pública saliendo. Ningún router de Internet ruta RFC 1918 (caerían en "nulo").

## Ejercicios

1. **(Recordar)** Escribe red, broadcast y rango utilizable de `192.168.20.0/26`.
2. **(Recordar)** ¿Cuántas subredes y hosts/subred salen de un /24 dividido con 3 bits prestados?
3. **(Aplicar)** Haz el AND: IP `10.1.100.200`, máscara `255.255.224.0`. ¿Red y rango?
4. **(Aplicar)** Divide `172.30.0.0/16` en 8 subredes de igual tamaño. Da red, rango y broadcast de la 3ª.
5. **(Aplicar)** VLSM para `192.168.100.0/24`: LAN-A 100, LAN-B 50, LAN-C 25, WAN 2. Asigna en orden sin desperdicio.
6. **(Comprender)** ¿Por qué `.0` y `.255` en una /24 no son asignables a hosts?
7. **(Analizar)** Una red con máscara `/30` para un enlace punto a punto: ¿cuántos hosts útiles? ¿Por qué /31 existe (RFC 3021) y qué ahorra?
8. **(Analizar)** Detecta los errores: "La LAN del depto es `192.168.1.0/24` con gateway `192.168.1.255` y un servidor en `192.168.1.0`". Corrige.

<details>
<summary>Soluciones</summary>

1. Red .0, broadcast .63, utilizables .1–.62 (62 hosts).
2. 3 bits → 8 subredes (/27); hosts = 2⁵−2 = 30. Bloque = 32.
3. Máscara /19 (224 = 11100000). 100 → 01100100 AND 11100000 = 01100000 = 96 → red `10.1.96.0/19`; rango .96.1–.127.254; broadcast .127.255; hosts 8190.
4. 8 subredes → 3 bits → /19. Bloque 32 (en segundo octeto): .0/19, .32/19, **.64/19** (3ª), .96/19... Tercera: red `172.30.64.0/19`, rango 172.30.64.1–172.30.95.254, broadcast 172.30.95.255.
5. LAN-A /25 `192.168.100.0/25` (.0–.126 útiles) · LAN-B /26 `.128/26` (.129–.190) · LAN-C /27 `.192/27` (.193–.222) · WAN /30 `.224/30` (.225–.226). Libre desde .228.
6. `.0` = dirección de red (host bits a 0) y `.255` = broadcast (host bits a 1): ambas reservadas; asignarlas rompe el routing (la red y el broadcast de la propia subred).
7. /30 → 2 hosts útiles (red y broadcast consumen los otros dos; ideal enlaces P2P). /31 (RFC 3021) usa los 2 bits de host como direcciones punto a punto sin red/broadcast: ahorra mitad de direcciones en enlaces P2P.
8. Errores: gateway no puede ser `.255` (es el broadcast de la /24; un gateway allí rompe la red y los hosts) y el servidor `.0` es la red: no asignable. Correcto: gateway `192.168.1.1` (u otro libre), servidor `.2`–`.254`, p. ej. `192.168.1.10`.
</details>

## Tips
- Truco del "bloque": tamaño = 256 − octeto de la máscara (192→64, 224→32, 240→16, 248→8, 252→4).
- El examen pregunta "¿cuántas subredes/hosts?" con potencias de 2: memoriza 2^n−2.
- AND en binario siempre: para /26, enmascara con 192 (11000000) el último octeto.
- Orden VLSM: **de mayor a menor necesidad**, si no, fragmentar mal condena la tabla.
- RFC 1918 = privadas; 169.254 = APIPA; 127 = loopback; 224/4 = multicast. Siempre pregunta.
- Routers de Internet no propagan RFC 1918: sin NAT no sales a Internet (salvo CGN).

## Datos curiosos
- CIDR (RFC 1519, 1993) eliminó las clases y la máscara con clase, permitiendo agregación de rutas y frenando el crecimiento de las tablas de Internet; IPv6 eliminó el concepto de clase del todo.
- El 3 de febrero de 2011 la IANA asignó los últimos /8 disponibles: el mundo "se quedó sin IPv4" a nivel global (los RIRs agotaron después: APNIC 2011, RIPE 2012, ARIN 2015, LACNIC 2014, AFRINIC 2016).
- RFC 1918 (1996) reemplazó a los antiguos rangos del RFC 1597; la reserva de 192.168.x.x venía de entornos de entrenamiento/labs y por eso tu router casa casi siempre lo usa.

## Para tu libreta de apuntes
- AND: red = IP ∧ máscara. Misma red sii AND igual.
- Hosts = 2^h − 2 · Subredes = 2^s · Bloque = 256 − máscara.
- /24=.0 /25=.128 /26=.192 /27=.224 /28=.240 /29=.248 /30=.252.
- Clases: A 1-126/8 · B 128-191/16 · C 192-223/24 · D 224-239 multicast · E 240-255.
- RFC1918: 10/8 · 172.16-31/12 · 192.168/16. Especiales: 127 loopback · 169.254 APIPA · 100.64/10 CGN.
- VLSM: ordenar de mayor a menor.

## Preguntas de autoevaluación

1. ¿Cuál es el resultado de AND entre `192.168.1.200` y `255.255.255.192`? **Respuesta:** `192.168.1.192` (red /26).
2. ¿Cuántos hosts útiles tiene una /27? **Respuesta:** 30 (2⁵−2).
3. Indica dos rangos de IP privada y el tamaño de cada uno. **Respuesta:** 10.0.0.0/8; 172.16.0.0–172.31.255.255 (/12); 192.168.0.0/16.
4. Ves `169.254.10.5` en un PC: ¿qué significa? **Respuesta:** APIPA: no hubo DHCP; revisar infraestructura (o forzar DHCP/renew).
5. Al planificar VLSM, ¿en qué orden se asignan las subredes y por qué? **Respuesta:** De mayor a menor requisito: evita que un bloque pequeño fragmente el espacio e impida alojar luego una subred grande.
