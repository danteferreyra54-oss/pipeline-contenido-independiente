# Pipeline de Contenido — Medio Independiente

Ecosistema de automatización IA para la generación de contenido periodístico sobre Club Atlético Independiente. Entrega final — CoderHouse, curso de IA Automation.

## Objetivo

Automatizar el monitoreo de fuentes RSS deportivas, la redacción de borradores periodísticos con IA y su publicación, manteniendo siempre un punto de validación humana antes de cualquier acción crítica.

## Stack (4 tecnologías integradas)

| Tecnología | Rol |
|---|---|
| **n8n** (self-hosted) | Orquestador del flujo completo |
| **Notion** | Base de datos / memoria del sistema |
| **OpenAI (GPT-4o-mini)** | Generación de borradores periodísticos |
| **Gmail** | Canal de salida y validación humana (HITL) |

## Flujo del sistema

1. **Trigger cada 3 horas** dispara el pipeline sin intervención manual.
2. Se obtienen las fuentes RSS activas desde Notion y se separan.
3. Se leen los feeds RSS (~123 items por corrida).
4. **Remove Duplicates** filtra las noticias ya procesadas en corridas anteriores (evita reprocesar y gastar tokens de más).
5. **OpenAI GPT-4o-mini** redacta un borrador con prompt 100% dinámico (sin datos hardcodeados).
6. El borrador se guarda en Notion con Estado = `Procesado por IA`.
7. Se envía un mail con el borrador y dos botones: **Aprobar** / **Rechazar**.
8. El click dispara un Webhook que actualiza el Estado en Notion (`Aprobado por humano` o `Rechazado`) vía PATCH a la API — **ninguna acción se ejecuta sin esa validación humana**.

### Ruta de error (resiliencia)

Si la llamada a OpenAI falla (autenticación, timeout, rate limit), la salida `Error` del nodo dispara una ruta alternativa que registra en Notion: Estado = `Error`, el mensaje de error real y la URL de la fuente. El resto del pipeline sigue funcionando con normalidad — el fallo no corta el flujo en silencio.

Esta ruta fue **validada con un test de estrés real**: se forzó una credencial de OpenAI inválida a propósito, se confirmó que el sistema capturaba el error correctamente en Notion, y se corrigieron 3 bugs de mapeo de variables detectados en el proceso (ver capturas en `/screenshots`).

## Contenido de este repositorio

- `Diagrama_Arquitectura_Pipeline_Independiente.pdf` — diagrama de arquitectura + descripción de decisiones de diseño y check de seguridad.
- `pipeline_contenido_independiente.json` — export del workflow de n8n, listo para importar (las credenciales no se incluyen; deben configurarse aparte al importar).
- Capturas de evidencia del camino infeliz (test de estrés) y del funcionamiento del error handling.

## Base de datos (Notion)

Link en modo lectura: *(agregar aquí)*

La tabla **Artículos** funciona como memoria central del sistema, con campos de estado (`Procesado por IA`, `Aprobado por humano`, `Rechazado`, `Error`), log de error y URL fuente.

## Check de seguridad

- ✅ Filtro contra bucles infinitos — Remove Duplicates evita reprocesar artículos ya vistos.
- ✅ Comparación de tipos consistente — el dedupe compara strings (URL) contra strings.
- ✅ Prompt dinámico — ninguna variable del sistema está hardcodeada.
- ✅ Nodos nombrados claramente en todo el flujo.
- ✅ Credenciales gestionadas fuera del flujo, vía credenciales de n8n.
- ✅ Rutas de error probadas con un camino infeliz real, no solo teórico.

## Autor

Dante Ferreyra
