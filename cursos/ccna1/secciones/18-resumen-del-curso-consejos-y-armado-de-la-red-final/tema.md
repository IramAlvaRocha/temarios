# 18 — Resumen del Curso, Consejos y Armado de la Red Final

> Duración del curso: 48m · 4 clases
> Fuente: temario.md del curso

## Objetivos de aprendizaje
- Integrar todo lo aprendido para planificar y montar una red pequeña de extremo a extremo.
- Explicar cómo escalar hacia redes más grandes y cómo comprobar las conexiones.
- Dominar los comandos de verificación y configuración en IOS y en hosts.
- Aplicar una metodología sistemática de troubleshooting por capas OSI.

## Teoría

### 18.1 Planificación de una Red Pequeña (Clase 18.1)
Flujo de diseño previo a tocar los equipos:
1. **Requisitos:** cuántos usuarios, qué servicios (web, correo, voz, ficheros), presupuesto, crecimiento (12–24 meses), SLA.
2. **Topología lógica:** subredes/VLANs por departamento, plan de direccionamiento (IP estáticas para servidores/impresora, DHCP para clientes), gateway y DNS.
3. **Equipamiento mínimo:** 1 router (salida a Internet + NAT), 1–2 switches de acceso, APs Wi‑Fi, opcionalmente firewall/UTM.
4. **Direccionamiento (ejemplo):** LAN `192.168.10.0/24` gateway `.1` (router) · switch gestión `.2` · servidores `.10–.19` · impresoras `.20–.29` · DHCP pool `.100–.199` · invitados VLAN `192.168.20.0/24` aislada.
5. **Seguridad desde el diseño:** segmentación, Wi‑Fi WPA2/WPA3, contraseñas cambiadas por defecto, gestión por SSH, backups de config.
6. **Documentación:** diagrama físico y lógico, plan IP, credenciales (en un gestor), inventario.

### 18.2 Escalabilidad a Redes más Grandes y Chequear las Conexiones (Clase 18.2)
Cuando la red crece, el diseño en capas lo hace manejable:
- **Jerarquía Cisco de 3 capas:** **acceso** (switches de usuario), **distribución** (agregación, VLANs/routing inter-VLAN, ACLs) y **núcleo (core)** (backbone de alta velocidad entre distribución). En redes pequeñas, acceso+distribución colapsados en uno.
- **Redundancia:** enlaces duales (Etherchannel, salto STP al siguiente enlace), fuentes redundantes, HA de router (HSRP/VRRP — fuera de alcance, parte 2), doble salida a Internet (dual-homing).
- **Segmentación:** más VLANs/subredes por edificio/piso; routing inter-VLAN en capa 3 (SVIs en multilayer switching) o router-on-a-stick.
- **Escalado de servicios:** DHCP relay (`ip helper-address`), servidores centralizados (DNS/DHCP/AD), controlador inalámbrico, monitorización (SNMP/Syslog/NetFlow).
- **Comprobación física:** `show cdp/lldp neighbors` (quién está y dónde), `show interfaces status`, prueba con `ping`/`traceroute`, verificación de velocidades y dúplex (`show interfaces` — negotiate 1G full).
- **Gestión de cambios:** mantenimiento programado, plan de rollback (config de respaldo), cambio fuera de horas.

### 18.3 Comandos para Usar en IOS y el Host (Clase 18.3)
**IOS (verif. rápido):**
```
show ip interface brief        show running-config      show startup-config
show version                   show mac address-table   show arp
show cdp neighbors [detail]    show interfaces [status] show ip route
show vlan brief                show spanning-tree (se estudia en la Parte 2 del curso)   show ip dhcp binding
```
**IOS (configuración):**
```
configure terminal · hostname · enable secret · service password-encryption
no ip domain-lookup · banner motd · line console 0 / line vty 0 15
interface vlan 1 · interface g0/0 · ip address · ipv6 address · no shutdown
ip default-gateway · copy running-config startup-config · reload
```
**Hosts:**
```
Windows: ipconfig [/all|/release|/renew|/flushdns] · ping · tracert · pathping · arp -a · netstat -an · route print · nslookup
Linux/macOS: ip addr · ip route · ip neigh · ping · traceroute · ss -tulpn · dig · cat /etc/resolv.conf
```

### 18.4 Métodos y Casos de Troubleshooting (Clase 18.4)
**Metodología general (7 pasos Cisco):**
1. Definir el problema (¿quién? ¿qué? ¿desde cuándo? ¿qué cambió?).
2. Recopilar información (síntomas, alcance).
3. Considerar posibilidades (hipótesis).
4. Crear plan de acción.
5. Implementar la solución.
6. Verificar la funcionalidad completa.
7. Documentar los resultados.

**Enfoques por capas OSI (descendente o en "divide y vencerás"):**
- **De abajo arriba:** empezar por capa 1 (cable/puertos/LEDs → `show int`), capa 2 (MAC/VLANs → `show mac`, duplex), capa 3 (IP/rutas → `ping`, `show ip route`, `traceroute`), capa 4+ (puertos → `telnet/ssh IP puerto`, `netstat`, Wireshark).
- **De arriba abajo:** cuando la aplicación falla pero la red parece bien: ¿puertos/DNS? Empezar por lo específico de la app.
- **Divide y vencerás:** corta el problema por la mitad (p. ej., ¿el problema es LAN o WAN? → ping al gateway primero).
- Regla de oro: **una variable cada vez** y documentar cada cambio.

**Casos típicos (y resolución):**
| Síntoma | Diagnóstico esperado |
|---------|---------------------|
| Sin LEDs en puerto | cable/puerto muerto, `shutdown`, distancia/daños |
| `up/down` protocolo | encapsulación/duplex mismatch, VLAN no permitida |
| Ping gateway OK, remoto no | ruta/NAT/ACL en salida, ISP caído |
| Ping por IP OK, por nombre no | DNS (`nslookup`, cambiar DNS) |
| Pérdida intermitente | duplex mismatch, errores CRC, cable marginal, EMI |
| Toda la LAN lenta | broadcasts excesivos, bucle L2/STP, enlace saturado |
| Solo un PC afectado | IP duplicada, NIC, drivers, malware |
| Velocidad 100 Mbps en vez de 1G | cable de 2 pares/Cat5, autonegociación |

## Ejemplos

### Ejemplo 1 — Montaje de la "red final" del curso en Packet Tracer
1. R1 (2911) + SW1 (2960) + 3 PCs (admin, ventas, invitados en VLANs distintas si estás en la parte 2; aquí misma LAN).
2. R1: `g0/0` `192.168.10.1/24` `no shutdown`; NAT básico fuera de alcance (usa 2911 con GE directa o nube de servicio DHCP).
3. SW1: hostname, secret, `interface vlan 1` `192.168.10.2/24` `no shutdown`, `ip default-gateway 192.168.10.1`.
4. PCs: IP manual o DHCP del router (`ip dhcp pool` en parte 2; en este curso, estática).
5. Verificación: `ping` cruzado, `show ip interface brief`, `copy run start` en ambos, y modo Simulation viendo ARP→ICMP.

### Ejemplo 2 — Sesión de troubleshooting en 5 minutos
Usuario: "No salgo a Internet". Preguntas: ¿otros PCs sí? → no → problema común (router/ISP/outage); sí → problema de ese PC.
`ping 127.0.0.1` ✓ → `ping 192.168.10.x` (propia) ✓ → `ping 192.168.10.1` ✓ → `ping 8.8.8.8` ✓ → `ping www.cisco.com` ✗ → **DNS**: `nslookup` falla → revisar `ipconfig /all` (¿DNS = 192.168.10.1?), `ipconfig /flushdns`, probar DNS público. Caso cerrado y documentado.

### Ejemplo 3 — Captura de verificación tras cambio (checklist)
Antes y después de añadir un switch nuevo:
- `show cdp neighbors` lista el nuevo SW2 ✓ · `show mac address-table` puebla tras tráfico ✓ · `ping` entre PCs de ambos switches ✓ · `copy run start` ✓ · diagrama actualizado ✓.

### Ejemplo 4 — Escalado: de 5 a 50 usuarios
5→50: montar diseño acceso/distribución: 2 switches de acceso (uno por planta) + 1 switch capa 3 (SVIs inter-VLAN) + router a Internet con firewall. DHCP relay a servidor central, VLANs 10/20/99 (users/voz/gestión), APs corporativos, monitorización SNMP. Cambios solo en capa de distribución/núcleo: cero parada en acceso.

## Ejercicios

1. **(Recordar)** Enumera las 7 fases de la metodología de troubleshooting Cisco.
2. **(Recordar)** Nombra los tres niveles de la arquitectura jerárquica Cisco y su función.
3. **(Aplicar)** Diseña el plan IP para una oficina: 30 PCs, 5 servidores, impresoras, VLAN invitados separada. Usa `192.168.x.0/24`.
4. **(Aplicar)** Resuelve: "PC conecta por cable pero `ipconfig` muestra 169.254.10.20; otros PCs navegan". Da el diagnóstico y 3 acciones.
5. **(Comprender)** ¿Por qué "de abajo arriba" en troubleshooting y cuándo conviene "divide y vencerás"?
6. **(Analizar)** Comparativa: red plana /24 con 200 nodos vs segmentada con 4 VLANs: colisiones/difusión, seguridad, rendimiento.
7. **(Analizar)** Un enlace negocia 100 Mbps full dúplex en lugar de 1 Gbps: causas de capa 1/2 y cómo verificarlas en IOS.
8. **(Aplicar)** Monta en Packet Tracer la topología R1—SW1—{PC1, PC2, SRV1} con plan IP completo, gateway, gestión del switch, verificación completa y guarda la config.

<details>
<summary>Soluciones</summary>

1. Definir problema → recopilar información → hipótesis → plan → implementar → verificar → documentar.
2. Acceso (conectar usuarios/dispositivos y aplicar políticas de puerto) · distribución (agregación: routing inter-VLAN, ACLs, políticas, agregación de enlaces) · núcleo (transporte de alta velocidad y disponibilidad entre distribución).
3. Ejemplo: LAN usuarios `192.168.10.0/24` (.1 GW, .2 SW, .10-.19 servidores, .20-.29 impresoras, DHCP .100-.199); invitados `192.168.20.0/24` aislada con su GW `.1`; gestión `192.168.99.0/24`. Reservar rangos; máscara /24; documentar en el plan IP.
4. APIPA: el PC no obtuvo DHCP. El resto de la red va bien → fallo local en la conversación DHCP: cable del puerto (mal par/LED), puerto switch en shutdown, servidor DHCP sin pool/licencia o MAC filtrada. Acciones: `ipconfig /release` + `/renew`; revisar puerto/cable/LED y `show interfaces status`; revisar leases/pool DHCP o poner IP estática temporal y escalar.
5. "De abajo arriba" respeta dependencias (sin capa física nada funciona) y desperdicia poco; "divide y vencerás" acorta cuando hay que aislar entre dos dominios probables (¿LAN o WAN?; ¿host o app?) cortando por la mitad con un ping conocido.
6. Plana: un único dominio de difusión (ARP/broadcast golpean a los 200 nodos; seguridad lateral nula — cualquier ARP spoof ve todo). Segmentada: difusión acotada por VLAN, políticas inter-VLAN en capa 3, menos tráfico de fondo y mejor rendimiento; escalable y auditable.
7. Causas: cable solo 2 pares o Cat5 real, cable demasiado largo (>100 m), conector dañado/mal ponchado, autonegociación fallida o speed/duplex fijado en un extremo y auto en el otro, puerto defectuoso. Verificar: `show interfaces g0/0` (línea de velocidad/duplex y contadores CRC/late collisions), `show interfaces status` y revisar negociación en ambos extremos; probar cable/puerto distinto.
8. Comprobar: G0/0 up/up `show ip int brief`; SVI `vlan 1` del switch con IP y `ip default-gateway`; PCs con IP/máscara/gateway coherentes; ping PC1↔SRV1 y PC1→gateway OK; config guardada `copy run start`; en Simulation se observa ARP + ICMP.
</details>

## Tips
- Troubleshooting Cisco = 7 pasos + una variable cada vez; el examen cae con "orden correcto de pasos".
- Orden de ping para aislar: **127.0.0.1 → propia IP → gateway → IP remota → nombre**. Memorízalo.
- `show cdp neighbors` revela quién es tu vecino Cisco y por qué puerto: oro para mapas mentales.
- Siempre `copy run start` tras verificar; un `reload` accidental sin guardar te devuelve a la config antigua.
- Antes de culpar al usuario: LEDs, cable, puerto — el 80% de "averías de red" es capa 1.
- Documenta: diagrama + plan IP + cambios con fecha. El siguiente que llega (que serás tú) lo agradece.

## Datos curiosos
- El modelo de **3 capas de Cisco** (acceso-distribución-núcleo) lo popularizó la guía de diseño de campus de Cisco en los 90 como respuesta a los problemas de las redes planas (difusión, escala, gestión); sigue siendo la plantilla mental de cualquier diseño moderno, incluidos los datacenters (CLOS/spine-leaf es su evolución).
- **CDP** (Cisco Discovery Protocol, propietario) frente a **LLDP** (IEEE 802.1AB, abierto): CDP descubre modelo/IOS/plataforma de vecinos; LLDP lo hacen también equipos no-Cisco.
- El término "hot swapping" y el "five 9s" (99,999% de disponibilidad ≈ 5 min/año caído) nacieron aquí: las redes diseñadas en capas con redundancia son las que se acercan a esa cifra.

## Para tu libreta de apuntes
- Diseño pequeño: requisitos → topología lógica → plan IP → seguridad → documentación.
- Escalado: acceso · distribución · núcleo; VLANs + inter-VLAN routing; redundancia; DHCP relay.
- Comandos estrella: `show ip int br` · `show mac address-table` · `show cdp neighbors` · `ping/tracert/ipconfig` · `copy run start`.
- Troubleshooting: 7 pasos · bottom-up · divide y vencerás · una variable cada vez · documentar.

## Preguntas de autoevaluación

1. Ordena los 7 pasos de troubleshooting Cisco (primero → último). **Respuesta:** Definir problema → recopilar info → hipótesis → plan → implementar → verificar → documentar.
2. Nombra las tres capas jerárquicas y qué ocurre en cada una. **Respuesta:** Acceso (usuarios), distribución (agregación/políticas/routing inter-VLAN), núcleo (backbone).
3. En la lista de pings de diagnóstico, ¿qué revela el último paso que los anteriores no? **Respuesta:** Resolución DNS (los anteriores solo prueban conectividad IP).
4. ¿Qué comando muestra los vecinos Cisco directamente conectados y sus puertos? **Respuesta:** `show cdp neighbors` (con `detail` para IOS/IP).
5. Red sin ping a Internet pero gateway responde: da dos causas y dos verificaciones. **Respuesta abierta:** causas: ruta/NAT/ISP caído, ACL/firewall bloqueando salida, gateway mal en el router; verificar: `show ip route` en R1, `ping`/traceroute desde el propio router hacia fuera y revisión de salida (NAT/ACL) e ISP.
