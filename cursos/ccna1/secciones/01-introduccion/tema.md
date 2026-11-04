# 01 — Introducción

> Duración del curso: 22m · 6 clases
> Fuente: temario.md del curso

## Objetivos de aprendizaje
- Descargar e instalar Cisco Packet Tracer y crear/usar una cuenta en NetAcad.
- Reconocer qué herramientas y recursos acompañan al curso (asesoramiento, preguntas).
- Preparar el entorno de laboratorio para el resto del curso.
- Conocer las recomendaciones de estudio y el alcance del curso dentro de la certificación CCNA 200-301.

## Teoría

Esta sección es administrativa: no introduce teoría técnica nueva, pero sienta las bases del entorno de trabajo. Cubre las clases 1.1 a 1.6.

### 1.1 Recomendaciones para una mejor experiencia en el curso
- Estudiar con **Packet Tracer** abierto en paralelo, no solo viendo vídeos: la memoria muscular de la CLI (command-line interface, interfaz de línea de comandos) es lo que el examen evalúa.
- Tomar notas propias (libreta de apuntes), pausar y repetir los comandos.
- Hacer los quizzes de cada módulo al terminar la sección; son la forma de autoevaluarse.
- Seguir el orden del curso: las secciones son acumulativas (Primero conceptos, luego IOS, luego capas).
- Practicar **subnetting** a diario desde el principio, aunque aún no lo dominen; es la habilidad que más se atrofia si no se entrena.

### 1.2 Descubrir la Ciberseguridad
- En el curso se menciona una ruta de aprendizaje enfocada a ciberseguridad. Es **opcional**: no afecta al CCNA. Si tu objetivo es CCNA 200-301, puedes omitirla sin perder nada del temario oficial.

### 1.3 Preparativos: descargar e instalar Packet Tracer (Clase 1.3)
**Cisco Packet Tracer** es un simulador/emulador de redes gratuito de Cisco que permite construir topologías con routers, switches, PCs y cables virtuales, y practicar la CLI de IOS sin hardware real.

Pasos habituales:
1. Crear una cuenta gratuita en **Cisco Networking Academy (NetAcad)**: https://www.netacad.com
2. Iniciar sesión y buscar "Packet Tracer" en el catálogo de recursos/descargas.
3. Descargar la versión para tu sistema operativo (Windows, macOS o Linux).
4. Instalar aceptando los valores por defecto; no requiere licencias ni claves.
5. Abrir Packet Tracer y comprobar que carga la ventana principal con la barra de dispositivos (Routers, Switches, End Devices, etc.).

### 1.4 Problemas al registrarte en NetAcad
- La descarga de Packet Tracer exige iniciar sesión en NetAcad. Problemas típicos: correo no verificado, cuenta creada con otro proveedor, o filtro corporativo/escolar que bloquea la web.
- Soluciones: verificar el correo de confirmación, probar otro navegador/incógnito, usar una conexión distinta (datos móviles), o contactar con soporte de NetAcad.
- Alternativa si NetAcad falla: versiones antiguas y espejos existen en la web, pero la vía soportada y segura es NetAcad; no descargues ejecutables de terceros desconocidos.

### 1.5 / 1.6 Asesoramiento y respuestas a preguntas
- En el curso se ofrece un canal de dudas (preguntas de alumnos respondidas por el instructor/comunidad). Úsalo para atascos reales: comparte la topología, el comando exacto y la salida obtenida.
- Cómo formular una buena pregunta técnica: qué intentas hacer, qué comando usaste, qué salida obtuviste, qué esperabas.

## Ejemplos

### Ejemplo 1 — Entorno mínimo de trabajo
1. Instala Packet Tracer 8.x (la versión exacta puede variar; cualquier versión reciente sirve para este curso).
2. Crea un proyecto nuevo → arrastra un router `2911`, un switch `2960` y dos PCs → conéctalos con cable de cobre directo (straight-through).
3. Guarda como `lab01.pkt`. Ese es tu entorno base para todo el curso.

### Ejemplo 2 — Verificación de la instalación
Abre Packet Tracer y confirma:
- La barra inferior muestra categorías: Routers, Switches, Hubs, Wireless Devices, End Devices, Security, WAN Emulation, Connections.
- Puedes colocar un PC y hacer doble clic en él: abre las pestañas Desktop, Config y Physical.
Si algo de esto no aparece, la instalación no terminó bien; reinstala.

### Ejemplo 3 — Uso del canal de dudas (plantilla)
> "Estoy en la sección 3. En el router, tras `enable` y `configure terminal`, al escribir `hostname R1` el prompt no cambia. Adjunto captura. Esperaba que el prompt pasara de `Router(config)#` a `R1(config)#`."
Esta formulación (objetivo + pasos + resultado + esperado) multiplica las probabilidades de una respuesta útil.

## Ejercicios

1. **(Recordar)** ¿De dónde se descarga oficialmente Packet Tracer y qué requisito previo hace falta?
2. **(Recordar)** ¿Cuál es el canal recomendado para resolver dudas del curso?
3. **(Aplicar)** Instala Packet Tracer y construye una topología con 1 router 2911, 1 switch 2960 y 2 PCs unidos con cables copper straight-through. Captura/anota qué categorías de dispositivos ves en la barra de herramientas.
4. **(Aplicar)** Escribe con tus palabras las 5 recomendaciones de estudio de la clase 1.1 y explícale el plan a un compañero en 3 frases.
5. **(Analizar)** Un alumno dice: "Ya sé instalar cosas, me salto Packet Tracer y solo veo los vídeos". Da dos argumentos concretos por los que es una mala idea para el CCNA.
6. **(Analizar)** NetAcad te exige verificar el correo y no llega ningún email de confirmación. Enumera 3 acciones ordenadas para resolverlo.

<details>
<summary>Soluciones</summary>

1. Se descarga desde Cisco NetAcad (netacad.com) tras registrarse e iniciar sesión (cuenta gratuita).
2. El canal de preguntas/asesoramiento del curso (y su comunidad), además de la documentación de Cisco.
3. Debe verse la barra de categorías (Routers, Switches, End Devices, Connections...). Si el PC no abre al hacer doble clic, revisa la instalación.
4. Respuesta libre; deben aparecer: practicar en paralelo, tomar notas, hacer los quizzes, seguir el orden, practicar subnetting a diario.
5. (a) El examen evalúa configuración real e interpretación de salidas de CLI, no solo teoría; (b) los vídeos no generan la habilidad de teclear comandos ni diagnosticar errores; sin práctica el conocimiento se olvida rápido (curva del olvido).
6. Revisar carpeta de spam/promociones → esperar unos minutos y reenviar verificación → probar otro navegador/red o contactar con soporte de NetAcad. (Y comprobar que el correo usado es el correcto.)
</details>

## Tips
- Instala Packet Tracer **antes** de seguir a la sección 2; lo necesitarás en la sección 3 y siguientes.
- Versión del curso: se menciona Packet Tracer por ser la herramienta oficial de NetAcad para CCNA; cualquier versión 8.x es válida.
- No confundas Packet Tracer (simulador de redes, solo Cisco) con GNS3 o EVE-NG (emulan imágenes reales de IOS). Para CCNA, Packet Tracer sobra.
- Anota tu versión instalada: algunos comandos del curso pueden diferir levemente entre versiones.

## Datos curiosos
- Packet Tracer fue creado por Cisco como apoyo a la iniciativa de educación de NetAcad y solo puede descargarse con cuenta registrada; es la misma herramienta usada en las academias oficiales de CCNA en todo el mundo.
- La certificación CCNA 200-301 unificó (desde 2020) las antiguas certificaciones como CCNA Routing & Switching en un solo examen de 120 minutos.

## Para tu libreta de apuntes
- NetAcad → cuenta gratis → descargar Packet Tracer (oficial).
- Flujo de estudio: vídeo → practicar en PT → quiz del módulo → repasar fallos.
- Atajo mental: CCNA = teoría + CLI + subnetting. Entrena los tres cada día.
- Recursos: netacad.com · curso · canal de dudas.

## Preguntas de autoevaluación

1. ¿Qué herramienta usa el curso para practicar la CLI sin hardware físico? **Respuesta:** Cisco Packet Tracer.
2. ¿Dónde se obtiene legalmente Packet Tracer? **Respuesta:** En Cisco Networking Academy (netacad.com), con registro gratuito.
3. La clase 1.2 sobre ciberseguridad es… **Respuesta:** Opcional; se puede omitir sin afectar al temario CCNA.
4. Verdadero/Falso: "Puedo aprobar el CCNA viendo solo los vídeos sin tocar la CLI". **Respuesta:** Falso; el examen evalúa interpretación de configuración y salidas.
5. Menciona dos problemas típicos de registro en NetAcad y una solución para cada uno. **Respuesta abierta:** correo sin verificar (reenviar/verificar, revisar spam), navegador/red que lo bloquea (incógnito, otra red, soporte NetAcad).
