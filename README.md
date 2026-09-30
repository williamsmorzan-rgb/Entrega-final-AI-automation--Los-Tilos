# Ecosistema de Automatización IA — Los Tilos Pádel

**Entrega Final · AI Automation (Coderhouse)** · Williams Morzan · Septiembre 2026

Pipeline de contenido con **RAG** y **Human-in-the-Loop** para un complejo de pádel de 4 canchas en Corrientes, Argentina.
Una idea semilla se transforma en un post con la voz de la marca a partir de una base de conocimiento del negocio. Nada llega a los clientes hasta que una persona lo aprueba. Después de la aprobación, el contenido se envía solo a los suscriptores que dieron consentimiento y todo queda registrado.

## Enlaces

| Recurso | Enlace |
|---|---|
| Base de datos (Airtable, solo lectura) | https://airtable.com/appuGAt5mttAnJTuz/shrJIW4dTcWIIEglV |
| Dashboard de control (KPIs y tasa de error) | https://airtable.com/appuGAt5mttAnJTuz/shrNA7eyvJjHApYEc |
| Video demo (3 min) | _(agregar enlace)_ |

## Stack

| Categoría | Herramienta | Uso |
|---|---|---|
| Orquestador | Make | 2 escenarios: Generación y Publicación |
| Base de datos | Airtable | 4 tablas vinculadas: Base de Conocimiento, Centro de Comando, Log de Ejecuciones y Suscriptores |
| Procesamiento IA | Anthropic Claude Haiku 4.5 | Redacción con RAG, prompt dinámico y `max_tokens` 500 |
| Canal de salida | Gmail | Notificación HITL, envío a suscriptores y respuesta en el mismo hilo (Thread ID) |

## Cómo funciona

1. **Escenario 1 · Generación.** Detecta ideas con Estado = Pendiente y valida que no estén vacías. Si están vacías, toma la ruta de error. Después consulta la base de conocimiento (RAG), Claude redacta el borrador, se guarda como *En revisión*, se notifica al revisor por Gmail y se registra en el Log.
2. **Human-in-the-Loop.** El revisor lee el borrador y tilda **Aprobado** en Airtable.
3. **Escenario 2 · Publicación.** Detecta el contenido aprobado, lo envía a cada suscriptor con consentimiento, marca la idea como *Publicado*, confirma en el hilo del mail de revisión y registra en el Log.

Las fallas de la API de IA o de Gmail se capturan con **error handlers**. Se registra el mensaje de error y la ejecución se guarda para reintento (directiva **Retry**, equivalente a *Break*).

## Contenido del repositorio

| Archivo | Descripción |
|---|---|
| `Entrega_Final_Ecosistema_IA_Los_Tilos.pdf` | Documento principal: arquitectura, manual de datos, matriz de costos, seguridad y resiliencia, dashboard |
| `Evidencias_Entrega_Final_Los_Tilos.pdf` | Capturas ordenadas: tablas, configuración, test de estrés (5 corridas) e incidentes resueltos |
| `blueprint-esc1-generacion.json` | Blueprint del Escenario 1 (Make) |
| `blueprint-esc2-publicacion.json` | Blueprint del Escenario 2 (Make) |

## Check de seguridad

- **Filtro anti-bucle:** cada trigger filtra por estado (`Pendiente` y `Aprobado + En revisión`) y el flujo cambia el estado al procesar.
- **Tipos de datos:** `Exists` / `Does not exist` sobre texto, casilla como booleano y `FIND` numérico.
- **Prompt dinámico:** usa `{{Idea Semilla}}` y el contexto RAG recuperado de la base, sin datos hardcodeados.
- **Minimización:** a la IA solo llegan la idea y el contexto del negocio. Los datos de los suscriptores nunca pasan por Claude.
- **Credenciales:** están guardadas en las conexiones de Make y no se incluyen en los blueprints.
