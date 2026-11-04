# 11 — Configuración Inicial del Router

> Duración del curso: 16m · 3 clases
> Fuente: temario.md del curso

## Objetivos de aprendizaje
- Aplicar el procedimiento de configuración básica de un router Cisco: hostname, contraseñas, banner, cifrado, acceso a consola/vty.
- Configurar direcciones IP en interfaces del router y verificarlas.
- Configurar el gateway por defecto (en switches) y rutas/práctica básica de conectividad entre redes.

## Teoría

### 11.1 Configurar Ajustes Básicos (Clase 11.1)
Como en los switches (sección 3), pero el router además **enruta** entre redes. Secuencia fundamental en cualquier equipo IOS nuevo:

```
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# enable secret class123
R1(config)# no ip domain-lookup
R1(config)# service password-encryption
R1(config)# banner motd #Acceso restringido a personal autorizado#
R1(config)# line console 0
R1(config-line)# password cisco
R1(config-line)# login
R1(config-line)# exec-timeout 10 0
R1(config-line)# exit
R1(config)# line vty 0 4
R1(config-line)# password cisco
R1(config-line)# login
R1(config-line)# exec-timeout 10 0
R1(config-line)# exit
```

Puntos extra del router frente al switch:
- Trae las interfaces **en `shutdown`** → hay que activarlas con `no shutdown`.
- Las interfaces de router son capa 3 por defecto: se les asigna IP directamente.
- El **hostname** se usa en prompts y mensajes; sin él, todo es "Router".
- `enable secret` siempre preferible a `enable password`; en equipos modernos el hash es tipo 5 (MD5), 8 (PBKDF2-SHA256) o 9 (scrypt en IOS recientes).
- `line vty 0 4`: hasta 5 sesiones Telnet/SSH simultáneas; usa `0 15` en switches (16 líneas).

### 11.2 Configurar las Interfaces y el Gateway por Defecto (Clase 11.2)
```
R1(config)# interface gigabitethernet0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# description LAN-Oficina
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface gigabitethernet0/1
R1(config-if)# ip address 10.0.0.1 255.255.255.0
R1(config-if)# description Enlace-WAN
R1(config-if)# no shutdown
R1(config-if)# end
R1# show ip interface brief
R1# copy running-config startup-config
```

Verificación:
```
R1# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     192.168.1.1     YES manual up                    up
GigabitEthernet0/1     10.0.0.1        YES manual up                    up
```

**Gateway por defecto:**
- En **hosts**: gateway = IP del router en su subred (en PC: configuración IP).
- En **switches L2**: `ip default-gateway <IP del router>` en modo de configuración global (solo gestión remota; el switch no enruta).
- En el **propio router**: no existe "ip default-gateway" — se usa una **ruta estática por defecto**: `ip route 0.0.0.0 0.0.0.0 <next-hop>`.
- Conectividad entre redes: cada interfaz up/up con IPs en subredes coherentes; rutas (conectadas automáticas; estáticas si hay más redes detrás).

### 11.3 Preguntas del Módulo
Quiz de autoevaluación.

## Ejemplos

### Ejemplo 1 — Configuración completa desde cero (lab Packet Tracer)
Topología: PC-A (`192.168.1.10/24`) — SW1 — R1 — PC-B (`192.168.2.10/24`).
1. En R1 (por consola): hostname, enable secret, no ip domain-lookup, banner, vty/console.
2. `interface g0/0`: `ip address 192.168.1.1 255.255.255.0`, `no shutdown`.
3. `interface g0/1`: `ip address 192.168.2.1 255.255.255.0`, `no shutdown`.
4. `copy run start`. En PC-A gateway=192.168.1.1; PC-B gateway=192.168.2.1.
5. Verificación: `show ip int brief` (ambas up/up), `ping` PC-A→PC-B. Si falla: revisar gateways y máscaras primero.

### Ejemplo 2 — Salida de `show ip interface brief` con fallo
```
GigabitEthernet0/1     unassigned      YES unset  administratively down down
```
→ falta `no shutdown`. Si figura `down/down` (no "administratively"): no hay portadora (cable mal, equipo al otro lado apagado).

### Ejemplo 3 — Switch gestionado en otra subred
SW1 (`vlan 1` = `192.168.1.2/24`) debe gestionarse desde la LAN de administración `10.10.0.0/24` vía R1 (`192.168.1.1` / `10.10.0.1`): añade `ip default-gateway 192.168.1.1` en SW1. Sin él, SW1 responde solo dentro de su subred (`ping 10.10.0.x` desde el PC no recibiría respuesta).

### Ejemplo 4 — Tabla de troubleshooting rápido
| Síntoma | Causa probable | Verificación |
|---|---|---|
| `Protocol down` en interfaz | cable/puerto switch en shutdown | `show int g0/0`, luces del switch |
| `administratively down` | falta `no shutdown` | `show ip int brief` |
| No ping al gateway | máscara/gateway mal en PC | `ipconfig` |
| Ping gateway OK, remoto no | ruta/interfaz remota mal | `show ip route` en routers |
| Resolución de nombres falla | DNS | `ping` por IP vs por nombre |

## Ejercicios

1. **(Recordar)** ¿Por qué las interfaces de router vienen en `shutdown` y qué comando las activa?
2. **(Recordar)** ¿Qué diferencia hay entre `ip default-gateway` (switch) y una ruta por defecto en un router?
3. **(Aplicar)** Escribe los comandos para configurar en R1: hostname `RTR-CENTRAL`, secret `n3tAcad!`, banner `#Solo personal#`, y la interfaz G0/0 con `172.16.1.1/24` activa.
4. **(Aplicar)** Configura acceso por consola (clave `con`) y vty 0-4 (clave `vty`) con timeout de 5 minutos en R1.
5. **(Comprender)** ¿Qué indica cada letra de `show ip interface brief`: "OK?" y "Method"?
6. **(Analizar)** Tras configurar todo, el ping PC→PC remoto falla pero el ping al gateway funciona. Enumera 3 causas posibles ordenadas de más a menos probable.
7. **(Analizar)** ¿Por qué `enable secret` y no `enable password`? ¿Qué pasa si ambas existen (cuál se usa)?
8. **(Aplicar)** En Packet Tracer, configura dos routers conectados entre sí, cada uno con una LAN distinta, y deja trazado de ping extremo a extremo funcional (usando rutas estáticas o conectado directo).

<details>
<summary>Soluciones</summary>

1. Seguridad/por diseño: las interfaces apagadas evitan tráfico no deseado en equipos no configurados aún. Se activan con `no shutdown` (en `config-if`).
2. En un **switch L2**, `ip default-gateway` solo sirve para el plano de gestión del switch (a dónde manda sus propios paquetes IP de administración). En un **router**, la ruta por defecto (`ip route 0.0.0.0 0.0.0.0 next-hop`) enruta el tráfico **de usuarios** hacia destinos desconocidos.
3. `configure terminal` → `hostname RTR-CENTRAL` → `enable secret n3tAcad!` → `banner motd #Solo personal#` → `interface g0/0` → `ip address 172.16.1.1 255.255.255.0` → `no shutdown` → `end` → `copy run start`.
4. `line console 0` → `password con` → `login` → `exec-timeout 5 0`; `line vty 0 4` → `password vty` → `login` → `exec-timeout 5 0`; `end`.
5. `OK?` = IP válida (YES) o no (NO); `Method` = cómo se asignó: `manual` (configurada), `dhcp`, `unset`.
6. (a) Máscara o IP mal en alguno de los dos PCs (subred equivocada); (b) falta `no shutdown` en la interfaz remota o cable desconectado; (c) falta ruta hacia la red remota (o gateway por defecto mal en el PC remoto). Revisar `ipconfig` en ambos y `show ip int brief`/`show ip route` en R1.
7. `enable secret` está hasheada; `enable password` queda ofuscada débilmente con tipo 7. Si coexisten, **siempre se usa `enable secret`** (el `enable password` se ignora).
8. Verificar: `show ip int brief` up/up en ambos routers, rutas estáticas en ambos sentidos (o directamente conectadas), y ping PC-A↔PC-B con 0% loss.
</details>

## Tips
- Orden universal: **hostname → secret → domain-lookup/banner → vty/console → interfaces → `copy run start`**.
- `no shutdown` + luces del puerto: comprobadas en 5 segundos.
- Un router sin startup-config arranca en el *System Configuration Dialog*: responde `no`.
- `exec-timeout 0 0` = sin límite (peligroso en producción; útil en labs).
- vty sin contraseña → error "Password required, but none set": siempre `login` + `password`.
- En Packet Tracer, el modo Simulation muestra el ping cruzando routers salto a salto: excelente para validar mentalmente la ruta.

## Datos curiosos
- Los primeros routers Cisco (AGS, 1986) conmutaban ~1.500 paquetes/segundo; hoy hay enrutadores de núcleo con decenas de Tbps en backplanes. Cisco tiene más instalaciones de equipamiento de red que ningún otro fabricante en la historia.
- El banner legal es un requisito habitual en auditorías (indemnización/acceso autorizado): en EE. UU. han servido como evidencia de "aviso de sistema privado" en casos judiciales.
- "enable secret" tipo 9 (scrypt) llegó con IOS 15.3(3)M/IOS XE para superar MD5 (tipo 5, vulnerable a cracking por GPU).

## Para tu libreta de apuntes
- Básicos: hostname, enable secret, no ip domain-lookup, service password-encryption, banner motd, line con 0/vty + password + login + exec-timeout.
- Interfaz router: `interface g0/0` → `ip address A.B.C.D M.M.M.M` → `no shutdown`.
- Verificar: `show ip interface brief` (Status+Protocol = up).
- Switch gestionado: SVI + `ip default-gateway`. Router: `ip route 0.0.0.0 0.0.0.0 nh`.
- Dinámico vs manual: `Method: dhcp | manual | unset`.

## Preguntas de autoevaluación

1. ¿Qué comando activa una interfaz de router recién configurada? **Respuesta:** `no shutdown`.
2. ¿Dónde se aplica `ip default-gateway` y para qué sirve? **Respuesta:** En switches L2; permite la gestión remota desde otras subredes (no enruta usuarios).
3. ¿Qué comando muestra IPs y estado up/down de todas las interfaces? **Respuesta:** `show ip interface brief`.
4. Si coexisten `enable password` y `enable secret`, ¿cuál prevalece? **Respuesta:** `enable secret`.
5. Escribe la configuración mínima para dar acceso por consola con contraseña `lab` en R1. **Respuesta:** `line console 0` → `password lab` → `login` (→ `exec-timeout` opcional) → `end`.
