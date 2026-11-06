# 03 — Cisco IOS

> Duración del curso: 1h 07m · 6 clases
> Fuente: temario.md del curso

## Objetivos de aprendizaje
- Explicar qué es Cisco IOS y sus modos de comando (EXEC de usuario, EXEC privilegiado, configuración global, interfaces, líneas).
- Navegar y usar ayuda sensible al contexto, atajos y teclas especiales en la CLI.
- Configurar hostname, contraseñas, banner, cifrado de contraseñas y acceso por consola.
- Guardar la configuración (running-config vs startup-config) y verificar con comandos `show`.

## Teoría

### 3.1 Introducción a Cisco IOS (Clase 3.1)
**Cisco IOS** (Internetwork Operating System) es el sistema operativo de los routers y switches Cisco. Se administra por:
- **CLI** (command-line interface, la vía principal y la que usa el examen),
- Consola (puerto físico RJ-45/USB), Telnet (obsoleto e inseguro) y **SSH** (remoto seguro),
- HTTP/HTTPS (interfaz gráfica en algunos equipos) y API (modelos modernos).

Acceso inicial: cable de consola (rollover) al puerto **Console**, un emulador de terminal (PuTTY, Tera Term, o la pestaña Desktop → Terminal de Packet Tracer) con parámetros: **9600 baudios, 8 bits de datos, sin paridad, 1 bit de stop, sin control de flujo**.

Modos de comando (jerarquía):

| Modo | Prompt típico | Entrar con | Función |
|------|---------------|-----------|---------|
| EXEC de usuario (user EXEC) | `Switch>` | — (tras login) | Comandos básicos de monitorización |
| EXEC privilegiado (privileged EXEC) | `Switch#` | `enable` | Todos los show + recarga, copia |
| Configuración global (global config) | `Switch(config)#` | `configure terminal` | Cambios de todo el equipo |
| Configuración de interfaz | `Switch(config-if)#` | `interface g0/1` | Direccionamiento/estado de puertos |
| Configuración de línea | `Switch(config-line)#` | `line con 0` / `line vty 0 15` | Acceso por consola/vty, contraseñas |
| Modo de subconfiguración (p. ej. VLAN) | `Switch(config-vlan)#` | `vlan 10` | Parámetros del objeto |

`exit` retrocede un nivel; `end` (o Ctrl+Z) salta directo a EXEC privilegiado; `disable` sale de privilegiado a usuario.

### 3.2 Cómo Moverse Dentro de IOS (Clase 3.2)
Ayuda y edición sensibles al contexto:
- `?` muestra comandos disponibles en el punto actual; `di?` muestra los que empiezan por "di".
- **Tab** completa el comando; las partes no ambiguas pueden abreviarse (`conf t` = `configure terminal`, `int g0/1` = `interface gigabitethernet0/1`).
- Mensajes de error: `% Ambiguous command` (faltan letras), `% Incomplete command` (faltan argumentos), `% Invalid input detected` (sintaxis errónea; el caret `^` marca dónde).

Atajos de teclado esenciales:

| Atajo | Acción |
|-------|--------|
| Ctrl+A / Ctrl+E | Ir al inicio / final de la línea |
| Ctrl+B / Ctrl+F (o ←/→) | Retroceder / avanzar un carácter |
| Ctrl+W | Borrar palabra anterior |
| Ctrl+U | Borrar toda la línea |
| Ctrl+C | Cancela la operación/diálogo actual |
| Ctrl+Z | Volver a EXEC privilegiado |
| Ctrl+Shift+6 | Interrumpe ping/traceroute (break) |
| Esc B / Esc F | Atrás/adelante una palabra |
| Ctrl+N / Ctrl+P (o ↓/↑) | Comando siguiente / anterior del historial |

Historial: `show history`, `terminal history size <n>`. Paginado: `--More--` se avanza con Espacio (página), Enter (línea), `q` (salir).

### 3.3 Comandos de IOS (Clase 3.3)
Comandos de verificación más usados:

```
Switch> show version           ! IOS, uptime, modelo, memoria
Switch# show running-config    ! configuración activa (RAM; EXEC privilegiado)
Switch# show startup-config    ! configuración guardada (NVRAM)
Switch# show ip interface brief   ! IPs y estado de interfaces (esencial)
Switch# show interfaces        ! contadores, errores, duplex/velocidad
Switch# show mac address-table ! tabla MAC del switch
Switch# show cdp neighbors     ! vecinos Cisco
Switch# show history           ! historial de comandos
```

Búsqueda y filtrado de salidas largas: `show running-config | include hostname`, `| section interface`, `| begin line`, `| exclude ...`.

### 3.4 Configuraciones Iniciales (Clase 3.4)
Secuencia de configuración básica recomendada (la pregunta clásica del examen y del lab):

```
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW1
SW1(config)# enable secret cisco123       ! contraseña de EXEC privilegiado (cifrada)
SW1(config)# no ip domain-lookup          ! evita que interprete typos como DNS
SW1(config)# service password-encryption  ! cifra contraseñas en texto plano del config
SW1(config)# banner motd #Solo personal autorizado#
SW1(config)# line console 0
SW1(config-line)# password cisco123
SW1(config-line)# login
SW1(config-line)# exec-timeout 5 0        ! 5 min de inactividad
SW1(config-line)# exit
SW1(config)# line vty 0 15
SW1(config-line)# password cisco123
SW1(config-line)# login                   ! mejor: transport input ssh + local con AAA (fuera de alcance parcial)
SW1(config-line)# exit
SW1(config)# interface vlan 1
SW1(config-if)# ip address 192.168.1.2 255.255.255.0   ! IP de gestión del switch
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# ip default-gateway 192.168.1.1            ! solo en switches L2
```

Notas:
- `enable secret` se almacena con hash fuerte (tipo 5 = MD5; tipo 8 = PBKDF2-SHA256; tipo 9 = scrypt en IOS modernos); es preferible a `enable password` (obsoleto, sin cifrado o cifrado débil reversible).
- `service password-encryption` usa Vigenère débil (tipo 7), ofusca pero **no** protege de verdad; aun así se exige en los labs.
- `no ip domain-lookup`: si escribes un typo (p. ej. `confog t`), IOS no intenta resolverlo como nombre DNS (proceso molesto y lento).
- `banner motd`: mensaje legal/aviso antes del login. Delimitador típico `#`.
- Contraseña para vty: sin ella, Telnet/SSH rechazan conexión ("Password required, but none set").
- Gestión remota del switch L2: IP en **VLAN de gestión** (normalmente VLAN 1 o una dedicada) + `ip default-gateway`.

### 3.5 Guardar los Cambios (Clase 3.5)
Memoria volátil vs no volátil:

| Fichero | Se guarda en | Contenido | Si reinicia el equipo |
|---------|--------------|-----------|----------------------|
| **running-config** | RAM | Config activa | Se pierde |
| **startup-config** | NVRAM | Config de arranque | Se carga |

Comandos:

```
SW1# copy running-config startup-config    ! o: write memory / wr
SW1# copy startup-config running-config    ! merge de NVRAM a RAM
SW1# erase startup-config                  ! borra NVRAM
SW1# reload                                ! reinicia (pide confirmar; entra system config dialog si no hay startup)
```

En routers, al arrancar sin startup-config aparece el *System Configuration Dialog*; se responde `no` y se configura con CLI. Verificación: `show running-config`, `show startup-config`, `show flash:`.

### 3.6 Preguntas del Módulo
Quiz de la sección; practica las preguntas de autoevaluación.

## Ejemplos

### Ejemplo 1 — Primera sesión en un switch nuevo (Packet Tracer)
Arrastra un switch 2960 → Desktop → Terminal → 9600/8/N/1 → Enter. Debes ver `Switch>`. Ejecuta:

```
Switch> enable
Switch# show version
Switch# configure terminal
Switch(config)# hostname SW1
SW1(config)# enable secret class
SW1(config)# service password-encryption
SW1(config)# no ip domain-lookup
SW1(config)# banner motd #Acceso restringido#
SW1(config)# line console 0
SW1(config-line)# password cisco
SW1(config-line)# login
SW1(config-line)# exit
SW1(config)# interface vlan 1
SW1(config-if)# ip address 192.168.1.10 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# end
SW1# copy running-config startup-config
```

Resultado esperado: prompt `SW1#`, IP de gestión asignada, config salvada (`[OK]`).

### Ejemplo 2 — Recuperar la contraseña de consola olvidada (concepto)
En un switch físico: cortar alimentación, mantener pulsado el botón **Mode** durante el arranque, entrar en modo rommon, inicializar flash e ignorar el startup-config (`flash_init`, `load_helper`, `dir flash:`, renombrar `config.text`), arrancar, renombrar de vuelta y sobrescribir contraseñas. En Packet Tracer no es reproducible; el examen solo pide el concepto: **el acceso físico implica control total** → seguridad física + banner legal.

### Ejemplo 3 — Diferencias running vs startup
Configura un hostname nuevo pero NO guardes. Comprueba `show running-config | include hostname` (aparece el nuevo) y `show startup-config | include hostname` (aparece el viejo). Haz `reload`: tras reiniciar, el hostname viejo vuelve. Conclusión: **siempre `copy run start` tras un cambio**.

### Ejemplo 4 — Salida típica de `show ip interface brief`

```
SW1# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
Vlan1                  192.168.1.10    YES manual up                    up
FastEthernet0/1        unassigned      YES unset  up                    up
FastEthernet0/2        unassigned      YES unset  down                  down
```
`Status` = capa física; `Protocol` = capa de enlace. Ambas en `up` = operativo.

## Ejercicios

1. **(Recordar)** ¿Qué prompt identifica el modo EXEC privilegiado y cómo se entra?
2. **(Recordar)** ¿Qué diferencia hay entre running-config y startup-config? ¿Dónde reside cada una?
3. **(Aplicar)** Escribe la secuencia exacta de comandos para: asignar hostname `R1`, contraseña enable `cisco`, cifrar contraseñas, desactivar resolución DNS y crear banner `#Solo autorizados#`.
4. **(Aplicar)** Configura acceso por consola con contraseña `lab` y timeout de 10 minutos. ¿Qué dos comandos son obligatorios dentro de `line console 0`? (El resto es recomendable, no obligatorio.)
5. **(Comprender)** ¿Por qué se recomienda `no ip domain-lookup` en equipos de prácticas? ¿Qué pasa si no lo pones y escribes `confog t`?
6. **(Aplicar)** Indica el comando para guardar la configuración y dos formas equivalentes de verificar que los cambios persisten tras un reload.
7. **(Analizar)** Un alumno hace `reload` sin `copy run start` y, al volver, ha perdido toda la config. Explica por qué y qué debería haber hecho. ¿Qué comando habría mostrado la diferencia antes de recargar?
8. **(Analizar)** ¿Por qué `enable secret` es superior a `enable password`? ¿Qué hace `service password-encryption` y qué NO garantiza?

<details>
<summary>Soluciones</summary>

1. `Switch#` / `Router#`; con el comando `enable` desde user EXEC.
2. running-config = RAM, activa, volátil. startup-config = NVRAM, de arranque, persistente.
3. `enable` → `configure terminal` → `hostname R1` → `enable secret cisco` → `service password-encryption` → `no ip domain-lookup` → `banner motd #Solo autorizados#` → `end`.
4. Obligatorios en `line console 0`: `password lab` y `login` (sin `login` la contraseña no se pide; sin `password`, `login` no tiene qué pedir). El resto es recomendable, no obligatorio: `exec-timeout 10 0` para el timeout (10 min de inactividad).
5. Sin él, IOS interpreta lo irreconocible como nombre de host y lanza una consulta DNS que tarda; con `no ip domain-lookup` responde al instante con error de sintaxis.
6. Guardar: `copy running-config startup-config` (o `write memory`). Verificar, forma 1: `show startup-config` (confirma que la copia en NVRAM ya incluye los cambios, antes de recargar). Verificar, forma 2: hacer `reload` y comprobar `show running-config` (los cambios persisten tras el arranque).
7. La RAM es volátil: al recargar se borra y arranca desde NVRAM. Debió hacer `copy run start`. `show running-config` vs `show startup-config` (o `show startup-config | include ...`) habría mostrado la diferencia.
8. `enable secret` se almacena con hash (tipo 5/9), `enable password` en claro/tipo 7 reversible. `service password-encryption` aplica ofuscación tipo 7 a contraseñas en plano; no es cifrado fuerte: un volcado del config permite revertirlas.
</details>

## Tips
- **Abreviar comandos:** solo mientras la abreviatura sea inequívoca (`conf t`, `int g0/1`, `sh ip int br`, `wr`).
- **Tab** completa; **?** es tu mejor amigo en el examen práctico.
- El examen pregunta mucho: prompts (`Router>` vs `Router#`), `copy run start` vs `write memory`, y diferencias RAM/NVRAM.
- Orden típico de preguntas: "¿Qué comando guarda la configuración?" → `copy running-config startup-config`.
- Si el prompt muestra `--More--`, usa Espacio/Enter o `q`. En scripts, `terminal length 0` desactiva el paginado (cuidado: no en producción real sin pensar).
- `show ip interface brief` es el comando de diagnóstico rápido nº 1: memorízalo.

## Datos curiosos
- Cisco IOS nació de Cisco IOS para los primeros routers integrados IP (finales de los 80); el nombre "IOS" sobrevivió incluso a las eras de CATOS (switches CatOS) y NX-OS (datacenter).
- El tipo de cifrado `service password-encryption` (tipo 7) fue público desde los 90; por eso Cisco recomienda `enable secret` con MD5 (tipo 5) o, mejor, SHA-256 (tipo 9) en equipos modernos.
- El modo `setup` (System Configuration Dialog) fue la consola "guiada" de los primeros routers; hoy solo aparece como cortesía cuando falta startup-config.

## Para tu libreta de apuntes
- Acceso consola: 9600 8N1. Modos: `>` user → `enable` → `#` privileged → `configure terminal` → `(config)#` → `interface/line` → `(config-if/-line)#`.
- `?`, Tab, abreviaturas, Ctrl+Z, Ctrl+C, Ctrl+Shift+6.
- Básicos: hostname, enable secret, service password-encryption, no ip domain-lookup, banner motd, line console 0 + password + login, exec-timeout.
- Guardar: `copy run start` = `wr`. RAM ↔ NVRAM. `reload`.
- Verificar: `show ip int br`, `show version`, `show run`, `show start`, `show mac address-table`.

## Preguntas de autoevaluación

1. ¿Qué prompt aparece en modo EXEC de usuario y cuál en privilegiado? **Respuesta:** `Router>` y `Router#`.
2. ¿Qué comando guarda la configuración actual para el próximo arranque? **Respuesta:** `copy running-config startup-config` (o `write memory`).
3. ¿Qué comando cifra las contraseñas almacenadas en texto plano del running-config? **Respuesta:** `service password-encryption`.
4. Un compañero ejecuta `reload` sin guardar; tras arrancar, ¿qué verá? **Respuesta:** La startup-config (NVRAM), no los cambios hechos en RAM.
5. ¿Qué dos subcomandos son imprescindibles en `line console 0` para que pida contraseña al entrar? **Respuesta:** `password <clave>` y `login`.
