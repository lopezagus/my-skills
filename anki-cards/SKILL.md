---
name: anki-cards
description: Generate Anki flashcards from a markdown course notes file and write them to a .md output file in Obsidian-to-Anki format. Use when user wants to generate flashcards, create Anki cards, or mentions "card" alongside a notes file or course topic.
---

# Anki Flashcard Generator

## Quick start

1. `view` the input markdown file
2. Identify the topic and language of the notes
3. Generate cards using the rules below
4. Write output to `{original_filename}_cards.md`
5. Present the file

## Card format

Each card uses the Obsidian-to-Anki plugin format. Numbered comment before each card for review reference:

```
<!-- 1 -->
START
Basic
Front: ¿Pregunta aquí?
Back: Respuesta aquí.
Tags: específico, general
END

<!-- 2 -->
START
Basic
Front: ...
Back: ...
Tags: ...
END
```

No preamble. No section headers. No commentary between cards.

## Language rule

Always generate cards in the **same language as the input notes**. Do not translate.

## Card generation rules

**Card-worthy content:**
- Definitions of key terms
- Laws, principles, and theorems
- Causal or logical relationships ("why" questions)
- Formulas and their meaning
- Reasoning steps worth internalizing

**Skip:**
- Trivial or self-evident facts
- Restatements of concepts already carded
- Transitional prose with no testable content

**Selectivity:** prefer depth over volume. If forgetting it wouldn't hurt on an exam or in practice, skip it.

## MathJax rules

Preserve MathJax as-is inside Front/Back fields. Add a variable legend on Back only when symbols are non-obvious:

```
<!-- 3 -->
START
Basic
Front: ¿Cuál es la fórmula de la energía libre de Gibbs?
Back: $$\Delta G = \Delta H - T\Delta S$$
Donde: ΔG = cambio en energía libre de Gibbs, ΔH = cambio de entalpía, T = temperatura (K), ΔS = cambio de entropía.
Tags: termodinámica, química
END
```

## Tagging rules

Each card gets 2 tags: **specific → general**, inferred from the content.

- Specific: the concept cluster within the unit (e.g., `entalpía`, `ácidos`, `enlace-covalente`)
- General: the course or discipline (e.g., `química`, `física`, `cálculo`)

Use lowercase and hyphens for multi-word tags.

## Output file

- Filename: `{original_filename}_cards.md`
- End the file with this comment: `<!-- ¿Falta algún concepto? Responde con el tema y genero más tarjetas. -->`
