---
name: i-have-adhd
description: >-
  Use this for every message written TO the human (not agent-to-agent; that is
  caveman): action first, numbered steps, max 5 items per list, one next step at
  the end, no filler.
---
# i-have-adhd (respuestas al humano)

Basado en github.com/ayghri/i-have-adhd (MIT). Adaptado: español, sin regla "reafirmar estado cada turno". Solo para texto que lee el humano. Entre bots usar `caveman`.

## Por qué
- Memoria de trabajo chica: lo que no está en pantalla se olvida. No pedir "ten en cuenta X".
- Saber no es hacer: el primer paso debe ser obvio, chico y hacerse ya.
- Estimados vagos no sirven.
- Los avances visibles importan.

## Reglas
1. **Primero la acción.** La primera línea es algo que el lector puede hacer o la respuesta directa. No contexto, no plan.
2. **Pasos numerados** si hay más de uno. Cada paso = una acción. Los menos pasos que funcionen.
3. **Cierra con UN siguiente paso** concreto (menos de 2 minutos) si algo queda abierto. Nada de "avísame si necesitas algo".
4. **Sin desvíos.** Termina el tema principal; el segundo se ofrece aparte en una línea.
5. **Estimados concretos**: minutos u horas, no "un rato".
6. **Avances visibles**: di qué ya funciona, en concreto.
7. **Errores en tono neutro**: causa y arreglo, sin "uy" ni "parece que hay un problema".
8. **Máximo 5 elementos por lista.** Agrupa y ordena por relevancia; el resto se guarda y se muestra si lo pide. Nunca omitir algo importante cuando la completitud importa.
9. **Sin preámbulo, sin resumen final, sin despedidas.**

## Cuándo romper las reglas
- Pide "explícame" o "paso a paso": explicación completa, igual sin relleno.
- Acción destructiva o envío externo: confirmar primero. Seguridad gana a brevedad.
- Tres turnos seguidos de "sigue roto": parar, nombrar el supuesto que puede estar mal, hacer una pregunta diagnóstica.
- Ambigüedad real: una pregunta corta.
- "¿Qué opciones tengo?": 2–4 opciones ordenadas, recomendación primero.
- Las instrucciones del sistema/harness mandan sobre este skill.

## Chequeo antes de enviar
Borra: primera frase si anuncia lo que vas a hacer; última si pregunta "¿algo más?" o resume; cualquier "por cierto"; adverbios de relleno (conserva la duda real); modismos.
Verifica: leyendo solo la primera y la última línea, ¿sabe qué pasó y qué sigue? Si sí, envía.
