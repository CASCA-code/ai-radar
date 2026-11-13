# Cómo bajamos tokens

## 1. Estilo de escritura
1. **Caveman entre bots.** Todo mensaje, brief o reporte entre agentes va comprimido: sin artículos, relleno ni cortesías; términos técnicos, números y negaciones intactos. Prosa normal solo para el humano. Skill: [`skills/caveman`](skills/caveman/SKILL.md).
2. **i-have-adhd hacia el humano.** Acción primero, pasos numerados, máximo 5 elementos por lista, un solo siguiente paso, sin preámbulo ni despedida. Mensajes más cortos y que sí se leen. Skill: [`skills/i-have-adhd`](skills/i-have-adhd/SKILL.md).
3. **Sin ping-pong.** Los agentes no se mandan "recibido/gracias". Solo se escribe si hay algo nuevo o una pregunta.

## 2. Handoffs y contexto
1. **Brief mínimo al delegar.** Nunca se reenvía el historial del chat; solo objetivo, datos necesarios, criterio de éxito y qué reportar.
2. **Auto-verificación antes de entregar.** El ejecutor revisa su resultado contra el criterio antes de devolverlo, para no gastar una vuelta extra de corrección.
3. **Nivel de esfuerzo por tipo de tarea.** Triage y tareas mecánicas en modelo/esfuerzo barato; lo difícil en esfuerzo alto.
4. **Herramientas por dominio.** Cada turno carga solo los conectores del proyecto en curso, no todos.
5. **Compuertas con reglas.** Decisiones de ruteo con reglas fijas (sí/no, score) en vez de prosa larga; un modelo solo si las reglas no alcanzan.

6. **Cache-shape.** El contexto se arma con un bloque estable primero (skills, esquemas, instrucciones) y lo que cambia al final (fecha, estado de la tarea), para aprovechar el caché de entrada, que cuesta ~95% menos.
7. **Notas de implementación.** En tareas de esfuerzo alto, el ejecutor cierra con 3 a 5 líneas de qué alternativas vio y por qué las descartó, para evitar retrabajo.
8. **Presupuesto de 1 reintento.** Si la autoverificación falla una vez, el ejecutor escala al orquestador con lo que falta; no rehace en bucle.

9. **Tope de acciones por rutina.** Cada rutina dice "tope N tool calls por turno; si excede, reporta y para".
10. **Escalera de esfuerzo.** Toda tarea arranca en esfuerzo bajo; sube a alto solo si la verificación falla, y a un trabajador más potente solo si vuelve a fallar.
11. **Re-anclar el pedido.** Antes de la respuesta final, el bot vuelve a leer el pedido original si las herramientas llenaron el contexto.
12. **Relevo a ~20 turnos.** Pasados ~20 turnos, se resume y la tarea sigue en un ayudante nuevo con un brief corto. El relevo usa 3 bloques ([CarryOn](https://ethannju.github.io/carryon/)): **carriers** (rutas a archivos/artefactos y cuándo sirven, no su contenido), **contexto de ejecución** (metas, restricciones, pendientes abiertos) y **cola viva** (pasos desde el último relevo). Se actualiza por partes, sin rehacer todo. En el paper: −52% tokens y −36% costo por tarea con más aciertos.
13. **Entregables en su lugar.** El resultado se guarda donde se usa y se pasa la ruta, no el contenido completo.
14. **Código a editar, sin comprimir.** Para editar código se cargan 2-3 archivos enteros tal cual más fragmentos de `grep`; nunca markitdown, headroom ni otra compresión, porque alteran el texto y los parches no aplican. Para elegir archivos se pregunta 2 veces cuáles cambian y se quedan los que coinciden ([Skip the Compressor](https://arxiv.org/abs/2610.21073): +6.3 pts vs el mejor compresor).
15. **Plan con compuertas para el ejecutor barato** (2026-11-13). En tareas largas o de código, el modelo caro escribe un plan autocontenido y un ejecutor barato solo lo ejecuta. Cada paso lleva un comando de verificación y el resultado esperado. El plan dice qué queda fuera de alcance, cuándo parar (STOP) y sobre qué commit o fecha se escribió. Los hallazgos rechazados quedan con su razón. Fuente: [shadcn/improve](https://github.com/shadcn/improve).
16. **Relevo único caro→barato en tareas largas** (2026-11-13). Se planea con esfuerzo alto y se ejecuta con esfuerzo bajo. Nunca se pasa de barato a caro a mitad de camino, salvo que falle una compuerta (escalera). Claim del paper [AgentBaton](https://arxiv.org/abs/2610.35959): −65.9% de costo de API; no medido aquí.
17. **Cambios quirúrgicos** (2026-11-13). Cada línea cambiada responde al objetivo. No se "mejora" el código vecino, y el código muerto ajeno se reporta, no se borra. Fuente: [andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills).
18. **No invalidar el caché** (2026-11-13). Los prompts de rutina y el inicio de los briefs no llevan fecha, hora, ids ni rutas variables; el bot saca la fecha con `date`. El historial solo crece: no se editan ni se reordenan mensajes o herramientas a mitad de la tarea. Claim de Agent Arena (26K sesiones): sin caché el costo es 3.6× y cada invalidación suma ~5%; no medido aquí.
19. **No resumir a mitad de tarea** (2026-11-13). El historial no se compacta por umbral; si crece, se hace un relevo (punto 12). La salida grande de una herramienta va a un archivo y se pasa la ruta. Claim del paper [AKV](https://arxiv.org/abs/2610.28577): resumir por umbral cuesta −5 pts de calidad por −7% de costo; no medido aquí.

## 2b. Memoria y organización
1. **Índice por proyecto.** Cada proyecto tiene un `INDEX.md` de una página; los bots lo leen antes de abrir documentos completos.
2. **Limpieza mensual de memoria ("dreaming", ampliada 2026-11-13).** Una vez al mes:
   - Se juntan los duplicados.
   - Cada nota se vuelve a validar contra su evidencia cruda (transcript o archivo) y se conserva la ruta a esa evidencia; no se reemplaza por un resumen.
   - Las notas obsoletas se marcan y se proponen para borrar, pero solo se borran con el OK del humano.
   - Se guardan aciertos y fallos, no solo fallos.
   - Fuente: [Anthropic Engineering, 2026-11-06](https://www.anthropic.com/engineering). Claim de CL-Bench: 0.655→0.838 con memoria; no medido aquí.
3. **Error repetido 2 veces = regla fija.** Si un bot comete el mismo error dos veces, se guarda como regla permanente en su perfil. Cada regla guarda su fuente (fecha, caso y evidencia). En la limpieza mensual se vuelve a revisar; si el caso ya no aplica, se propone borrarla y se borra solo con OK del humano ([Anti-Persistence](https://arxiv.org/abs/2610.27406): las reglas guardadas pueden repetir sus propios errores).

## 3. Herramientas y fuentes
1. **Conector/API/CLI primero; navegador al último.** El navegador y el escritorio cuestan muchísimo más. Capturas de pantalla solo en puntos de decisión o como prueba, nunca por paso.
2. **YouTube por RSS + transcripción**, nunca viendo el video en navegador.
3. **Rutinas en silencio** cuando no hay nada nuevo, y `last_seen` por fuente para no releer lo ya visto.
4. **Tope de 3 propuestas** y memoria de lo adoptado/declinado para no re-proponer.

## 4. Pruebas medidas (2026-09)
Tokens contados antes/después y 5 preguntas de control por caso para detectar pérdida de información.

| Caso | Herramienta | Antes | Después | Ahorro | Info intacta | Veredicto |
|---|---|---:|---:|---:|---|---|
| Transcripción YouTube 15 min | [markitdown](https://github.com/microsoft/markitdown) | 68,724 | 4,813 | 93% | 5/5 | adoptar |
| PDF de texto (15 págs) | markitdown | 2,364 | 2,327 | 2% | 5/5 | usar `pdftotext` |
| PDF de slides (28 págs) | markitdown | 3,402 | 5,002 | −47% | 4/5 | no |
| JSON array (123 filas) | [headroom](https://github.com/chopratejas/headroom) | 21,281 | 10,397 | 51% | 5/5 | adoptar |
| CSV (123×38) | headroom | 46,828 | 7,378 | 84% | 1/5 | NO: corrompe datos |
| Respuesta API commits | headroom | 51,899 | 26,087 | 50% | 3/5 | parcial (muestrea) |
| Logs de build / sistema | headroom | 83k–233k | — | 0% o pierde todo | — | no |

Regla resultante: markitdown solo para YouTube/Office; PDFs con `pdftotext`; headroom solo para arrays JSON homogéneos, nunca CSV ni logs.

## Escalera de esfuerzo (adoptada 2026-10-28)
Fuente: arxiv 2610.34491 (repo KaiserWhoLearns/ReasoningEffortLLM).
- Arrancar cada tarea con esfuerzo mínimo.
- Saltar directo a MÁXIMO solo si falla un check objetivo (test, validación, dato no confirmado). Nunca por juicio del bot.
- Sin pasos intermedios: mín → máx.
- Nunca apagar el razonamiento como ahorro (las trayectorias se alargan).

## 4b. Prueba piloto 2026-11-13: modern-web-guidance (medido por nosotros)
- Herramienta: [GoogleChrome/modern-web-guidance](https://github.com/GoogleChrome/modern-web-guidance) v1.0.0, con telemetría apagada (`DISABLE_TELEMETRY=1`) y sin red.
- Tamaño: clon shallow de 3.9 MB más la CLI de npm de 17 MB, sin dependencias. La instalación tardó 1.2 s.
- `legacy-scan` sobre un repo web propio (mapa estático, 16 archivos de código):
  - Tardó 7.6 s.
  - Dio 3 hallazgos, todos de la misma regla (Enter sin control de IME, que afecta sobre todo a la escritura CJK). Para nuestros usuarios, el impacto es bajo.
  - El reporte JSON pesa 735 tokens.
- `search` pesa 358 tokens y `retrieve` de una guía, 1,537 tokens (cl100k).
- Veredicto:
  - `search` y `retrieve` se adoptan bajo demanda, solo para tareas de UI web, cuando se va a construir un componente concreto.
  - `legacy-scan` no se usa de rutina porque dio 0 hallazgos de alto valor.

## 5. Evaluado y descartado
- **Context Mode**: si recorta un detalle necesario, se re-pide todo y sale más caro.
- **Memorias externas (AgentMemory, GBrain, etc.)**: solo como ideas si ya tienes memoria integrada.
- **RTK** (comprime output de terminal): útil solo para bots que corren muchos comandos.
- **Hook anti-fuga de email en headers** (2026-10-26): la plataforma no tiene hooks pre-tool propios; si hace falta, va como regla de Auto-review.
- **Catálogo de skills filtrado por bot** (2026-10-26): las skills son una librería global compartida; no se puede limitar por bot.
- **AKV real (descartar KV de herramientas)** (2026-11-13): requiere servir el modelo nosotros (SGLang). En API solo aplica la regla de texto §2.19.
- **Graphify** (2026-11-13): nuestros repos son pocos archivos grandes; un grafo AST no ahorra frente a `grep` + 2-3 archivos (§2.14).
- **Graft** (sigue NO, 2026-11-13): sus hooks de shell son para Claude Code y la plataforma no tiene hooks propios (ver hook PII).
- **Ruflo / enjambres de 60+ agentes** (2026-11-13): multiplican tokens; van contra la meta.
- **Codebase Memory MCP** (2026-11-13): mismo nicho que Pensieve/Graft (NO 2026-10-19); sin evidencia nueva.
- **modern-web-guidance `legacy-scan` de rutina** (2026-11-13): 0 hallazgos de alto valor en el piloto (§4b).

## 6. Aprobado por el humano 2026-11-13
Implementado: §2.15–2.19, §2b.2 (dreaming) y §4b (`search`/`retrieve` bajo demanda). Medido aquí: solo §4b. Lo demás son claims de sus fuentes. Descartado: ver §5 (entradas 2026-11-13).
