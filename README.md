# AI Radar

Asistente de investigación que lee fuentes de IA 3 veces por semana y propone **máximo 3 ideas** para que un equipo de asistentes/agentes de IA gaste menos tokens y esté mejor organizado. No implementa nada sin aprobación.

## Qué hace
1. Lunes, miércoles y viernes revisa las fuentes (ver [`SOURCES.md`](SOURCES.md)).
2. Solo lee texto: feeds RSS y transcripciones vía fetch web. Nunca abre navegador ni escritorio (eso gasta muchos tokens).
3. Filtra solo lo que sirve para orquestación: tokens, contexto, rutinas, skills, handoffs entre agentes, repos útiles.
4. Si no hay nada nuevo, no manda nada.
5. Manda al humano un resumen corto y al orquestador máximo 3 propuestas tipo "¿hacemos esto?".

## Contenido
- [`TOKENS.md`](TOKENS.md): todo lo que hemos hecho para bajar tokens, con resultados medidos.
- [`SOURCES.md`](SOURCES.md): las 6 fuentes y sus filtros.
- [`routine/radar.md`](routine/radar.md): instrucciones de la rutina.
- [`skills/`](skills/): `caveman` (bot↔bot), `i-have-adhd` (bot→humano), `getting-started` (primera conversación).

## Cómo usarlo
Copia las instrucciones de `routine/radar.md` en una rutina de tu asistente (cron sugerido `50 8 * * 1,3,5`), carga las skills y corre `getting-started` para elegir fuentes, horario y enfoque.

Créditos: `caveman` basado en [JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman); `i-have-adhd` basado en [ayghri/i-have-adhd](https://github.com/ayghri/i-have-adhd) (MIT).
