# Temarios

Recopilación de **temarios y materiales de estudio** generados a partir de cursos en línea, pensados para autoestudio: teoría por sección, ejercicios, ejemplos, tips y datos curiosos, más una auditoría técnica del contenido.

> ⚠️ **Aviso**
> - Este repo contiene **apuntes y temarios escritos por IA/humanos**, no vídeos ni transcripciones de los cursos originales.
> - No está afiliado a Udemy, Cisco ni ninguna certificación u organización.
> - Si estudias para una certificación oficial (p. ej. CCNA), la fuente de verdad es siempre el blueprint y la documentación oficial del proveedor.
> - Úsalo bajo tu propio criterio y respeta los Términos de Servicio del sitio donde tomaste el curso.

## Estructura

```text
.
├── .opencode/agents/          # Agentes para generar y auditar el material
│   ├── ccna-autor.md          # Genera secciones/<NN-slug>/tema.md a partir de un temario.md
│   └── ccna-evaluador.md      # Auditoría técnica y pedagógica del material (solo lectura)
└── cursos/
    ├── ccna1/
    │   ├── temario.md         # Temario del curso (secciones y clases)
    │   └── secciones/
    │       └── 01-.../tema.md # Material por sección: teoría, ejemplos, ejercicios, tips…
    └── solid-y-clean-code/
        └── temario.md
```

## Cursos incluidos

| Curso | Estado |
| --- | --- |
| [Certificación Cisco CCNA 200-301: Introducción a las Redes — Parte 1](cursos/ccna1/temario.md) | Material completo (20 secciones), auditado y con correcciones aplicadas |
| [SOLID y Clean Code](cursos/solid-y-clean-code/temario.md) | Solo temario (pendiente de generar material) |

## Uso de los agentes

Con [OpenCode](https://opencode.ai) en este directorio:

```text
Usa el subagent ccna-autor para generar el material a partir de cursos/ccna1/temario.md

Usa el subagent ccna-evaluador para auditar cursos/ccna1/secciones/
```

Los agentes actuales están orientados a CCNA 1; para un curso nuevo basta con copiarlos y adaptar las rutas y el dominio.

## Contribuciones

PRs de correcciones técnicas bienvenidas. Si encuentras un error conceptual o un comando incorrecto, una issue con la fuente oficial (RFC, doc de Cisco, docs de la librería, etc.) ayuda mucho.
