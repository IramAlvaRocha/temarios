# 06 — Capa 1: Capa Física

> Duración del curso: 25m · 5 clases
> Fuente: temario.md del curso

## Objetivos de aprendizaje
- Describir las características de la capa física: codificación de señales, tipos de medio y ancho de banda.
- Identificar cables de cobre (UTP/STP, categorías) y fibra óptica, y sus conectores (RJ-45, SC/LC).
- Explicar el armado de un conector RJ-45 (568A/568B, directo vs cruzado).
- Describir la transmisión inalámbrica (Wi‑Fi) y sus bandas/estándares.

## Teoría

### 6.1 Características (Clase 6.1)
La **capa física** (OSI capa 1) se ocupa de convertir bits en señales físicas y transportarlos. Define:
- **Codificación de señal:** cómo se representan los bits (pulsos eléctricos, luz, radio). Ejemplos: codificación **Manchester** (transición en mitad del bit, 10BASE-T), **4B/5B** (Fast Ethernet sobre fibra), **8B/10B** (Gigabit Ethernet sobre fibra), **PAM-5** (1000BASE-T).
- **Ancho de banda (bandwidth):** capacidad del medio en bps. Ethernet evolucionó: 10 Mbps (Ethernet, 1983) → 100 Mbps (Fast) → 1000 Mbps (Gigabit) → 10G/40G/100G+.
- **Throughput** (rendimiento real) ≤ ancho de banda; afectado por ruido, colisiones, distancia, congestión.
- **Tipos de señalización:** banda base (baseband, una señal a la vez, p. ej. Ethernet) vs banda ancha (broadband, varias señales/frecuencias, p. ej. cable HFC/TV).
- **Medios:** cobre (eléctrico), fibra (luz), inalámbrico (radio).

### 6.2 Cableado (Clase 6.2)
**Cobre — Par trenzado no blindado (UTP, Unshielded Twisted Pair):**
El estándar es **T568** (TIA/EIA-568). Categorías:

| Categoría | Velocidad | Distancia máx. | Uso |
|-----------|-----------|----------------|-----|
| Cat3 | 10 Mbps | 100 m | Voz, Ethernet antiguo |
| Cat5 | 100 Mbps | 100 m | Fast Ethernet (obsoleta) |
| Cat5e | 1 Gbps | 100 m | Gigabit Ethernet (mínimo actual) |
| Cat6 | 1–10 Gbps | 55 m (10G) / 100 m (1G) | Gigabit/10G |
| Cat6a | 10 Gbps | 100 m | 10G pleno |
| Cat7/7a | 10–40 Gbps | 100 m | Centros de datos (apantallado, fuera de CCNA apenas) |
| Cat8 | 25/40 Gbps | 30 m | Data center |

- **STP/FTP (blindado):** mejor contra EMI (motores, fluorescentes), pero más caro y rígido; requiere puesta a tierra. UTP domina en oficinas por coste/flexibilidad.
- **Conector RJ-45** (8P8C). Pines 1-8.
- **Directo (straight-through):** mismo estándar en ambos extremos (568B-568B). Conecta dispositivos de **distinto** tipo: PC↔switch, switch↔router.
- **Cruzado (crossover):** 568A en un extremo y 568B en el otro (pares TX/RX intercambiados). Conecta dispositivos del **mismo** tipo: PC↔PC, switch↔switch, router↔router. (Equipos modernos con **auto-MDIX** detectan y cruzan internamente, haciendo el cruzado opcional.)
- **Consola (rollover):** pinout invertido 1↔8, 2↔7... cable azul de consola para el puerto Console.

**Fibra óptica:**
- **Monomodo (single-mode, SMF):** núcleo ~9 µm, láser, largas distancias (hasta decenas de km), mayor ancho de banda. Amarilla.
- **Multimodo (multimode, MMF):** núcleo 50/62.5 µm, LED/VCSEL, distancias cortas (300–550 m a 1/10G). Naranja/agua (OM3/OM4).
- Conectores: **SC** (push-pull cuadrado), **LC** (pequeño, tipo RJ, muy usado en switches), **ST** (bayoneta, antiguo). Tipos de pulido: PC/UPC/APC (verde = APC 8°).
- Inmune a EMI, mayor seguridad (difícil de pinchar), mayor distancia y velocidad. Desventaja: coste, fragilidad, instalación especializada.

### 6.3 Práctica de Armado de RJ45 (Clase 6.3)
Proceso de ponchado:
1. Pelar la cubierta exterior ~2–3 cm sin dañar los pares.
2. Destrenzar y ordenar los cables según T568B (el estándar más usado) o T568A.
   - **T568B:** 1-Blanco/Naranja, 2-Naranja, 3-Blanco/Verde, 4-Azul, 5-Blanco/Azul, 6-Verde, 7-Blanco/Marrón, 8-Marrón.
   - **T568A:** 1-Blanco/Verde, 2-Verde, 3-Blanco/Naranja, 4-Azul, 5-Blanco/Azul, 6-Naranja, 7-Blanco/Marrón, 8-Marrón.
3. Cortar los cables rectos a la misma longitud (~12–13 mm).
4. Insertar en el RJ-45 con la pestaña hacia abajo y los pines mirando al frente (orden 1→8).
5. Ponchar con la crimpadora (verifica que los contactos atraviesan la aislación).
6. Probar con un **cable tester** (continuidad 1→1 o el mapeado cruzado).
Errores típicos: pares destrenzados más de 13 mm (crosstalk), orden incorrecto, contactos sin crimpar, 568A/B mixto sin pretender cruzado.

### 6.4 Transmisión Wireless (Clase 6.4)
Wi‑Fi (IEEE 802.11):
- **Bandas:** 2.4 GHz (mayor alcance, más interferencias: microondas, Bluetooth; canales solapados, usar 1/6/11) y 5 GHz (menos alcance e interferencias, más canales, mayor velocidad) y 6 GHz (Wi‑6E/Wi‑7).
- Estándares: 802.11a (5 GHz, 54 Mbps), 802.11b (2.4, 11), 802.11g (2.4, 54), 802.11n (Wi‑Fi 4, MIMO, 600), 802.11ac (Wi‑Fi 5, 5 GHz, Gbps), 802.11ax (Wi‑Fi 6/6E).
- **CSMA/CA** (Carrier Sense Multiple Access with Collision Avoidance): evita colisiones escuchando el canal; no las detecta como Ethernet (CSMA/CD).
- Seguridad: WPA2/WPA3 (AES); WEP está roto (nunca usarlo). Ocultar SSID NO es seguridad real.
- Otros inalámbricos: Bluetooth (PAN, ~10 m), Zigbee, satélite, celular (3G/4G/5G).

### 6.5 Preguntas del Módulo
Quiz de autoevaluación.

## Ejemplos

### Ejemplo 1 — ¿Qué cable usar?
- PC ↔ switch: **directo**.
- Switch ↔ switch (sin auto-MDIX): **cruzado**.
- PC ↔ PC (red rápida sin hub): **cruzado**.
- Switch ↔ router: **directo**.
- Puerto Console ↔ PC (con adaptador): **rollover**.
- Centro de datos a 40 km: **fibra monomodo**.
- Planta industrial con motores: **STP apantallado** o fibra.

### Ejemplo 2 — Ponchado paso a paso (568B)
Pela → desenreda → ordena: B/N, N, B/V, Az, B/Az, V, B/M, M → corta recto ~12 mm → inserta (clip abajo) → crimpa → testea continuidad. Si el tester marca el par 3↔6 cruzado y pintaste 568A en un extremo + 568B en otro → tienes un cruzado.

### Ejemplo 3 — Cálculo de alcanzable en Gigabit
1000BASE-T necesita los **4 pares** y alcance máximo **100 m** (90 m fijos en pared + 10 m latiguillos según TIA). Si superas 100 m: atenuación excesiva → errores CRC → usar fibra o repetidores (switch intermedio).

### Ejemplo 4 — Wi‑Fi doméstico vs empresarial
Hogar: un AP, WPA2/WPA3-PSK, canal automático o 1/6/11, 2.4+5 GHz. Empresa: múltiples APs controlados por un **WLC** (Wireless LAN Controller), autenticación **802.1X/RADIUS**, VLAN de invitados separada, QoS para voz.

## Ejercicios

1. **(Recordar)** ¿Qué estandarizan T568A y T568B y en qué se diferencian?
2. **(Recordar)** Enumera 3 ventajas de la fibra sobre el cobre.
3. **(Aplicar)** Indica el tipo de cable (directo/cruzado/rollover/fibra): (a) PC→switch, (b) switch→switch sin auto-MDIX, (c) PC→PC, (d) router→switch, (e) consola.
4. **(Aplicar)** Escribe el orden de colores T568B del pin 1 al 8.
5. **(Comprender)** ¿Por qué el destrenzado del par debe ser mínimo al ponchar RJ-45? ¿Qué fenómeno combate?
6. **(Analizar)** Un switch negocia 100 Mbps full dúplex en un enlace que debería ser 1 Gbps. Cita dos causas probables de capa física.
7. **(Analizar)** ¿Por qué 2.4 GHz sufre más interferencias que 5 GHz y qué canales Wi‑Fi de 2.4 GHz no se solapan?
8. **(Aplicar)** En Packet Tracer, crea la topología PC–SW–R con cables directos y un enlace cruzado entre switches; verifica con el modo Simulation que el LED del enlace queda en verde.

<details>
<summary>Soluciones</summary>

1. Estandarizan la asignación pin↔color en RJ-45. Se diferencian en el intercambio de los pares naranja y verde (T568A: verde-naranja; T568B: naranja-verde). Ambos extremos con el mismo estándar = directo.
2. Mayor distancia, mayor ancho de banda/velocidad, inmunidad electromagnética (EMI) e intrusión más difícil de detectar/pinchar.
3. (a) directo; (b) cruzado; (c) cruzado; (d) directo; (e) rollover.
4. Blanco/Naranja, Naranja, Blanco/Verde, Azul, Blanco/Azul, Verde, Blanco/Marrón, Marrón.
5. El trenzado cancela el **crosstalk** (interferencia entre pares y EMI); destrenzar más de ~13 mm lo degrada y provoca errores/retransmisiones.
6. (a) Cableado inadecuado (solo 2 pares usados, Cat5 en vez de Cat5e/6); (b) puerto mal negociado o en 10/100 half (duplex mismatch) o cable largo/dañado. (También un extremo en 100 Mbps fijo.)
7. Porque 2.4 GHz es una banda congestionada (microondas, Bluetooth, otros Wi‑Fi) y solo dispone de 3 canales no solapados: **1, 6 y 11**. 5 GHz tiene muchos más canales no solapados.
8. Observar LEDs verdes en los extremos de los enlaces directos y el cruzado; enlaces con cable incorrecto quedan apagados/ámbar.
</details>

## Tips
- **Directo = distinto tipo de equipo; cruzado = mismo tipo** (regla clásica pre-auto-MDIX).
- T568B es el más usado en instalaciones comerciales (EE. UU.); T568A es obligatorio en instalaciones federales y es el requerido por estándares de residencial en algunos países. Eso sí: **no mezclar** salvo para cruzado.
- El examen pregunta: "¿Qué cable conecta switch con switch?" → cruzado (o directo si hay auto-MDIX).
- 100 m es el límite Ethernet en cobre; más allá: fibra.
- Fibra: monomodo = distancia/láser/amarillo; multimodo = corta distancia/LED/naranja-agua. Conector LC = el más común en switches actuales.
- Ethernet de hoy va casi siempre **full-duplex** (sin colisiones); half-duplex y CSMA/CD solo en hubs históricos.

## Datos curiosos
- Ethernet se desarrolló en 1973 en Xerox PARC (Robert Metcalfe y David Boggs) a ~2.94 Mbps sobre coaxial compartido ("el éter" — de ahí el nombre). En 1979 Metcalfe fundó 3Com para comercializarlo y en 1983 se publicó el estándar **IEEE 802.3**; 10BASE-T y la topología estrella llegaron en 1990.
- El RJ-45 original (Registered Jack) es en realidad un conector telefónico; el de Ethernet se llama formalmente **8P8C**, aunque todo el mundo dice "RJ-45".
- El IEEE ha ido escalando el estándar: 10/100/1000 Mbps, luego 10G/25G/40G/100G y, en centros de datos, 200G/400G/800G (grupos 802.3ba, 802.3bs y 802.3df).

## Para tu libreta de apuntes
- Capa 1 = bits → señales. Codificación: Manchester, 4B/5B, PAM-5.
- UTP: Cat5e (1G) · Cat6 (1–10G) · Cat6a (10G) · máx 100 m · RJ-45.
- Directo (B-B): distinto tipo · Cruzado (A-B): mismo tipo · Rollover: consola.
- Fibra: SMF 9 µm larga distancia · MMF 50/62.5 corta · conectores SC/LC · inmune EMI.
- Wi‑Fi 2.4/5/6 GHz · CSMA/CA · canales 1/6/11 · WPA2/WPA3.

## Preguntas de autoevaluación

1. ¿Qué cable usarías para conectar dos switches sin auto-MDIX? **Respuesta:** Cable cruzado (crossover).
2. ¿Cuál es la distancia máxima de un enlace UTP Ethernet y a qué velocidad opera Cat6a? **Respuesta:** 100 m; Cat6a = 10 Gbps.
3. Ordena de pin 1 a 8 un RJ-45 T568B. **Respuesta:** BN, N, BV, Az, BAz, V, BM, M.
4. Ventaja principal de la fibra frente al cobre en un entorno industrial: **Respuesta:** Inmunidad a interferencias electromagnéticas (EMI) y mayor distancia.
5. ¿Qué canal Wi‑Fi de 2.4 GHz elegirías para evitar solape con otros AP? **Respuesta:** Uno de los no solapados: 1, 6 u 11.
