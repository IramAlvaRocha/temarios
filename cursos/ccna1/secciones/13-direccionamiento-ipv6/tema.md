# 13 — Direccionamiento IPv6

> Duración del curso: 1h 04m · 8 clases
> Fuente: temario.md del curso

## Objetivos de aprendizaje
- Convertir entre hexadecimal y binario y leer la notación IPv6 (formato, compresión).
- Identificar los tipos de dirección IPv6: GUA, LLA, ULA, loopback, multicast, anycast.
- Configurar direcciones GUA y LLA estáticas en interfaces Cisco y hosts.
- Explicar el direccionamiento dinámico: SLAAC (EUI-64 y aleatorio), DHCPv6 (sin/stateful), y la generación EUI-64 para LLAs.
- Identificar las direcciones multicast (FF00::/8) y realizar subnetting básico en IPv6.

## Teoría

### 13.1 Sistema Hexadecimal (Clase 13.1)
- Base 16: dígitos `0-9` y `A-F` (A=10 … F=15). Agrupa **4 bits** (un *nibble*) por dígito.
- Conversiones rápidas: `A`=1010, `F`=1111, `FF`=11111111=255, `1A`=16+10=26.
- Un **byte** = 2 hex; una dirección IPv4 en hex (32 bits) = 8 hex; una IPv6 (128 bits) = 32 hex (8 grupos de 4).
- Tip: 256 = 0x100; las "mascaras" en IPv6 casi siempre son /64.

### 13.2 Formato y Tipos de Direcciones IPv6 (Clase 13.2)
**Formato:** 8 grupos de 4 hex separados por `:`; 128 bits: `2001:0db8:0000:0001:0000:0000:0000:0001`.

**Compresión (RFC 5952):**
1. Quitar ceros a la izquierda de cada grupo: `2001:db8:0:1:0:0:0:1`.
2. Sustituir UNA única secuencia de grupos consecutivos de 0 por `::`: `2001:db8:0:1::1`. `::` solo puede aparecer una vez.
- También permitido `::1` (loopback) y `::` (no especificada).

**Tipos principales:**

| Tipo | Prefijo | Análogo IPv4 | Uso |
|------|---------|--------------|-----|
| Global Unicast (GUA) | `2000::/3` | Pública | Enrutable en Internet |
| Link-Local (LLA) | `fe80::/10` | APIPA/169.254 (pero obligatoria y funcional) | Solo en el enlace; siempre presente |
| Unique Local (ULA) | `fc00::/7` (en la práctica fd00::/8) | Privada RFC 1918 | Privadas internas (no enrutables fuera) |
| Loopback | `::1/128` | 127.0.0.1 | Local |
| No especificada | `::/128` | 0.0.0.0 | Origen antes de asignar |
| Multicast | `ff00::/8` | 224.0.0.0/4 | Grupos (sin broadcast) |
| Anycast | no tiene bloque propio | — | Misma dirección en varios nodos; gana el más cercano |

Notas clave de examen:
- **No hay broadcast en IPv6**: todo es unicast o multicast.
- Una interfaz IPv6 tiene **siempre** LLA + (normalmente) GUA.
- La LLA sirve para el default gateway (el RA anuncia el link-local del router) y para protocolos vecinos.
- `ff02::1` = todos los nodos del enlace; `ff02::2` = todos los routers; `ff02::1:ffxx:xxxx` = solicited-node.
- Anycast: la **misma dirección GUA** se anuncia desde varias ubicaciones; el routing lleva al anuncio "más cercano" (usado en DNS raíz y CDNs). No tienes que configurar nada especial en el host; es una propiedad del routing.

### 13.3 Configurar Direcciones GUA y LLA Estáticas (Clase 13.3)
```
R1(config)# interface g0/0
R1(config-if)# ipv6 address 2001:db8:acad:1::1/64
R1(config-if)# ipv6 address fe80::1 link-local
R1(config-if)# no shutdown
R1(config)# ipv6 unicast-routing      ! global: el router enruta IPv6
```
- La LLA puede fijarse a `fe80::1` para facilitar el troubleshooting.
- `ipv6 unicast-routing` es **obligatorio** para que el router enrute IPv6 (sin él, solo hace ping, no enruta).
- Verificación: `show ipv6 interface brief`, `show ipv6 interface g0/0`, `ping 2001:db8:acad:1::1`.
- En Windows: `netsh interface ipv6 show addresses`, `ping -6`.

### 13.4 Direccionamiento Dinámico para GUAs IPv6 (Clase 13.4)
**SLAAC** (Stateless Address Autoconfiguration, RFC 4862):
1. El host aprende el prefijo /64 vía **RA** (Router Advertisement) o lo conoce.
2. Genera la parte de host:
   - **EUI-64 modificado**: toma la MAC (48b), la parte por la mitad, inserta `FF:FE` y **invierte el bit 7** (U/L) del primer byte: MAC `00:1B:44:11:3A:B7` → `021b:44ff:fe11:3ab7` → GUA `2001:db8:acad:1:021b:44ff:fe11:3ab7/64`.
   - *Nota (numeración de bits U/L e I/G):* la sección 08 numera los bits del primer octeto en orden **LSB-first** (bit 0 = I/G = 0x01; bit 1 = U/L = 0x02), mientras que esta sección habla de "bit 7" (y la literatura EUI-64) en orden **MSB-first** contando desde 1 (bit 7 = U/L = 0x02; bit 8 = I/G = 0x01). Se trata de los mismos bits: U/L = 0x02 (la máscara que se invierte en EUI-64), I/G = 0x01.
   - **Identificador aleatorio/estables de privacidad** (extensión RFC 4941/7217): preferido hoy por privacidad (evita tracking por MAC). Windows/macOS/Linux usan por defecto IDs aleatorios ("temporary addresses").
3. **DAD** verifica unicidad antes de asignar.

**DHCPv6:**
- **Sin estado (stateless, flag O=1 en RA):** SLAAC para la dirección + DHCPv6 solo para DNS/dominio.
- **Con estado (stateful, flag M=1):** el servidor DHCPv6 asigna dirección + DNS; el host no usa SLAAC para la dirección.
- Los routers envían RA periódicos (o solicitados por RS) anunciando prefijos y flags A (SLAAC), M, O.
- `ipv6 address autoconfig` en interfaces Cisco habilita SLAAC en el equipo; hosts modernos lo tienen por defecto.

### 13.5 Direccionamiento Dinámico para LLAs IPv6 (Clase 13.5)
- LLA siempre se autogenera al habilitar IPv6 (EUI-64 o aleatorio), o se fija manualmente (`fe80::1`).
- Rule: todo nodo IPv6 tiene LLA; sin LLA el enlace no funciona.
- EUI-64 para LLA: MAC → mitad + FFFE + flip bit U/L → `fe80::21b:44ff:fe11:3ab7` (ejemplo con la misma MAC).
- En Windows, `ipconfig /all` muestra "Link-local IPv6 Address". En Linux: `ip -6 addr`.

### 13.6 Direcciones IPv6 Multicast (Clase 13.6)
Formato: `ff` + 4 bits de flags + 4 bits de **scope** + 112 bits de grupo:
- Flags (4 bits): el más significativo es el bit **T** (transient): **0 = permanente** (bien conocidos, como `ff02::1`, y los solicited-node `ff02::1:ffxx:xxxx`), **1 = transitorio/temporal** (asignado para un fin concreto). Los grupos solicited-node son permanentes (flag 0).
- Scope (nibble): **1** = interfaz, **2** = enlace-local, **5** = site, **e** = global.
Grupos clave:
- `ff02::1` todos los nodos · `ff02::2` todos los routers · `ff02::5`/`ff02::6` OSPF · `ff02::a` EIGRP · `ff02::1:2` DHCPv6 · `ff02::fb` mDNS.
- **Solicited-node** `ff02::1:ff00:0/104` (+24 bits bajos de la IPv6): usada por NDP — cada dirección unicast/anycast genera una.
- IPv6 elimina el broadcast: multicast lo reemplaza.

### 13.7 Subnetting en IPv6 (Clase 13.7)
- Recomendación oficial: **/64 por LAN** (necesario para EUI-64 y SLAAC); los primeros 64 bits = prefijo de red, los últimos 64 = ID de interfaz.
- Subnetting: ampliar el prefijo: p. ej., un `/48` de ISP → 65.536 subredes /64 (16 bits de subred: bits 49–64).
- Ejemplo: de `2001:db8:acad::/48` puedes tener `2001:db8:acad:0000::/64` … `…:ffff::/64`. Lo habitual: /48 (cliente típico) → /64 LANs; pequeñas empresas: /56 o /60.
- No "ahorres" con /80: rompe SLAAC/EUI-64 y las buenas prácticas.
- Anycast de ejemplo del examen: asignar la misma GUA a 2+ routers; el cliente alcanza el "más cercano".

### 13.8 Preguntas del Módulo
Quiz de autoevaluación.

## Ejemplos

### Ejemplo 1 — Compresión correcta
`2001:0db8:0000:0000:0000:00ff:0000:0001`
- Quitar ceros izq: `2001:db8:0:0:0:ff:0:1` → una secuencia de ceros (grupos 3-5) → `2001:db8::ff:0:1`. ✔

### Ejemplo 2 — EUI-64 paso a paso
MAC del PC: `3C:97:0E:12:AB:CD`.
1. Mitades: `3C:97:0E` + `12:AB:CD`.
2. Insertar FF:FE: `3C:97:0E:FF:FE:12:AB:CD`.
3. Flip bit 7 del primer byte: `3C` = 00111100 → 00111110 = `3E` → `3E97:0EFF:FE12:ABCD`.
4. LLA: `fe80::3e97:eff:fe12:abcd`. GUA con prefijo `2001:db8:acad:1::/64`: `2001:db8:acad:1:3e97:eff:fe12:abcd`.

### Ejemplo 3 — Lab básico SLAAC en Packet Tracer
1. R1: `ipv6 unicast-routing`; G0/0 = `2001:db8:1::1/64`, `no shutdown`; lo demás en automático.
2. PC: Desktop → IP Configuration → IPv6: marcar "Auto Config". Obtendrá dirección autoconfigurada + gateway fe80:: de R1.
3. `ping` PC → `2001:db8:1::1` (éxito); `show ipv6 interface brief` en R1 lista la GUA y la LLA.

### Ejemplo 4 — Diseño de subnetting de empresa
ISP da `2001:db8:a000::/48`. Plan: VLAN10 Corp → `…:10::/64`, VLAN20 Invitados → `…:20::/64`, VLAN30 Voz → `…:30::/64`, gestión → `…:99::/64`. Capacidad 65.536 /64: crecimiento garantizado; documentar mapa en el plan de direccionamiento.

## Ejercicios

1. **(Recordar)** Convierte `2A` a binario y `10101111` a hex.
2. **(Recordar)** Comprime al máximo: `2001:0db8:0000:0000:00f0:0000:0000:0001`.
3. **(Aplicar)** Dada la MAC `00:25:96:AB:CD:EF`, calcula el EUI-64 y la LLA resultante; luego la GUA con prefijo `2001:db8:1::/64`.
4. **(Aplicar)** Configura en R1: routing IPv6 activo, G0/0 = `2001:db8:77::1/64`, LLA = `fe80::77`. Comandos completos.
5. **(Comprender)** ¿Qué flags RA activan SLAAC, DHCPv6 stateless y stateful?
6. **(Comprender)** ¿Por qué `ff02::2` no es broadcast y `ff02::1` sí se parece?
7. **(Analizar)** Un host obtiene IP IPv6 "temporal" que cambia cada día: ¿qué mecanismo/privacidad es y por qué existe?
8. **(Analizar)** Del /48 `2001:db8:bee::/48`, extrae los 4 primeros /64 y di cuántos /64 caben en total.

<details>
<summary>Soluciones</summary>

1. `2A` = 00101010; `10101111` = AF (10·8 + F=15).
2. `2001:db8::f0:0:0:1` → con un solo `::` para la única secuencia más larga de ceros (grupos 3–4): `2001:db8::f0:0:0:1`. (Si hay dos secuencias iguales, se comprime la primera — RFC 5952.)
3. EUI-64: 00:25:96 | FF:FE | AB:CD:EF → `0025:96ff:feab:cdef`; flip bit 7 de `00`→`02`: **`0225:96ff:feab:cdef`**. LLA `fe80::225:96ff:feab:cdef`. GUA `2001:db8:1:0:225:96ff:feab:cdef` (forma comprimida `2001:db8:1::225:96ff:feab:cdef`).
4. `configure terminal` → `ipv6 unicast-routing` → `interface g0/0` → `ipv6 address 2001:db8:77::1/64` → `ipv6 address fe80::77 link-local` → `no shutdown` → `end` → `copy run start`.
5. Flag **A** (autonomous) en RA → SLAAC (el host se auto asigna). Flag **O** (other) → DHCPv6 stateless (solo info extra: DNS). Flag **M** (managed) → DHCPv6 stateful (dirección + DNS desde servidor).
6. Ambas son multicast con scope ff02::/16 enlace-local: `ff02::1` = todos los **nodos** (todos los hosts escuchan); `ff02::2` = solo **routers**. No hay broadcast IPv6; multicast restringido es su sustituto.
7. **Privacy Extensions / temporary addresses (RFC 4941/7217)**: ID de interfaz aleatorio para que no se pueda rastrear al usuario por su MAC/EUI-64 en la red global; rota periódicamente (típ. 24 h).
8. `…:bee:0::/64`, `…:bee:1::/64`, `…:bee:2::/64`, `…:bee:3::/64`; caben 2^16 = **65.536** /64 en un /48.
</details>

## Tips
- `::` solo UNA vez por dirección; el examen penaliza compresiones inválidas.
- EUI-64 = MAC partida + `FFFE` + invertir bit 7. Flip: `00→02`, `02→00`, `08→0A`, `0A→08`, `18→1A`, etc.
- IPv6: **sin broadcast, sin ARP, sin checksum de cabecera** — los tres "no" favoritos del examen.
- `ipv6 unicast-routing` olvidado = el router no enruta IPv6 (pero sí hace ping).
- El default gateway de un host IPv6 suele ser la **LLA** del router (fe80::...).
- GUA routing en Internet: solo los prefijos del bloque **2000::/3** son asignables globalmente (p. ej. 2001::/16 es solo una parte de ese bloque).

## Datos curiosos
- IPv6 tiene 340 282 366 920 938 463 463 374 607 431 768 211 456 direcciones (~3.4×10³⁸): asignando un trillón por cada estrella observable aún sobran.
- Las direcciones **2001:db8::/32** están reservadas para documentación (RFC 3849): úsalas en ejemplos y nunca en producción.
- La IANA agotó el IPv4 el 2011, pero IPv6 se estandarizó antes (RFC 2460, 1998, actualizado por RFC 8200 en 2017): llevamos casi tres décadas diciendo "IPv6 es el futuro".
- Google reporta que cerca de la mitad del tráfico a sus servicios llega por IPv6 (dato orientativo; varía mucho por región y por fecha).

## Para tu libreta de apuntes
- Hex = 4 bits/dígito; IPv6 = 8×4 hex = 128 bits. `::` una vez.
- GUA 2000::/3 · LLA fe80::/10 · ULA fc00::/7 · loopback ::1 · multicast ff00::/8.
- EUI-64: MAC/2 + FF:FE + flip bit7.
- SLAAC (A) · DHCPv6 stateless (O) · DHCPv6 stateful (M) · DAD antes de usar.
- Multicast: ff02::1 nodos · ::2 routers · ff02::1:ffxx:xxxx solicited-node.
- /64 fijo por LAN · `/48` → 65.536 LANs.

## Preguntas de autoevaluación

1. Comprime `fe80:0000:0000:0000:0201:23ff:fe45:6789`. **Respuesta:** `fe80::201:23ff:fe45:6789`.
2. ¿Qué rango usan las link-local? **Respuesta:** `fe80::/10`.
3. ¿Qué hará un router Cisco con `ipv6 enable` en una interfaz sin comando `ipv6 address`? **Respuesta:** Crea solo la LLA (automática), no una GUA. (El routing requiere `ipv6 unicast-routing` y GUAs.)
4. ¿Qué mecanismo da dirección IPv6 sin servidor DHCP? **Respuesta:** SLAAC (RA + EUI-64 o identificador aleatorio).
5. ¿Qué dirección multicast escucha un router IPv6 para saber que debe atender RSs? **Respuesta:** `ff02::2` (todos los routers del enlace).
