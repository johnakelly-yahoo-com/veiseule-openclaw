# Contribuir al Modelo de Amenazas de OpenClaw

Gracias por ayudar a que OpenClaw sea más seguro. Este modelo de amenazas es un documento vivo y damos la bienvenida a contribuciones de cualquiera; no necesitas ser un experto en seguridad.

## Formas de contribuir

### Añadir una amenaza

¿Has detectado un vector de ataque o un riesgo que no hayamos cubierto? Abre un issue en [openclaw/trust](https://github.com/openclaw/trust/issues) y descríbelo con tus propias palabras. No necesitas conocer ningún framework ni rellenar todos los campos; solo describe el escenario.

**Útil incluir (pero no obligatorio):**

- El escenario del ataque y cómo podría explotarse
- Qué partes de OpenClaw están afectadas (CLI, gateway, canales, ClawHub, servidores MCP, etc.)
- Qué tan grave crees que es (bajo / medio / alto / crítico)
- Cualquier enlace a investigaciones relacionadas, CVE o ejemplos del mundo real

Nosotros nos encargaremos del mapeo ATLAS, los IDs de amenaza y la evaluación de riesgos durante la revisión. Si quieres incluir esos detalles, genial, pero no es obligatorio.

> **Esto es para añadir al modelo de amenazas, no para reportar vulnerabilidades activas.** Si has encontrado una vulnerabilidad explotable, consulta nuestra [página de Trust](https://trust.openclaw.ai) para obtener instrucciones de divulgación responsable.

### Sugerir una mitigación

¿Tienes una idea de cómo abordar una amenaza existente? Abre un issue o un PR haciendo referencia a la amenaza. Las mitigaciones útiles son específicas y accionables; por ejemplo, "limitación de tasa por remitente de 10 mensajes/minuto en el gateway" es mejor que "implementar limitación de tasa".

### Proponer una cadena de ataque

Las cadenas de ataque muestran cómo múltiples amenazas se combinan en un escenario de ataque realista. Si ves una combinación peligrosa, describe los pasos y cómo un atacante las encadenaría. Una breve narrativa de cómo se desarrolla el ataque en la práctica es más valiosa que una plantilla formal.

### Corregir o mejorar contenido existente

Errores tipográficos, aclaraciones, información desactualizada, mejores ejemplos: los PR son bienvenidos, no se necesita issue.

## Qué utilizamos

### MITRE ATLAS

Este modelo de amenazas se basa en [MITRE ATLAS](https://atlas.mitre.org/) (Panorama de Amenazas Adversarias para Sistemas de IA), un marco diseñado específicamente para amenazas de IA/ML como inyección de prompts, uso indebido de herramientas y explotación de agentes. No necesitas conocer ATLAS para contribuir; durante la revisión asignamos las propuestas al marco.

### Identificadores de amenazas

Cada amenaza recibe un ID como `T-EXEC-003`. Las categorías son:

| Código  | Categoría                                  |
| ------- | ------------------------------------------ |
| RECON   | Reconocimiento - recopilación de información     |
| ACCESS  | Acceso inicial - obtención de acceso             |
| EXEC    | Ejecución - ejecución de acciones maliciosas      |
| PERSIST | Persistencia - mantenimiento del acceso           |
| EVADE   | Evasión de defensas - evitación de la detección       |
| DISC    | Discovery - learning about the environment |
| EXFIL   | Exfiltration - stealing data               |
| IMPACT  | Impact - damage or disruption              |

IDs are assigned by maintainers during review. You don't need to pick one.

### Risk Levels

| Level        | Meaning                                                           |
| ------------ | ----------------------------------------------------------------- |
| **Critical** | Full system compromise, or high likelihood + critical impact      |
| **Alto**     | Significant damage likely, or medium likelihood + critical impact |
| **Medium**   | Moderate risk, or low likelihood + high impact                    |
| **Low**      | Unlikely and limited impact                                       |

If you're unsure about the risk level, just describe the impact and we'll assess it.

## Review Process

1. **Triage** - We review new submissions within 48 hours
2. **Assessment** - We verify feasibility, assign ATLAS mapping and threat ID, validate risk level
3. **Documentation** - We ensure everything is formatted and complete
4. **Merge** - Added to the threat model and visualization

## Resources

- [ATLAS Website](https://atlas.mitre.org/)
- [ATLAS Techniques](https://atlas.mitre.org/techniques/)
- [ATLAS Case Studies](https://atlas.mitre.org/studies/)
- [OpenClaw Threat Model](./THREAT-MODEL-ATLAS.md)

## Contact

- **Security vulnerabilities:** See our [Trust page](https://trust.openclaw.ai) for reporting instructions
- **Threat model questions:** Open an issue on [openclaw/trust](https://github.com/openclaw/trust/issues)
- **General chat:** Discord #security channel

## Recognition

Contributors to the threat model are recognized in the threat model acknowledgments, release notes, and the OpenClaw security hall of fame for significant contributions.
