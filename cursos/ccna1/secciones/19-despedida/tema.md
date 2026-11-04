# 19 — Despedida

> Duración del curso: 4m · 1 clase
> Fuente: temario.md del curso

## Objetivos de aprendizaje
- Hacer balance del temario completado y situarlo dentro del camino completo a la certificación CCNA 200-301.
- Planificar el estudio de la Parte 2 y el examen oficial.
- Conocer los recursos oficiales de Cisco (NetAcad, documentación) y la siguiente etapa profesional.

## Teoría

Esta sección es de cierre: no añade teoría técnica, pero consolida el plan de estudio y carrera (Clase 19.1).

### 19.1 Despedida del curso
**Balance del curso (Parte 1 — Introducción a las Redes):**
- Cubriste los fundamentos: qué es una red, modelos OSI/TCP-IP, capas 1–7, switching Ethernet, routing, IPv4/IPv6, ARP/NDP, ICMP, transporte (TCP/UDP), aplicación (HTTP/DNS/DHCP/correo), seguridad básica y el método de troubleshooting.
- Todo ello es el 40–50 % del alcance conceptual del CCNA 200-301; el resto se aborda en la Parte 2: VLANs, STP, EtherChannel, HSRP, seguridad aplicada (port-security, AAA), NAT, ACLs, OSPF, y automatización/programmability (conceptos de APIs, JSON — el nuevo en CCNA 200-301).

**Qué hacer ahora:**
1. Repetir los labs de Packet Tracer una semana después (curva del olvido: repaso espaciado).
2. Completar las preguntas de autoevaluación de cada `tema.md` hasta fallar 0.
3. Esperar a la Parte 2 del curso para el resto del temario.
4. Registrarse en NetAcad y revisar el **Study Plan de Cisco** para el examen 200-301.

**Estructura oficial del examen 200-301 (referencia):**
- Un único examen de 120 minutos; nota de corte variable (~825/1000 según histórico; Cisco no publica la exacta).
- Dominios (v1.1 oficial): 1.0 Fundamentos de red 20% · 2.0 Acceso a la red 20% · 3.0 Conectividad IP 25% · 4.0 Servicios IP 10% · 5.0 Fundamentos de seguridad 15% · 6.0 Automatización y programabilidad 10%. (Los porcentajes exactos pueden variar ligeramente con las actualizaciones del blueprint; verifica siempre el "Exam Topics" oficial en cisco.com.)

**Consejos de carrera:**
- Título + laboratorio: la certificación abre puertas; las prácticas en home lab son lo que te hace empleable. Empieza con Packet Tracer → GNS3/EVE-NG → eBay racks por horas.
- Perfiles naturales: NOC/helpdesk → network admin → network engineer; especializaciones (seguridad, SDN/ACI, voz, cloud networking) llegan después.
- Documenta tu aprendizaje (GitHub con labs y notas, blog); comunica tu progreso en LinkedIn: los recruiters miran actividad.
- Comunidad: Reddit r/ccna, foro NetAcad, Cisco Learning Network.

## Ejemplos

### Ejemplo 1 — Plan de 8 semanas hacia el CCNA (plantilla)
Semanas 1–2: fundamentos (este curso, Parte 1) + subnetting diario (10 min/día). Semanas 3–4: Parte 2 (VLANs, STP, OSPF, ACLs). Semana 5: laboratorios integrales (diseña una red de campus completa en PT). Semana 6: bancos de preguntas oficiales (Boson, ExSim; cuidado con dumps ilegales: violan NDA de Cisco). Semanas 7–8: simulacros de examen (tiempo real), repaso de los puntos débiles, logística Pearson VUE y descanso el día antes.

### Ejemplo 2 — Checklist del día del examen
ID oficial con foto, Pearson VUE confirmación, llegar 30 min antes, nada de apuntes/móvil (centro de examen), desayuno ligero; durante el examen, si una pregunta no la sabes, responde lo mejor que puedas y sigue — no se puede volver atrás. (Hay labs interactivos en el 200-301: identificar problemas de topología y configurar equipos en un entorno simulado.)

### Ejemplo 3 — Qué subir a tu portfolio tras este curso
- 3-4 labs documentados (topología + objetivos + verificación con capturas: "Red pequeña empresarial desde cero").
- Una hoja "comandos que uso de memoria" (show ip int br, copy run start...).
- Certificado de finalización del curso en LinkedIn con habilidades: CCNA, TCP/IP, IPv4/IPv6, routing & switching.

## Ejercicios

1. **(Recordar)** Enumera los 6 dominios del blueprint oficial 200-301 y su peso aproximado.
2. **(Comprender)** ¿Por qué el repaso espaciado (1 semana después) mejora la retención frente a releer la misma tarde?
3. **(Aplicar)** Diseña tu plan personal de 4 semanas post-curso Parte 1: objetivos, horas, recursos.
4. **(Analizar)** Un compañero usa dumps de examen filtrados: ¿qué riesgos tiene (técnico y profesional)?
5. **(Aplicar)** Redacta el bullet de LinkedIn/CV con lo que sabes hacer tras este curso.

<details>
<summary>Soluciones</summary>

1. 1.0 Fundamentos de red 20% · 2.0 Acceso a la red 20% · 3.0 Conectividad IP 25% · 4.0 Servicios IP 10% · 5.0 Fundamentos de seguridad 15% · 6.0 Automatización y programabilidad 10%. (Blueprint v1.1; verificar siempre en Cisco.)
2. La curva del olvido de Ebbinghaus: la retención cae drásticamente si no se recupera el material; evocar el contenido a intervalos crecientes "rescata" el recuerdo y lo consolida (memoria a largo plazo) — mucho más eficaz que el reconocimiento pasivo.
3. Respuesta libre; deben aparecer: repaso de Parte 1 (fallos en quizzes), práctica diaria de subnetting, avance en Parte 2, simulacro semanal, fecha objetivo de examen.
4. Técnico: apruebas sin saber (fallarás en el trabajo real/el examen incluye labs que delatan). Profesional/legal: violación del NDA de Cisco (expulsión/invalidación del certificado), mala fama, y las dumps están desactualizadas con la v1.1. Alternativa: simulacros legales (Boson, Cisco practice, labs propios).
5. Ejemplo: "Aprendiz de redes certificado-en-curso (CCNA 200-301): fundamentos OSI/TCP-IP, IPv4/IPv6 y subnetting, switching y routing estático, IOS básico, seguridad de red y troubleshooting — labs en Packet Tracer."
</details>

## Tips
- Repaso espaciado + "active recall" (te preguntas a ti mismo) > re-lectura.
- El examen 200-301 dura 120 min y puede incluir labs interactivos y drag-and-drop; no permite ir atrás: responde y sigue.
- Cisco renueva la certificación a los 3 años: **Continuing Education** (exámenes/cursos) o re-examinarse.
- No pagues "dumps" ni centros que prometen aprobados; paga Pearson VUE y labs.
- El inglés técnico básico basta para leer los exam topics oficiales; son tu Biblia: imprímelos y tacha los que dominas.

## Datos curiosos
- La primera certificación Cisco (1993) fue la **CCIE**; el currículum empezaba con routing/switching real. CCENT/CCNA aparecieron después para el nivel de entrada.
- Cisco estima en sus estudios (Annual Cybersecurity / Jobs Report) que para ~2025-2030 quedarán sin cubrir millones de puestos de ciberseguridad/redes en el mundo: la demanda estructural sigue alta.
- El 200-301 se estrenó el 24 de febrero de 2020 y fusionó 10 certificaciones anteriores (CCNA R&S, Wireless, Security, etc.) en una sola: menos exámenes, más amplitud.

## Para tu libreta de apuntes
- Parte 1 hecha: OSI/TCP-IP · IPv4/IPv6 · switching · IOS · seguridad básica · troubleshooting.
- Siguiente: Parte 2 (VLANs, STP, OSPF, ACLs, NAT, automatización).
- Examen: 120 min · 6 dominios (20/20/25/10/15/10) · Pearson VUE · sin marcha atrás.
- Método: repaso espaciado + labs + simulacros legales.
- Carrera: CCNA → especialización → seguridad/cloud/SDN.

## Preguntas de autoevaluación

1. ¿Cuántos dominios tiene el blueprint 200-301 y cuál pesa más? **Respuesta:** 6 dominios; Conectividad IP (3.0) con 25%.
2. ¿Qué práctica de estudio es más eficaz: releer o autoevaluarse? **Respuesta:** El active recall (autoevaluación) + repaso espaciado.
3. ¿Qué recurso oficial define exactamente lo que entra al examen? **Respuesta:** Los "Exam Topics" (blueprint) oficiales de cisco.com para el 200-301.
4. ¿Puedo llevar el CCNA toda la vida sin renovar? **Respuesta:** No: caduca a los 3 años; se renueva con Continuing Education o re-examen.
5. ¿Qué habilidad separa al que "aprobó" del que "sabe"? **Respuesta:** La práctica en laboratorio real/virtual (IOS, packet tracer, diagnóstico).
