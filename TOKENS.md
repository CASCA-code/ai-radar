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

## 5. Evaluado y descartado
- **Context Mode**: si recorta un detalle necesario, se re-pide todo y sale más caro.
- **Memorias externas (AgentMemory, GBrain, etc.)**: solo como ideas si ya tienes memoria integrada.
- **RTK** (comprime output de terminal): útil solo para bots que corren muchos comandos.
