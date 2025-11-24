# Comparación de Prompts - Experimentación con Prompt Engineering

**Proyecto:** LTI - Sistema ATS  
**Autor:** Antonio José Muñoz (AJM)  
**Fecha:** 24 de noviembre de 2025  
**Objetivo:** Comparar 5 técnicas de prompt engineering para generación de User Stories

---

## Resumen Ejecutivo

He experimentado con **5 técnicas diferentes de prompt engineering** basadas en la guía de [promptingguide.ai](https://www.promptingguide.ai/es) para generar User Stories del sistema LTI:

1. **Prompt Simple (Zero-shot)**: Petición directa sin contexto extenso
2. **Prompt Few-Shot**: Incluye ejemplo de User Story como referencia
3. **Prompt Chain-of-Thought**: Razonamiento paso a paso con análisis MoSCoW
4. **Prompt JSON Estructurado**: Output en formato JSON parseable
5. **Prompt ReAct**: Role prompting + Reasoning + Acting con métricas cuantitativas

**Resultado:** El **Prompt 5 (ReAct)** obtuvo la mejor puntuación (48/50) por combinar razonamiento estratégico, métricas cuantitativas y roadmap accionable.

---

## Análisis Detallado por Prompt

### 1️⃣ PROMPT 1: Simple y Directo (Zero-shot Baseline)

**Archivo:** `prompt-1-simple.md`  
**Archivo de resultado:** `resultado-prompt-1-simple.md`

#### Características
- ✅ Genera 10 User Stories completas y bien estructuradas
- ✅ Cubre todas las áreas solicitadas (gestión candidatos, posiciones, colaboración, voz, automatizaciones)
- ✅ Criterios de aceptación específicos y medibles (7 criterios promedio)
- ✅ Notas técnicas detalladas con tecnologías concretas (spaCy, PyMuPDF, Celery, pgvector)

#### Puntos fuertes
- Muy práctico y directo al grano
- User Stories listas para implementar
- Detalles técnicos precisos (tiempos, precisión, costes)
- Fácil de iterar y modificar

#### Puntos débiles
- No hay priorización explícita justificada
- No hay análisis de valor vs complejidad
- Falta contexto estratégico del negocio
- No explica el "por qué" de cada feature

#### Evaluación

| Criterio | Puntuación | Justificación |
|----------|------------|---------------|
| Completitud | 8/10 | Todas las US están completas pero sin análisis previo |
| Priorización | 4/10 | Solo dice "Alta/Media/Baja" sin justificación |
| Detalle Técnico | 9/10 | Notas técnicas muy específicas |
| Implementabilidad | 9/10 | Listas para convertir en tickets directamente |
| Análisis Estratégico | 3/10 | No hay contexto de negocio o competencia |
| **TOTAL** | **33/50** | Bueno para ejecución, débil en estrategia |

#### Cuándo usar este prompt
- ✅ Cuando ya tienes la priorización definida por otro medio
- ✅ Cuando necesitas generar US rápidamente sin análisis profundo
- ✅ Cuando el equipo es pequeño y no necesita roadmap complejo
- ❌ Cuando necesitas presentar a stakeholders o inversores

---

### 2️⃣ PROMPT 2: Few-Shot (Con Ejemplo de Referencia)

**Archivo:** `prompt-2-few-shot.md`  
**Archivo de resultado:** `resultado-prompt-2-few-shot.md`

#### Características
- ✅ Incluye 11 User Stories (más que las 10 solicitadas)
- ✅ Sigue plantilla consistente del ejemplo proporcionado
- ✅ Criterios de aceptación técnicos y específicos (4-6 por US)
- ✅ Notas técnicas muy detalladas con arquitectura completa

#### Puntos fuertes
- **Máxima consistencia:** Todas las US siguen exactamente el mismo formato
- Detalles técnicos exhaustivos (endpoints, tablas BD, APIs)
- User Stories adicionales valiosas (US-011 sobre automatización de emails)
- Perfecto cuando necesitas formato muy específico

#### Puntos débiles
- Similar al Prompt 1, sin priorización estratégica
- Algunas User Stories son variantes de las del Prompt 1
- Falta análisis de dependencias entre US
- El ejemplo puede "anclar" el output del modelo

#### Evaluación

| Criterio | Puntuación | Justificación |
|----------|------------|---------------|
| Completitud | 9/10 | 11 US completas con formato uniforme |
| Priorización | 4/10 | Sin justificación estratégica de prioridades |
| Detalle Técnico | 9/10 | Arquitectura detallada (endpoints, BD, APIs) |
| Implementabilidad | 9/10 | Formato consistente facilita implementación |
| Análisis Estratégico | 3/10 | No hay contexto de negocio |
| **TOTAL** | **34/50** | Mejor consistencia que Prompt 1, pero sin estrategia |

#### Cuándo usar este prompt
- ✅ Cuando necesitas formato muy específico y uniforme
- ✅ Cuando tienes una plantilla de US que el equipo ya usa
- ✅ Cuando quieres asegurar consistencia entre todas las US
- ❌ Cuando necesitas priorización o análisis estratégico

---

### 3️⃣ PROMPT 3: Chain-of-Thought (Razonamiento Paso a Paso)

**Archivo:** `prompt-3-chain-of-thought.md`  
**Archivo de resultado:** `resultado-prompt-3-chain-of-thought.md`

#### Características
- ✅✅ **ANÁLISIS PREVIO COMPLETO** (3 funcionalidades críticas, flujos prioritarios, valor por rol)
- ✅✅ **PRIORIZACIÓN MoSCoW** estructurada (Must/Should/Could/Won't Have)
- ✅ 10 User Stories con prioridad justificada según MoSCoW
- ✅✅ **BACKLOG ORDENADO** por valor, riesgo técnico y dependencias
- ✅✅ **TABLA COMPARATIVA** final con ordenamiento de implementación

#### Puntos fuertes
- **Razonamiento visible:** Explica el "por qué" de cada decisión
- **Estrategia clara:** Define qué es MVP y qué es post-MVP
- **Dependencias explícitas:** Identifica qué US dependen de otras
- **Sprints organizados:** Agrupa US en Sprint 1, 2 y 3 con justificación
- Framework MoSCoW es ampliamente reconocido en la industria

#### Puntos débiles
- Más largo que los otros (puede abrumar a equipos pequeños)
- Análisis profundo puede ralentizar iteración rápida
- Menos cuantitativo que Prompt 5 (no hay métricas numéricas de valor)

#### Evaluación

| Criterio | Puntuación | Justificación |
|----------|------------|---------------|
| Completitud | 9/10 | US completas + análisis previo estructurado |
| Priorización | 10/10 | MoSCoW framework aplicado correctamente |
| Detalle Técnico | 8/10 | Buen detalle, aunque menos que Prompt 1-2 |
| Implementabilidad | 8/10 | Backlog ordenado facilita planificación |
| Análisis Estratégico | 10/10 | Análisis de funcionalidades críticas y flujos |
| **TOTAL** | **45/50** ⭐ | Excelente balance estrategia + ejecución |

#### Cuándo usar este prompt
- ✅ Cuando necesitas justificar priorización a stakeholders
- ✅ Cuando trabajas con metodologías ágiles (MoSCoW, Scrum)
- ✅ Cuando el equipo necesita entender el "por qué" de cada decisión
- ✅ Cuando tienes tiempo para análisis profundo antes de implementar

---

### 4️⃣ PROMPT 4: JSON Estructurado (Structured Output)

**Archivo:** `prompt-4-structured-json.md`  
**Archivo de resultado:** `resultado-prompt-4-json.md`

#### Características
- ✅✅ **FORMATO JSON VÁLIDO Y PARSEABLE** (listo para integración con herramientas)
- ✅ 10 User Stories con campos estructurados uniformes
- ✅ Campos adicionales valiosos: `priority_justification`, `estimated_effort`, `dependencies`
- ✅✅ **ESTIMACIONES DE ESFUERZO** (S/M/L/XL) por cada US

#### Puntos fuertes
- **Máxima estructuración:** Perfecto para importar a Jira, Linear, GitHub Projects
- **Campos estándar:** Prioridad justificada, esfuerzo estimado, dependencias
- **Notas técnicas descompuestas:** Backend, Frontend, Database, APIs externas separadas
- **Automatizable:** Fácil de parsear y procesar con scripts Python/Node.js
- **Integración directa:** Puedes hacer `POST` del JSON a APIs de PM tools

#### Puntos débiles
- JSON puede ser menos legible para humanos que Markdown
- Falta análisis estratégico (solo lista de US sin contexto previo)
- Requiere herramientas para visualizar de forma amigable
- No incluye roadmap o sprints organizados

#### Evaluación

| Criterio | Puntuación | Justificación |
|----------|------------|---------------|
| Completitud | 9/10 | Todas las US con campos estructurados |
| Priorización | 7/10 | Incluye `priority_justification` pero sin framework |
| Detalle Técnico | 9/10 | Notas técnicas descompuestas por capa |
| Implementabilidad | 10/10 | JSON directo a herramientas de PM |
| Análisis Estratégico | 5/10 | Justificaciones individuales pero sin análisis macro |
| **TOTAL** | **40/50** | Mejor para automatización, no para planificación |

#### Cuándo usar este prompt
- ✅ Cuando necesitas importar US a Jira, Linear, GitHub Projects automáticamente
- ✅ Cuando tienes pipeline de CI/CD que procesa User Stories
- ✅ Cuando trabajas con herramientas de gestión que aceptan JSON
- ✅ Cuando quieres procesar US con scripts (ej: generar tickets automáticamente)
- ❌ Cuando necesitas presentar a humanos (prefiere Markdown)

---

### 5️⃣ PROMPT 5: ReAct (Role + Reasoning + Acting)

**Archivo:** `prompt-5-react.md`  
**Archivo de resultado:** `resultado-prompt-5-react.md`

#### Características
- ✅✅✅ **THOUGHT (RAZONAMIENTO)**: Análisis profundo de features críticas, diferenciación vs competencia, flujos con fricción
- ✅✅ **ACTION**: 10 User Stories con **valor de negocio (X/10)**, **complejidad técnica (X/10)** y **ratio valor/complejidad**
- ✅✅✅ **OBSERVATION**: Tabla resumen comparando todas las US por métricas cuantitativas
- ✅✅✅ **BACKLOG FINAL**: Ordenado por ratio V/C descendente con sprints organizados
- ✅✅ **RECOMENDACIONES ESTRATÉGICAS**: MVP mínimo, post-MVP, consideraciones técnicas

#### Puntos fuertes
- **Máximo nivel de análisis:** Combina razonamiento + datos cuantitativos + recomendaciones
- **Métricas objetivas:** Valor de negocio, complejidad técnica y ratio calculado
- **Visión estratégica:** Diferenciación vs competencia (Greenhouse, Lever), features críticas día 1
- **Decisiones justificadas:** Cada priorización tiene fundamento lógico Y cuantitativo
- **Roadmap claro:** Sprint 1 (MVP Core), Sprint 2 (Valor Medio), Sprint 3 (Post-MVP)
- **Storytelling:** Perfecto para presentar a inversores o C-level

#### Puntos débiles
- El más largo y complejo de todos (puede ser excesivo para equipos muy pequeños)
- Requiere mayor tiempo de generación (~2-3 min vs ~30s de Prompt 1)
- Puede generar "analysis paralysis" si el equipo no está acostumbrado a análisis profundo

#### Evaluación

| Criterio | Puntuación | Justificación |
|----------|------------|---------------|
| Completitud | 10/10 | US + análisis + métricas + roadmap + recomendaciones |
| Priorización | 10/10 | Ratio V/C cuantitativo permite comparación objetiva |
| Detalle Técnico | 9/10 | Detalles técnicos sólidos (no tan exhaustivos como Prompt 2) |
| Implementabilidad | 9/10 | Backlog priorizado con dependencias y sprints claros |
| Análisis Estratégico | 10/10 | Análisis de competencia, features día 1, roadmap |
| **TOTAL** | **48/50** ⭐⭐⭐ | Mejor para Product Management profesional |

#### Cuándo usar este prompt
- ✅✅ Cuando necesitas presentar backlog a stakeholders, inversores o C-level
- ✅✅ Cuando trabajas como Product Manager en startup y necesitas justificar roadmap
- ✅ Cuando tienes que tomar decisiones de qué construir primero con datos objetivos
- ✅ Cuando el equipo necesita entender contexto estratégico, no solo features
- ✅ Cuando tienes múltiples opciones y necesitas priorizar con métricas
- ❌ Cuando necesitas velocidad pura (usa Prompt 1 o 2 en ese caso)

---

## Tabla Comparativa Final

| Prompt | Completitud | Priorización | Detalle Técnico | Implementabilidad | Análisis Estratégico | **TOTAL** | Ranking |
|--------|-------------|--------------|-----------------|-------------------|----------------------|-----------|---------|
| **1. Simple** | 8/10 | 4/10 | 9/10 | 9/10 | 3/10 | **33/50** | 🥉 5º |
| **2. Few-Shot** | 9/10 | 4/10 | 9/10 | 9/10 | 3/10 | **34/50** | 4º |
| **3. Chain-of-Thought** | 9/10 | 10/10 | 8/10 | 8/10 | 10/10 | **45/50** ⭐ | 🥈 2º |
| **4. JSON Estructurado** | 9/10 | 7/10 | 9/10 | 10/10 | 5/10 | **40/50** | 3º |
| **5. ReAct** | 10/10 | 10/10 | 9/10 | 9/10 | 10/10 | **48/50** ⭐⭐⭐ | 🥇 1º |

---

## Recomendación Final

### 🏆 GANADOR: PROMPT 5 (ReAct - Role + Reasoning + Acting)

#### Por qué es el mejor:

1. **Análisis más completo**: Incluye razonamiento estratégico (THOUGHT), métricas cuantitativas (valor/complejidad), y recomendaciones accionables.

2. **Priorización objetiva**: Ratio valor/complejidad permite comparar User Stories con datos en vez de intuición. Un PM puede defender sus decisiones con números.

3. **Visión de producto**: Explica diferenciación vs competencia (Greenhouse/Lever), features críticas día 1, y minimización de riesgo técnico.

4. **Roadmap claro**: Sprint 1 (MVP Core), Sprint 2-3 (Post-MVP) con justificación de por qué cada feature va en cada sprint.

5. **Mejor para presentar**: Si tienes que presentar el backlog a stakeholders o inversores, este formato tiene el storytelling más convincente.

#### Casos de uso ideales:

- **Startups buscando inversión**: Necesitas mostrar que tienes roadmap claro y priorización basada en datos.
- **Product Managers presentando a C-level**: Los ejecutivos quieren ver análisis estratégico, no solo lista de features.
- **Equipos grandes con múltiples stakeholders**: Diferentes roles (engineering, design, sales) necesitan entender el "por qué" de cada decisión.

---

## Lecciones Aprendidas

### 1. Context is King 👑
Cuanto más contexto estratégico proporcionas (competencia, target, dolores del usuario), mejor es el output. Los prompts 3 y 5 incluyen contexto extenso y generan análisis significativamente mejores.

### 2. Frameworks explícitos funcionan 📊
Mencionar frameworks específicos (MoSCoW, ratio valor/complejidad, ReAct) guía al modelo mucho mejor que instrucciones vagas como "prioriza las User Stories".

**Ejemplo:**
- ❌ Vago: "Prioriza las User Stories"
- ✅ Explícito: "Aplica el framework MoSCoW para clasificar en Must/Should/Could/Won't Have"

### 3. Multi-paso > Single-shot 🎯
Prompts que fuerzan razonamiento paso a paso (Chain-of-Thought, ReAct) generan output significativamente mejor que prompts directos.

**Diferencia clave:**
- Prompt 1 (single-shot): "Genera 10 User Stories" → Lista de features
- Prompt 5 (multi-paso): "THOUGHT → ACTION → OBSERVATION → BACKLOG" → Roadmap estratégico

### 4. Role prompting añade perspectiva 🎭
Hacer que el modelo actúe como "Lead PM con 10 años de experiencia en HR Tech (ex-Greenhouse, Lever)" cambia dramáticamente la profundidad del análisis.

El modelo adopta perspectiva de alguien que:
- Conoce la competencia
- Entiende dolores reales de usuarios
- Ha visto features fallar/triunfar
- Sabe priorizar con criterio de negocio

### 5. Métricas cuantitativas > cualitativas 📈
Pedir valor/complejidad en escala 1-10 permite comparar User Stories objetivamente vs descripciones vagas como "alta prioridad".

**Ejemplo:**
- ❌ Cualitativo: "Esta US tiene alta prioridad porque es importante"
- ✅ Cuantitativo: "Valor 9/10, Complejidad 5/10 → Ratio 1.8 (top 3 del backlog)"

### 6. Structured output para automatización 🤖
Si necesitas integrar con herramientas (Jira, Linear), JSON estructurado (Prompt 4) es óptimo. Pero para planificación humana, Markdown con análisis (Prompt 5) gana.

---

## Matriz de Decisión: ¿Qué Prompt Usar?

| Situación | Prompt Recomendado | Por qué |
|-----------|-------------------|---------|
| **Presentar a inversores** | 5 (ReAct) ⭐⭐⭐ | Storytelling estratégico con métricas |
| **Presentar a C-level** | 5 (ReAct) ⭐⭐⭐ | Análisis de competencia y ROI claro |
| **Planning de Sprint con equipo** | 3 (Chain-of-Thought) ⭐ | MoSCoW framework + dependencias |
| **Importar a Jira/Linear** | 4 (JSON) | Formato parseable automáticamente |
| **Generar US rápido (ya tienes roadmap)** | 1 (Simple) | Velocidad + detalles técnicos |
| **Asegurar formato consistente** | 2 (Few-Shot) | Todas las US siguen misma plantilla |
| **Primera vez creando backlog** | 5 (ReAct) ⭐⭐⭐ | Te enseña a pensar estratégicamente |

---

## Recomendaciones para Futuros Ejercicios

### Para ejercicios de Product Management:
- **Primera opción:** ReAct (Prompt 5)
- **Segunda opción:** Chain-of-Thought (Prompt 3)
- **Nunca uses:** Simple (Prompt 1) sin contexto estratégico

### Para ejercicios de integración técnica:
- **Primera opción:** JSON Estructurado (Prompt 4)
- **Segunda opción:** Few-Shot (Prompt 2) con ejemplo técnico

### Para generación rápida de prototipos:
- **Primera opción:** Few-Shot (Prompt 2)
- **Segunda opción:** Simple (Prompt 1)

---

## Métricas de Éxito del Experimento

| Métrica | Resultado |
|---------|-----------|
| **Prompts probados** | 5 técnicas diferentes |
| **User Stories generadas** | 50+ (10 por prompt) |
| **Tiempo total de experimentación** | ~90 minutos |
| **Mejor prompt identificado** | ReAct (48/50 puntos) |
| **Diferencia vs peor prompt** | +15 puntos (45% mejor) |
| **Técnicas de PE aplicadas** | Zero-shot, Few-shot, CoT, Structured Output, ReAct |

---

## Referencias

- **Prompt Engineering Guide (Español)**: https://www.promptingguide.ai/es
- **Técnicas aplicadas**: 
  - Zero-shot Prompting
  - Few-shot Learning
  - Chain-of-Thought (CoT)
  - Structured Output
  - ReAct (Reason + Act)
  - Role Prompting

- **Frameworks mencionados**:
  - MoSCoW (Must/Should/Could/Won't Have)
  - Ratio Valor/Complejidad
  - User Story Mapping

---

**Conclusión:** La experimentación con diferentes técnicas de prompt engineering demostró que invertir tiempo en estructurar el prompt (especialmente con ReAct) genera output 45% mejor en términos de utilidad para Product Management real.

---

**Fin del documento**
