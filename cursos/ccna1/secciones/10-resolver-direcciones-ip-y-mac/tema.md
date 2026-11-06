# 10 — Resolver Direcciones IP y MAC

> Duración del curso: 17m · 3 clases
> Fuente: temario.md del curso

## Objetivos de aprendizaje
- Explicar qué problema resuelve ARP (IP → MAC) y cómo funciona solicitud/respuesta.
- Interpretar la caché ARP y sus entradas (`arp -a`, `show arp`).
- Describir cómo IPv6 descubre vecinos sin ARP: NDP (Neighbor Discovery Protocol, ICMPv6).

## Teoría

### 10.1 Protocolo de Resolución de Direcciones (ARP) (Clase 10.1)
**ARP** (Address Resolution Protocol, RFC 826) traduce **IPv4 → MAC** dentro de una red local. Necesario porque las tramas se direccionan por MAC y las aplicaciones usan IP.

Funcionamiento (escenario típico):
1. Host A (`192.168.1.10`) quiere enviar a `192.168.1.20`: no conoce su MAC.
2. Consulta su **caché ARP**: si existe entrada válida → usa la MAC.
3. Si no, envía un **ARP request** por **broadcast** (`FF:FF:FF:FF:FF:FF`): "¿Quién tiene 192.168.1.20? Dígamelo a 192.168.1.10 (MAC-A)".
4. Todos en el segmento procesan el broadcast; solo el dueño responde con un **ARP reply** (unicast a MAC-A): "192.168.1.20 está en MAC-B".
5. A guarda `IP → MAC` en su caché (entradas dinámicas, caducan en decenas de segundos a minutos según el SO — Linux: `gc_stale_time` = 60 s por defecto; Windows moderno aplica detección de inalcanzabilidad (estilo ND) sobre ARP, con tiempos mayores) y envía la trama.
- Si el destino está fuera de la subred: la caché ARP se usa para el **gateway**, no para el host remoto.
- **Proxy ARP:** un router responde ARP en nombre de hosts de otra red si tiene ruta (deshabilítalo/considera su implicación de seguridad; por defecto en Cisco está habilitado).
- **Gratuitous ARP:** ARP "yo soy IP X con MAC Y" anunciado sin pregunta — usado para detectar IPs duplicadas y actualizar tablas (failover VRRP/HSRP).

Comandos:

```
PC> arp -a                ! Windows/Linux: ver caché
PC> arp -d 192.168.1.20   ! borra entrada (Windows: arp -d *)
PC> arp -s 192.168.1.20 00-1a-2b-3c-4d-5e   ! entrada estática
R1# show arp              ! caché ARP del router
```
`show arp` típico en router:
```
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  192.168.1.1             -   000c.cf00.0101  ARPA   GigabitEthernet0/0
Internet  192.168.1.10            5   0050.7966.6801  ARPA   GigabitEthernet0/0
```
(`-` = entrada propia de la interfaz.)

Seguridad: **ARP spoofing/poisoning** (respuestas falsas → MITM). Mitigaciones: entradas estáticas en servidores, **Dynamic ARP Inspection** (DAI) + DHCP snooping en switches (fuera de este curso, parte 2/seguridad).

### 10.2 Descubrir Vecinos Usando IPv6 (Clase 10.2)
IPv6 **no usa ARP**. Lo sustituye **NDP** (Neighbor Discovery Protocol, RFC 4861), basado en **ICMPv6** + multicast:

| Función | Mensaje ICMPv6 | Tipo |
|---------|----------------|------|
| Solicitar router (host nuevo) | Router Solicitation (RS) | 133 |
| Anunciar router (periódico/respuesta) | Router Advertisement (RA) | 134 |
| Resolver IP → MAC (≈ ARP) | Neighbor Solicitation (NS) | 135 |
| Responder NS (≈ ARP reply) | Neighbor Advertisement (NA) | 136 |
| Redirigir (mejor ruta) | Redirect | 137 |

- **NS/NA**: equivalente funcional a ARP request/reply, pero dirigido a la **solicited-node multicast** `FF02::1:FFxx:xxxx` (no broadcast: solo los nodos cuya IP acaba en los mismos 24 bits escuchan) → muchísimo menos ruido.
- **RS/RA**: el host pide y el router anuncia prefijos, MTU, default gateway y flags (A: SLAAC, M: DHCPv6 stateful, O: DHCPv6 stateless). El RA también lleva el **Hop Limit** y el lifetime.
- **DAD** (Duplicate Address Detection): antes de usar una dirección, el nodo envía NS a su propia solicited-node; si nadie responde, la IP es única.
- En los hosts: `ip -6 neigh` (Linux), `netsh interface ipv6 show neighbor` (Windows). En IOS: `show ipv6 neighbors`.

### 10.3 Preguntas del Módulo
Quiz de autoevaluación.

## Ejemplos

### Ejemplo 1 — Flujo ARP completo (PC a PC en la misma LAN)
1. PC1 `ping 192.168.1.20`: consulta caché (vacía).
2. Emite ARP request broadcast: `Who has 192.168.1.20? Tell 192.168.1.10`.
3. PC2 responde unicast: `192.168.1.20 is at 00:50:79:66:68:01`.
4. PC1 guarda y envía ICMP echo request. En `arp -a` aparece la entrada; en el switch, `show mac address-table` ya la conocía.

### Ejemplo 2 — ARP para el gateway (destino remoto)
PC1 (`192.168.1.10`) → `8.8.8.8`: ARP resuelve la MAC de `192.168.1.1` (gateway), no de 8.8.8.8. La trama lleva MAC dst = gateway; el paquete IP lleva dst = 8.8.8.8. Prueba: `arp -a` tras un ping externo mostrará solo la entrada del gateway.

### Ejemplo 3 — NDP en IPv6 (DAD + SLAAC)
Un PC arranca con IPv6:
1. Genera link-local `fe80::xxxx` desde EUI-64 o aleatoria.
2. **DAD**: envía NS a `FF02::1:FFxx:xxxx` preguntando por su propia dirección; sin respuesta → válida.
3. Envía **RS** a `FF02::2` (todos los routers).
4. Router responde **RA** con prefijo `2001:db8:acad:1::/64` y flag A=1 → el PC construye su GUA (2001:db8:acad:1::xxxx) y rutas. Gateway por defecto = link-local del router (fe80::...).

### Ejemplo 4 — ARP spoofing y defensa básica
Atacante envía ARP replies falsos: "el gateway está en MI mac". Víctimas reenvían el tráfico al atacante (MITM). Defensa en switch gestionado: DHCP snooping (lista IP↔MAC↔puerto de confianza) + Dynamic ARP Inspection (descarta ARP no válidos). En hosts: revisar `arp -a` buscando MACs duplicadas para IPs críticas.

## Ejercicios

1. **(Recordar)** ¿Qué resuelve ARP y con qué direcciones trabaja (origen/destino de la solicitud)?
2. **(Recordar)** ¿Qué protocolo sustituye a ARP en IPv6 y sobre qué protocolo viaja?
3. **(Aplicar)** Escribe el diálogo ARP completo (request/reply) entre A `10.0.0.5` (MAC AAAA) y B `10.0.0.9` (MAC BBBB).
4. **(Aplicar)** En Windows, ¿qué comandos muestran y vacían la caché ARP? ¿Y en un router Cisco?
5. **(Comprender)** ¿Por qué ARP usa broadcast y NDP usa multicast solicited-node? Ventaja de NDP.
6. **(Analizar)** Sin ARP (o con caché inconsistente), un host no llega al gateway. Síntomas y pasos de diagnóstico.
7. **(Analizar)** ¿Qué es Gratuitous ARP y para qué se usa? Da dos usos.
8. **(Aplicar)** En Packet Tracer, borra la tabla ARP de un PC, lanza un `ping` y usa Simulation para ver el ARP request/reply antes de los ICMP.

<details>
<summary>Soluciones</summary>

1. Resuelve IP→MAC en la LAN. La solicitud (request) va de IP origen propia a broadcast L2 (FF:FF:FF:FF:FF:FF) preguntando por una IP concreta; la respuesta (reply) va en unicast.
2. NDP (Neighbor Discovery Protocol), basado en ICMPv6 (RS/RA/NS/NA/Redirect).
3. A → broadcast: "Who has 10.0.0.9? Tell 10.0.0.5 (AAAA)". B → unicast a AAAA: "10.0.0.9 is at BBBB". A cachea y envía su tráfico a BBBB.
4. Windows: `arp -a` (ver), `arp -d *` (vaciar), `arp -s` (estática). Cisco: `show arp`; no hay "clear arp" general salvo `clear arp-cache` (en routers sí existe `clear arp-cache`).
5. ARP no tenía otra opción: necesidad de "preguntar a todos" en difusión. NDP usa multicast solicited-node: solo reciben la consulta los nodos con los mismos 24 bits finales de IP → menos carga en CPU para el resto.
6. Síntomas: ping al gateway falla con "Destination host unreachable" o timeouts; `arp -a` muestra entrada incompleta/errónea o ausente. Pasos: `arp -d`, re-ping, verificar IP/máscara/gateway, `show arp` en router, y descartar duplicados/loops físicos.
7. ARP gratuito: el emisor anuncia su propia IP→MAC sin que nadie pregunte. Usos: detección de IP duplicadas y actualización de tablas/conmutación por error en alta disponibilidad (HSRP/VRRP, failover de NIC).
8. En Simulation: primero ARP request (broadcast) inundado por el switch, respuesta del destino, y solo después los 4-5 echo request/reply ICMP.
</details>

## Tips
- ARP = "¿Quién TIENE esta IP?"; respuesta = "YO la tengo, y mi MAC es...".
- Destino remoto → ARP solo para el **gateway**; nunca para IPs de otras subredes.
- NDP usa multicast → no satura la LAN como ARP; el examen compara ambos.
- `show arp` en router y `arp -a` en host son las dos ventanas que pregunta el CCNA.
- Seguro: entradas estáticas en servidores críticos o DAI; gratis: `arp -a` periódico no es mala costumbre.
- Las entradas ARP caducan (desde segundos en Windows moderno hasta minutos; en Cisco, el `arp timeout` por defecto son 4 horas / 14400 s para entradas completas).

## Datos curiosos
- ARP es anterior a IP en cuanto a formalidad: RFC 826 (noviembre 1982), por David C. Plummer. Ya el ARPANET necesitaba un puente entre direccionamiento lógico y físico.
- El esquema 48 bits de MAC hace improbable (pero no imposible) la colisión de direcciones; los fabricantes registran rangos OUI ante el IEEE Registration Authority (un bloque MA-L cuesta US$ 3.585 según la tarifa vigente).
- NDP está definido en RFC 4861/4862 (2007) y sustituye también a la detección de encaminadores ICMP y autoconfig de IPv6 antiguos (RFC 1970).
- IPv4 puede operar sin ARP solo en medios punto a punto (PPP) donde la MAC no existe: ahí IP funciona "directo" sobre el enlace.

## Para tu libreta de apuntes
- ARP: IP→MAC · request broadcast · reply unicast · caché caduca.
- Destino fuera de subred → ARP al gateway.
- NDP (IPv6): ICMPv6 133 RS · 134 RA · 135 NS · 136 NA · 137 Redirect · DAD.
- Solicited-node multicast: FF02::1:FFxx:xxxx.
- Comandos: `arp -a`, `arp -d`, `show arp`, `show ipv6 neighbors`.

## Preguntas de autoevaluación

1. ¿Qué mensaje ARP se envía por broadcast y cuál por unicast? **Respuesta:** Request = broadcast; reply = unicast.
2. En IPv6, ¿qué reemplaza al ARP y con qué mensaje verifica unicidad de una dirección? **Respuesta:** NDP (ICMPv6); DAD usa Neighbor Solicitation.
3. ¿Por qué un PC que hace ping a 8.8.8.8 no ARPa 8.8.8.8? **Respuesta:** Está fuera de su subred; envía la trama a la MAC de su gateway.
4. ¿Qué comando muestra la caché ARP en un router Cisco? **Respuesta:** `show arp`.
5. Define Gratuitous ARP y cita un uso. **Respuesta:** Anuncio no solicitado IP→MAC; detección de duplicados o actualización en failover (HA).
