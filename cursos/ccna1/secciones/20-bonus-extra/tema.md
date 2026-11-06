# 20 — Bonus Extra

> Duración del curso: 3m · 1 clase
> Fuente: temario.md del curso

## Objetivos de aprendizaje
- Aprovechar recursos extra del curso (clase extra) para reforzar los fundamentos vistos.
- Reconocer rutas complementarias de formación (p. ej., ciberseguridad) y dónde encajan dentro del camino CCNA.
- Cerrar el curso con un plan de práctica continua y recursos confiables.

## Teoría

Sección bonus (Clase 20.1 — Clase Extra). Contenido breve y orientado a ampliar el estudio; no introduce un bloque teórico nuevo del temario oficial.

### 20.1 Clase Extra
- **Valor de la clase extra del curso:** suele ofrecer perspectivas adicionales del instructor: aclaraciones de dudas frecuentes de alumnos, ejemplos de topologías del mundo real, o recursos para profundizar (documentación, herramientas, recomendaciones). Tómala como refuerzo, no como temario obligatorio.
- **Ruta complementaria de ciberseguridad** (mencionada también en 1.2): para quien descubra interés en seguridad ofensiva/defensiva, las certificaciones de entrada lógicas serían (tras CCNA o en paralelo): CompTIA Security+, Cisco CyberOps Associate, eJPT (ofensiva de entrada). No las necesitas para el 200-301.
- **Refuerzos prácticos recomendados tras esta parte:**
  - Subnetting diario (10 min: escribe subredes de /24, /16, /19, /27 sin calculadora).
  - Un lab semanal en Packet Tracer: "cada viernes resuelves una topología nueva en <45 min".
  - Leer RFCs ligeros: 791 (IP), 768 (UDP), 793 (TCP), 4443 (ICMPv6), 4861 (NDP), 3021 (/31) — en 10 minutos cada uno ganas más precisión que con horas de vídeo.
  - Capturas con **Wireshark** (gratuito) de tráfico real (tu propia red): filtra `arp`, `icmp`, `dns`, `tcp.port==80` y observa los encabezados vistos en esta parte.
- **Recursos oficiales y confiables:**
  - Cisco Exam Topics 200-301 (cisco.com): la referencia de qué entra.
  - Cisco Packet Tracer / NetAcad (netacad.com): labs y cursos.
  - RFC Editor (rfc-editor.org): los estándares en sí.
  - Cisco Learning Network (learningnetwork.cisco.com): comunidad oficial.
  - Documentación de IOS de Cisco (Cisco IOS Configuration Guides, gratis en cisco.com).

## Ejemplos

### Ejemplo 1 — Sesión "bonus" de 30 minutos
1. Abre Packet Tracer: monta la red del curso completa de memoria (R1 + SW1 + 2 PCs).
2. Configura IPs, gateway y gestión del switch; guarda.
3. Abre Wireshark (o Simulation en PT) y captura un `ping` y un `nslookup`: identifica campos de capas 2, 3, 4 en la captura.
4. Anota en tu libreta 3 cosas que no recordabas.

### Ejemplo 2 — Mapa de decisiones: ¿sigo con redes o pivoteo?
- Te encantan las topologías y el troubleshooting → **CCNA → rutas Cisco/SDN/Data Center/Security**.
- Te atrae romper sistemas legalmente → tras fundamentos: **eJPT / Junior pentesting (siempre en laboratorio y con autorización)**.
- Te gusta la gestión de usuarios y políticas → **SysAdmin / cloud (Azure/AWS) con base de redes**.
La base de redes es común a todas; el CCNA no cierra puertas, las abre.

### Ejemplo 3 — Checklist de cierre del curso
- [ ] Todos los `tema.md` repasados; quizzes de cada módulo sin errores.
- [ ] Lab "red completa" reproducible desde blanco en Packet Tracer.
- [ ] Subnetting resuelto sin calculadora (10 ejemplos variados).
- [ ] Ping/tracert/ipconfig y comandos IOS memorizados de uso diario.
- [ ] Parte 2 del curso programada en el calendario.

## Ejercicios

1. **(Recordar)** Nombra 3 recursos oficiales para preparar el CCNA 200-301.
2. **(Aplicar)** Diseña tu rutina semanal de 3 sesiones: una de subnetting, una de labs, una de lectura técnica. Especifica horas y entregables.
3. **(Comprender)** ¿Por qué Wireshark es el complemento perfecto a Packet Tracer? ¿Qué muestra cada uno?
4. **(Analizar)** Comparativa coste-beneficio: quedarte atascado en "ver vídeos" vs hacer labs imperfectos. ¿Por qué ganar haciendo > ver perfecto?
5. **(Aplicar)** Escribe tu "plan bonus personal" en 5 líneas: qué harás tras el curso y para cuándo.

<details>
<summary>Soluciones</summary>

1. Cisco Exam Topics 200-301 · NetAcad/Packet Tracer + Study Plan · RFC Editor · Cisco Learning Network · Guías oficiales de Cisco IOS. (Cualquiera de ellas valen.)
2. Ejemplo: martes 25 min subnetting (hoja de ejercicios auto-corregida); jueves 45–60 min lab PT (1 topología nueva + verificación con show/ping); domingo 30 min lectura (1 tema de Exam Topics + 1 RFC o captura Wireshark documentada).
3. Packet Tracer **simula** la red (equipos virtuales, configuración IOS, topologías); Wireshark **captura** paquetes reales de una red existente para analizar encabezados y protocolos en vivo. Juntos cubren la construcción y la comprensión profunda.
4. Ver = reconocimiento pasivo; hacer = recuerdo activo + depuración. Los errores que cometes en un lab (máscara mal, `no shutdown` olvidado, VLAN errónea) crean el rastro de memoria que necesitas para diagnosticar en producción y en el examen. Los vídeos solo muestran el camino feliz.
5. Respuesta libre; recomendable incluir: repaso Parte 1, avance Parte 2, fecha meta de examen (p. ej., 8-12 semanas), método (labs + simulacros), métrica (nota en simulacros ≥85 %).
</details>

## Tips
- Esta clase extra es el sitio donde el instructor mete matices: agrégala a notas y vuelve a ella antes del examen.
- Construye tu "cuaderno de errores": cada fallo en quizzes/labs con su corrección; léelo antes de cada simulacro.
- No colecciones cursos: termina uno, certifícalo, pasa al siguiente.
- Wireshark con filtros: `icmp`, `arp`, `dns`, `tcp.handshake` → memoria muscular sobre bytes reales.
- Comparte tus notas: explicar en voz alta (rubber duck) revela lagunas.

## Datos curiosos
- Wireshark (antes Ethereal, 1998) es el analizador de paquetes más usado del mundo; nació como proyecto de Gerald Combs y hoy es mantenido por la comunidad bajo GPL; enseñarlo en un currículo aumenta la comprensión de protocolos más que cualquier diagrama.
- El primer "curso en línea masivo" de redes fue una pequeña revolución: Cisco NetAcad (1997) entrenó vía web a decenas de miles de estudiantes cuando Internet de consumo aún gateaba — el programa sigue siendo referencia mundial de la formación en redes.
- Muchos instructores CCNA repiten el mismo consejo: "el día que configuras tu primera red real (aunque sea en casa), el curso pasa de ser vídeos a ser tuyo".

## Para tu libreta de apuntes
- Bonus ≠ nuevo temario: refuerzo + recursos (Exam Topics, NetAcad, RFCs, Packet Tracer, Wireshark).
- Rutina: subnetting diario · lab semanal · lectura CCNA.
- Cierre: cuaderno de errores · checklist · Parte 2.
- Examen: práctica > vídeos.

## Preguntas de autoevaluación

1. Nombra dos recursos gratuitos oficiales para seguir estudiando. **Respuesta:** NetAcad (Packet Tracer/cursos) y Exam Topics oficiales de Cisco (cisco.com); además RFC Editor y Cisco Learning Network.
2. ¿Qué herramienta usarías para observar los bits de una trama ARP real en tu LAN? **Respuesta:** Wireshark con filtro `arp`.
3. ¿Qué es un "cuaderno de errores" y para qué sirve? **Respuesta:** Registro personal de fallos en quizzes/labs con su corrección; refuerza la memoria (error + corrección se recuerda mejor) y acelera el último repaso pre-examen.
4. Tras este curso, ¿el siguiente paso natural es…? **Respuesta:** La Parte 2 del curso (VLANs, STP, OSPF, ACLs, NAT, automatización) y posteriormente el examen 200-301; opcionalmente en paralelo reforzar subnetting y labs.
5. Verdadero/Falso: "La clase extra es obligatoria para aprobar el CCNA". **Respuesta:** Falso — es complementaria; lo evaluable viene del blueprint oficial y tu práctica.
