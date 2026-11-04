# 04 — Configurar IPs en Endpoints y Dispositivos de Red

> Duración del curso: 20m · 3 clases
> Fuente: temario.md del curso

## Objetivos de aprendizaje
- Configurar una dirección IPv4, máscara de subred y puerta de enlace (gateway) en un PC y en interfaces de router/switch.
- Explicar la configuración estática vs automática (DHCP) de los endpoints.
- Verificar la conectividad con `ping`, `ipconfig`/`ifconfig`/`ip addr` y comandos `show` de IOS.

## Teoría

### 4.1 Configurar IPs (Clase 4.1)

#### En endpoints (hosts)
Toda dirección IPv4 unicast lleva tres datos en el host:
1. **Dirección IP** (p. ej. `192.168.1.10`)
2. **Máscara de subred** (p. ej. `255.255.255.0` = `/24`): qué parte es red y qué parte es host.
3. **Puerta de enlace predeterminada** (default gateway, p. ej. `192.168.1.1`): por dónde salen los paquetes destinados a otra red (normalmente la interfaz del router).

Opcionalmente: **servidor DNS** (p. ej. `8.8.8.8` o el del ISP) para resolver nombres.

Dos métodos:
- **Estático:** se escribe a mano. Recomendado para servidores, impresoras, dispositivos de red. Riesgo: duplicados por error humano.
- **Dinámico (DHCP):** la red asigna IP/máscara/gateway/DNS automáticamente (protocolo DHCP, sección 16.6). Recomendado para PCs y móviles. Incluye un *lease* (concesión temporal) y renovación.

Cómo configurarlo en Windows: Panel de control → Centro de redes → Cambiar configuración del adaptador → Propiedades → IPv4 → usar IP estática o "obtener automáticamente". En Packet Tracer: PC → pestaña **Desktop** → **IP Configuration**.

En Linux/macOS: `ifconfig` / `ip addr` y fichero/red de configuración; macOS: Preferencias → Red → TCP/IP.

#### En dispositivos de red (Cisco IOS)
Interfaces de router:

```
R1> enable
R1# configure terminal
R1(config)# interface gigabitethernet0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# description LAN oficina
R1(config-if)# no shutdown              ! las interfaces de router vienen apagadas por defecto
R1(config-if)# exit
```

Switch de capa 2 (IP de gestión en VLAN, no en puertos):

```
SW1(config)# interface vlan 1
SW1(config-if)# ip address 192.168.1.2 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# ip default-gateway 192.168.1.1   ! necesario para gestionarlo desde otra red
```

Puntos críticos:
- Las interfaces de **router** arrancan en `shutdown` (administrativamente apagadas): hay que hacer `no shutdown`.
- La **VLAN 1** en switches viene habilitada por defecto: por eso la gestión básica usa `interface vlan 1`.
- `ip default-gateway` en un switch L2 solo es necesario para administración desde fuera de su subred; no enruta tráfico de usuarios.
- Verificación: `show ip interface brief`, `show running-config interface g0/0`.

### 4.2 Verificar que Estén Correctamente Conectados (Clase 4.2)
Comprobaciones ordenadas (método del CCNA):

1. **Capa física:** LEDs del puerto (verde/ámbar), cable correcto (directo PC→switch, cruzado switch→switch antiguo, o auto-MDIX en equipos modernos), `no shutdown`, `show ip interface brief` (Status/Protocol en `up`).
2. **Capa 2/3 en el propio equipo:** `show arp`, `show mac address-table`.
3. **Capa 3 desde el host:**

```
PC> ipconfig            ! Windows: IP, máscara, gateway
PC> ipconfig /all       ! además: MAC, DHCP, DNS
PC> ipconfig /release   ! libera lease DHCP
PC> ipconfig /renew     ! renueva lease DHCP
PC> ipconfig /flushdns  ! vacía caché DNS
PC> ping 192.168.1.1    ! prueba gateway (4 echo requests)
PC> ping 192.168.1.2    ! prueba otro host
PC> tracert 8.8.8.8     ! ruta hasta el destino (Windows)
```

Linux/macOS: `ip addr` (o `ifconfig`), `ping`, `traceroute`, `arp -a`.

En IOS: `ping 192.168.1.10`, `traceroute 192.168.1.10`, `show arp`.

Interpretación de ping:
- `! !!!!!` → respuestas (en IOS, `!` = éxito).
- `.....` → timeout (no hay respuesta). `U` → destino inalcanzable. En Windows: "Request timed out" vs "Destination host unreachable" (este último lo genera el propio host sin ruta ARP/gateway).
- **Ping al propio host** (`127.0.0.1`) verifica el stack TCP/IP; **ping al gateway** verifica la LAN; **ping a una IP remota** verifica el routing end-to-end; **ping por nombre** verifica DNS.

### 4.3 Preguntas del Módulo
Quiz de autoevaluación.

## Ejemplos

### Ejemplo 1 — Escenario completo en Packet Tracer
Topología: `PC1 — SW1 — R1 — PC2` (dos redes distintas: `192.168.1.0/24` y `192.168.2.0/24`).
1. PC1: IP `192.168.1.10`, máscara `255.255.255.0`, gateway `192.168.1.1`.
2. R1: interfaz G0/0 `192.168.1.1/24` `no shutdown`; interfaz G0/1 `192.168.2.1/24` `no shutdown`.
3. PC2: IP `192.168.2.20`, máscara `255.255.255.0`, gateway `192.168.2.1`.
4. Verificación: `ping` de PC1 a PC2 debe dar TTL≈127 (Windows, TTL inicial 128, 1 salto por R1) o 63 (Linux/macOS, TTL inicial 64, 1 salto por R1). En Packet Tracer, la respuesta es exitosa y se ve el paquete haciendo la ruta PC1 → R1 → PC2 en modo Simulation.

### Ejemplo 2 — Salida de ipconfig en Windows

```
C:\> ipconfig /all
Adaptador de Ethernet:
   Dirección física . . . . . . . : 00-60-2F-3A-1B-0C
   DHCP habilitado . . . . . . . : Sí
   Dirección IPv4. . . . . . . . : 192.168.1.10(Preferida)
   Máscara de subred . . . . . . : 255.255.255.0
   Concesión obtenida. . . . . . : miércoles, 4 de noviembre de 2026 08:12:33
   Puerta de enlace predeterminada: 192.168.1.1
   Servidores DNS. . . . . . . . : 192.168.1.1
```

### Ejemplo 3 — Falla de conectividad paso a paso
PC1 (`192.168.1.10`) no llega a `192.168.2.20`:
1. `ping 127.0.0.1` → OK (stack sano).
2. `ping 192.168.1.10` → OK (NIC propia).
3. `ping 192.168.1.1` → falla → revisar cable, `no shutdown` en G0/0, máscara/gateway del PC.
4. Si el gateway responde pero PC2 no: revisar G0/1 de R1, la máscara de PC2 y su gateway.
5. Si responde por IP pero no por nombre (`ping www.cisco.com` falla): problema de DNS, no de red.

### Ejemplo 4 — Gateway por defecto mal configurado
PC con IP `192.168.1.10/24` pero gateway `192.168.2.1`: navega por la LAN (ping al gateway correcto fallaría) pero no sale fuera. Síntoma típico: "navega el router pero no Internet". Corrección: gateway = IP de la interfaz del router en **la misma subred** del PC.

## Ejercicios

1. **(Recordar)** ¿Qué tres datos mínimos necesita un host IPv4 para comunicarse fuera de su subred?
2. **(Recordar)** ¿En qué pestaña de Packet Tracer configuras la IP de un PC y con qué comando la verificas en Windows?
3. **(Aplicar)** Configura en IOS la interfaz G0/0 de R1 con `10.0.0.1/8`, descripción `Enlace-LAN`, y actívala. Escribe los comandos completos.
4. **(Aplicar)** Un switch L2 debe gestionarse desde otra subred. ¿Qué configurar? (IP + comando extra + por qué).
5. **(Comprender)** ¿Qué significa que la interfaz de un router esté `up/up`? ¿Qué indica `administratively down/down`?
6. **(Analizar)** PC con `ping 8.8.8.8` OK pero `ping www.google.com` falla. ¿Dónde está el problema? Propón solución.
7. **(Analizar)** Enumera en orden los 4 pings diagnósticos clásicos y qué conclusión da cada uno.
8. **(Aplicar)** En Packet Tracer, monta PC–SW–R–PC en dos subredes, verifica con `show ip interface brief` y `ping` extremo a extremo, y captura la tabla que muestra qué interfaz está up/up.

<details>
<summary>Soluciones</summary>

1. IP, máscara de subred y puerta de enlace predeterminada (default gateway). (DNS como opcional para nombres.)
2. Pestaña **Desktop → IP Configuration**; se verifica con `ipconfig` (o `ipconfig /all`) en la terminal de Desktop/Command Prompt.
3. `enable` → `configure terminal` → `interface gigabitethernet0/0` → `ip address 10.0.0.1 255.0.0.0` → `description Enlace-LAN` → `no shutdown` → `end` → `copy run start`.
4. `interface vlan 1` (o VLAN de gestión) con IP + `no shutdown`, y **`ip default-gateway <IP del router>`**: sin gateway, el switch no sabe por dónde contestar a tráfico que viene de fuera de su subred.
5. `up/up` = capa física y protocolo de enlace operativos. `administratively down/down` = interfaz en `shutdown` (falta `no shutdown`); el primer `down` deshabilitado por administración.
6. El problema es DNS (resolución de nombres), no la conectividad IP. Solución: revisar/fijar el servidor DNS (`ipconfig /all`, configurar DNS correcto, probar `nslookup`).
7. (1) `ping 127.0.0.1` → stack TCP/IP local OK; (2) `ping <IP propia>` → NIC/driver OK; (3) `ping <gateway>` → conectividad local/LAN OK; (4) `ping IP pública remota` → routing end-to-end OK; (si falla solo por nombre → DNS).
8. Esperado: G0/0 y G0/1 en `up/up`; ping entre PCs exitoso (0% loss). En modo Simulation se observa ARP inicial y luego ICMP.
</details>

## Tips
- **Routers traen las interfaces apagadas → siempre `no shutdown`.** Los switches traen los puertos encendidos, pero el SVI `vlan 1` también necesita `no shutdown`.
- `show ip interface brief` antes que cualquier otro comando: detecta el 80% de los fallos básicos.
- En Windows: `ipconfig /release` y `/renew` para forzar DHCP; `/flushdns` para problemas de nombres.
- El examen pregunta qué indica "Destination host unreachable" generado por el propio PC (no tiene ARP/ruta) vs "Request timed out" (no contesta).
- TTL del ping: Windows inicia en 128 (restar saltos), Linux/macOS en 64. Útil para estimar saltos.
- IPs duplicadas → conflicto ("IP address conflict"); evitar asignando estáticas fuera del rango DHCP o con reservas.

## Datos curiosos
- La dirección `127.0.0.1` es de **loopback** (RFC 1122): todo el bloque `127.0.0.0/8` está reservado; los paquetes a loopback ni siquiera salen por la NIC — los procesa internamente el propio stack.
- **APIPA** (169.254.0.0/16, RFC 3927): cuando un PC no encuentra servidor DHCP, Windows se autoasigna `169.254.x.x`; verlo en `ipconfig` es señal de que falta DHCP o hay un fallo en la red local.
- ICMP nació como parte del diseño original de IP (RFC 792, 1981) para informar de errores; `ping` lo usa (echo request/reply).

## Para tu libreta de apuntes
- Host necesita: IP + máscara + gateway (+DNS). Estática = servidores; DHCP = clientes.
- Router: `interface g0/0` → `ip address X Y` → `no shutdown`.
- Switch L2: `interface vlan 1` → IP → `no shutdown` → `ip default-gateway`.
- Verificación: `show ip int br` | `ping 127.0.0.1 → propia → gateway → remota` | `ipconfig /all`.
- DHCP: `ipconfig /release|/renew|/flushdns`.

## Preguntas de autoevaluación

1. ¿Qué comando de Windows libera y renueva la concesión DHCP? **Respuesta:** `ipconfig /release` y `ipconfig /renew`.
2. En un switch de capa 2, ¿dónde se configura la IP de gestión y qué comando adicional permite alcanzarlo desde otra red? **Respuesta:** En el SVI (`interface vlan 1` o VLAN dedicada), más `ip default-gateway`.
3. ¿Por qué un ping a `127.0.0.1` triunfa aunque el cable esté desconectado? **Respuesta:** Es loopback: lo procesa internamente el stack TCP/IP sin salir por la NIC.
4. Ves `169.254.35.10` en `ipconfig`. ¿Qué ha pasado y qué compruebas? **Respuesta:** APIPA: no encontró DHCP. Revisar cable, switch, servidor DHCP (o conexión del router).
5. ¿Qué significa "administratively down" en `show ip interface brief`? **Respuesta:** La interfaz está en `shutdown`; falta `no shutdown`.
