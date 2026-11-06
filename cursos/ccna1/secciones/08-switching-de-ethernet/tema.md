# 08 — Switching de Ethernet

> Duración del curso: 42m · 5 clases
> Fuente: temario.md del curso

## Objetivos de aprendizaje
- Desglosar la trama Ethernet (campos, longitudes, tipos comunes).
- Explicar el direccionamiento MAC: OUI, bits I/G y U/L, direcciones unicast/multicast/broadcast.
- Interpretar y gestionar la tabla de direcciones MAC (`show mac address-table`).
- Comparar los métodos de reenvío/almacenamiento: store-and-forward, cut-through, fragment-free, y sus implicaciones (latencia, errores, buffers).

## Teoría

### 8.1 Tramas de Ethernet (Clase 8.1)
Ethernet (IEEE **802.3**) es la tecnología de red cableada dominante. Formatos de trama:

**Ethernet II (DIX)** — el usado en IP:
```
| 7 B preámbulo | 1 B SFD | 6 B MAC dst | 6 B MAC src | 2 B EtherType | 46–1500 B datos | 4 B FCS |
```
- EtherTypes comunes: `0x0800` IPv4 · `0x0806` ARP · `0x86DD` IPv6 · `0x8100` VLAN tag 802.1Q (inserta 4 B tras la MAC src: TPID 0x8100 + TCI con VLAN ID de 12 bits).
- **Tamaño mínimo 64 B** (sin contar preámbulo/SFD): 6+6+2+46+4. **Máximo 1518 B** (1522 con tag VLAN). Menor de 64 → *runt*; mayor de 1518 → *giant*.

**Trama IEEE 802.3 clásica:** el campo tipo se convierte en *Length* y se añade cabecera **LLC (802.2)** + opcionalmente **SNAP** para indicar el protocolo (así se transportaban protocolos propietarios). En la práctica, hoy todo es Ethernet II para IP; 802.3 LLC sobrevive en algunos protocolos de red industrial/SAP.

### 8.2 Conceptos de las Direcciones MAC (Clase 8.2)
- **48 bits** (6 bytes), hexadecimales: `00:1B:44:11:3A:B7` o `001b.4411.3ab7` (formato Cisco con puntos cada 4 dígitos).
- Estructura: **OUI** (3 primeros bytes, asignados por IEEE al fabricante) + **NIC** (3 últimos, únicos por tarjeta).
- Bits especiales (primer byte):
  - **I/G (Individual/Group)**, bit 0 del primer octeto: 0 = unicast/individual, 1 = multicast/group.
  - **U/L (Universal/Local)**, bit 1: 0 = universal (OUI real), 1 = administrada localmente (LAA, usable por software/hypervisores).
  - *Nota (numeración de bits):* esta sección numera los bits del primer octeto en orden **LSB-first** (bit 0 = I/G = 0x01; bit 1 = U/L = 0x02). La sección 13 (EUI-64) numera en orden **MSB-first** contando desde 1 (bit 7 = U/L = 0x02; bit 8 = I/G = 0x01). Son los mismos bits (máscaras 0x02 / 0x01); solo cambia la convención de numeración.
- Tipos: **unicast** (I/G=0), **multicast** (I/G=1; IPv4 multicast → `01:00:5E:00:00:00`–`01:00:5E:7F:FF:FF`), **broadcast** (`FF:FF:FF:FF:FF:FF`).
- MAC plano: no jerárquico, válido solo en el segmento local; no se "routea". Cambiar de subred = nuevas MAC por salto, la IP se conserva.
- Dirección MAC de un switch/router de interfaces: `show interfaces` / `show version` (la base MAC del chasis en Cisco deriva las de las interfaces).

### 8.3 Tabla de Direcciones MAC (Clase 8.3)
El switch aprende MACs dinámicamente:
1. Llega una trama por el puerto X con MAC origen M → el switch anota `M → X` en la **tabla CAM** (Content Addressable Memory).
2. Consulta la tabla para la MAC destino:
   - **Conocida (known unicast):** reenvía solo por el puerto correspondiente.
   - **Desconocida (unknown unicast):** inunda (*flooding*) por todos los puertos del dominio menos el de entrada.
   - **Broadcast/Multicast:** inunda (salvo snooping/IGMP para multicast).
3. Entradas dinámicas caducan por inactividad (~300 s por defecto en switches Cisco); también existen entradas **estáticas** (`mac address-table static ...`, poco habituales).

Comandos:

```
SW1# show mac address-table
SW1# show mac address-table dynamic
SW1# show mac address-table address 0050.7966.6801
SW1# show mac address-table count
SW1# clear mac address-table dynamic   ! para pruebas
```

Salida típica:
```
          Mac Address Table
-------------------------------------------
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0050.7966.6801    DYNAMIC     Fa0/1
   1    000c.cf39.5a2b    DYNAMIC     Fa0/2
```

Notas de examen:
- El switch **inunda unicast desconocido** pero **no** lo reenvía por el puerto origen.
- La tabla CAM es finita; ataques de **MAC flooding** llenan la tabla y convierten al switch en "hub" → mitigación: port-security (sección 17).
- `mac address-table aging-time` ajusta el envejecimiento.

### 8.4 Métodos de Reenvío (Clase 8.4)
Cómo decide el switch cuándo empezar a reenviar una trama:

| Método | Cuándo reenvía | Latencia | Detección de errores | Uso típico |
|--------|----------------|----------|---------------------|------------|
| **Store-and-forward** | Tras recibir la trama completa y validar FCS | Mayor (proporcional al tamaño) | Sí (descarta tramas corruptas/runts/giants) | Por defecto en switches Cisco actuales |
| **Cut-through** | Tras leer la MAC destino (primeros 6 B tras preámbulo) | Mínima (~pocos µs) | No (puede propagar basura) | Baja latencia/HPC (con variantes) |
| **Fragment-free** (híbrido) | Tras los primeros 64 B | Media | Mejor que cut-through (cubre la zona de colisiones) | Intermedio, casi en desuso |

Detalles:
- **Store-and-forward** memoriza la trama entera en el buffer, comprueba CRC y longitud, y reenvía solo tramas sanas; obliga a almacenar tramas jumbo.
- **Cut-through** reenvía casi de inmediato (latencia constante, p. ej. <10 µs) — útil en entornos sensibles (trading, HPC). Riesgo: propaga tramas con CRC erróneo aguas abajo.
- Adaptativo (adaptive switching): algunos switches arrancan en cut-through y cambian a store-and-forward si la tasa de errores crece.
- Además: **buffering** (colas de salida; puertos congestionados encolan tramas) y **symmetric/asymmetric switching** (mismo u distinto ancho de banda entrada/salida; p. ej. 100→1000 Mbps), **memoria compartida vs por puerto**.
- **Autonegociación Ethernet/802.3u**: velocidad/duplex (10/100/1000 y half/full); duplex mismatch produce colisiones tardías y degradación.

### 8.5 Preguntas del Módulo
Quiz de autoevaluación.

## Ejemplos

### Ejemplo 1 — Aprendizaje de la tabla CAM (3 pasos)
PC-A (`0050....`) en Fa0/1 envía a PC-B (`000c.cf...`) en Fa0/3.
1. Trama entra por Fa0/1 → switch aprende `0050.xxxx → Fa0/1`.
2. Busca destino en la tabla: no existe → inunda por Fa0/2, Fa0/3, ... (nunca por Fa0/1).
3. PC-B responde → switch aprende `000c.cfxx → Fa0/3`; a partir de ahora el tráfico A↔B va solo por Fa0/1 y Fa0/3.

### Ejemplo 2 — `show mac address-table` antes y después
Recién arrancado un switch limpio: tabla vacía. Tras 30 s de tráfico:
```
Sw1# show mac address-table dynamic
        Mac Address Table
-------------------------------------------
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    aabb.cc00.0101    DYNAMIC     Fa0/1
   1    aabb.cc00.0201    DYNAMIC     Fa0/2
```
Si desconectas PC de Fa0/1 y pasan 5 min, la entrada desaparece (aging 300 s por defecto).

### Ejemplo 3 — Store-and-forward vs cut-through en un broker financiero
Cluster HPC con 10GbE y mensajes de 64–256 B: cut-through reduce la latencia en microsegundos por salto. Red de campus con clientes Windows: store-and-forward evita propagar tramas corruptas a todos los puertos (mejor integridad; la latencia extra de una trama de 1518 B a 1 Gbps es ~12 µs, despreciable).

### Ejemplo 4 — Ataque MAC flooding (concepto)
De repente `show mac address-table count` alcanza el máximo (p. ej. 8192) por un torrente de MACs falsas: el switch inunda todo → el atacante con un sniffer ve tráfico de todos. Mitigación (sección 17): `switchport port-security maximum 2`, violación `shutdown`, y aprendizaje sticky.

## Ejercicios

1. **(Recordar)** Escribe el EtherType de IPv4, IPv6 y ARP.
2. **(Recordar)** ¿Qué significa OUI y cuántos bytes ocupa?
3. **(Aplicar)** Dada la MAC `01:00:5E:10:20:30`, indica: ¿unicast o multicast? ¿Por qué? Cita otro ejemplo de MAC multicast.
4. **(Aplicar)** Un switch recién encendido recibe una trama a `AAAA.BBBB.CCCC` desconocida. ¿Qué hace? ¿Y si llega por Fa0/1 con MAC origen `AAAA.BBBB.1111`?
5. **(Comprender)** Diferencia entre `show mac address-table` y `show arp`: ¿qué capa/dirección usa cada uno?
6. **(Aplicar)** Escribe los comandos para ver la tabla MAC, la entrada concreta de `0050.7966.6801` y el contador total.
7. **(Analizar)** Comparación store-and-forward vs cut-through: tabla con latencia, detección de errores y caso de uso.
8. **(Analizar)** Encapsulación completa: paquete IP de 1000 B sale por Ethernet. ¿Cuántas capas/encabezados atraviesa y cuál es el tamaño de la trama en el medio?

<details>
<summary>Soluciones</summary>

1. IPv4 = 0x0800; IPv6 = 0x86DD; ARP = 0x0806.
2. Organizationally Unique Identifier: 3 primeros bytes (24 bits) asignados por IEEE al fabricante; los 3 últimos (NIC) los asigna el fabricante para unicidad.
3. Multicast: el primer byte `01` tiene activado el bit I/G (01 = 00000001). Otras: `01:80:C2:00:00:00` (STP, BPDUs), `01:00:5E:xx` (IPv4 multicast), `33:33:xx` (IPv6 multicast).
4. Inunda la trama por todos los puertos de la VLAN menos el de entrada (unknown unicast flooding). Además aprende `AAAA.BBBB.1111 → Fa0/1`. Cuando PC-B conteste, aprenderá su puerto y dejará de inundar.
5. `show mac address-table`: tabla CAM capa 2 (MAC → puerto); `show arp`: caché capa 3 (IP → MAC) del dispositivo (router/L3) o host.
6. `show mac address-table` · `show mac address-table address 0050.7966.6801` · `show mac address-table count`.
7. Store-and-forward: reenvía tras recibir toda la trama y validar CRC → latencia ∝ tamaño, detecta runts/giants/CRC; ideal campus. Cut-through: reenvía tras leer MAC destino (6 B: primeros 6 bytes tras el preámbulo = MAC destino) → latencia mínima fija, no detecta errores; ideal HPC. Fragment-free: 64 B iniciales, compromiso.
8. El paquete IP viaja dentro de una trama Ethernet: la app genera datos → TCP (+20-60 B) → IP (+20-60 B, el paquete de 1000 B ya incluye cabecera IP) → Ethernet (+18 B: MAC dst 6 + src 6 + EtherType 2 + FCS 4) → trama total = 1000 + 18 = **1018 B** (sin padding adicional); si el payload de la trama < 46 B se rellenaría (aquí no aplica). Está dentro de los límites 64–1518 B.
</details>

## Tips
- Formato Cisco MAC: `xxxx.xxxx.xxxx` (3 grupos de 4 hex). En PC: `xx-xx-xx-xx-xx-xx`. Es la misma dirección.
- **Unknown unicast flooding** es el comportamiento esperado; no es un error.
- Aging por defecto = **300 segundos** en Cisco.
- El examen pide: "switch aprende por MAC **origen**, conmuta por MAC **destino**" — memorízalo tal cual.
- Broadcast = FF:FF:FF:FF:FF:FF siempre se inunda dentro de la VLAN.
- Port-security contra MAC flooding está fuera de esta sección pero ligado; el CCNA lo une con seguridad.
- EtherType 0x8100 = 802.1Q tag: necesario para VLANs trunk (fuera de este curso, parte 2).

## Datos curiosos
- Las direcciones MAC se llamaron originalmente **MAC-48** y luego **EUI-48** (Extended Unique Identifier). Existe EUI-64 para autoconfiguración de IPv6 (ver sección 13).
- La CAM del switch es memoria especializada de acceso en un ciclo de reloj, diseñada para búsquedas paralelas masivas: de ahí la conmutación por hardware (tabla CAM), del orden de sub-microsegundo, sin CPU general.
- El ataque **MAC flooding** fue descrito públicamente en los 90s (camisetas y conferencias); la respuesta de la industria fue port-security y DHCP snooping/802.1X.
- 01:80:C2:00:00:00 es la MAC destino de las BPDUs STP: por eso `show mac address-table` no muestra BPDUs aprendidas (no son "aprendidas": el switch las consume/procesa localmente).

## Para tu libreta de apuntes
- Trama II: preámbulo8 SFD1 | dstMAC6 | srcMAC6 | EtherType2 | payload46–1500 | FCS4.
- EtherTypes: 0800 IPv4 · 0806 ARP · 86DD IPv6 · 8100 VLAN.
- Aprendizaje: por MAC **origen**; conmutación: por MAC **destino**; aging 300 s.
- Desconocido/broadcast → flood (todos menos el de entrada).
- Store-and-forward (CRC OK) vs cut-through (rápido, sin CRC).
- I/G bit: 0 unicast / 1 multicast · U/L: 0 universal.

## Preguntas de autoevaluación

1. ¿Cómo aprende un switch las ubicaciones de las MACs? **Respuesta:** Mirando la MAC **origen** de las tramas recibidas por cada puerto y asociándola en la tabla CAM.
2. Una trama con MAC destino desconocida se reenvía por… **Respuesta:** Todos los puertos de la VLAN excepto el de entrada (flooding).
3. ¿Qué método de reenvío comprueba el FCS antes de reenviar? **Respuesta:** Store-and-forward.
4. EtherType de 802.1Q: **Respuesta:** 0x8100.
5. ¿Cuánto tiempo vive por defecto una entrada dinámica en la tabla MAC de un switch Cisco? **Respuesta:** 300 segundos (5 minutos).
