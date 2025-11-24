Técnica: Structured output + validación estricta

text
Actúa como Product Manager experto en ATS y generación de User Stories.

# CONTEXTO:
Sistema LTI: ATS con IA para startups tech. Funcionalidades clave:
- Matching semántico de candidatos (IA)
- Colaboración en tiempo real
- Asistente de voz conversacional
- Automatizaciones de workflow

Stack: FastAPI, React+TS, Supabase, OpenAI, Deepgram, ElevenLabs.

# TAREA:
Genera 10 User Stories para LTI en formato JSON válido siguiendo este esquema:

{
"user_stories": [
{
"id": "US-001",
"title": "Título corto y descriptivo",
"user_type": "Reclutador | Hiring Manager | Candidato | Admin",
"user_story": "Como [user_type], quiero [acción] para [beneficio]",
"priority": "Alta | Media | Baja",
"priority_justification": "Explicación de por qué esta prioridad",
"acceptance_criteria": [
"Criterio específico y medible 1",
"Criterio específico y medible 2",
"Criterio específico y medible 3",
"Criterio específico y medible 4"
],
"technical_notes": {
"backend": "Endpoints, servicios, lógica",
"frontend": "Componentes React, state management",
"database": "Tablas, queries",
"external_apis": "APIs externas necesarias"
},
"estimated_effort": "S | M | L | XL",
"dependencies": ["US-XXX", "US-YYY"]
}
]
}

text

Distribuye las User Stories:
- 3 de Gestión de Candidatos (parsing, matching, scoring)
- 2 de Gestión de Posiciones
- 2 de Colaboración en Tiempo Real
- 2 de Asistente de Voz
- 1 de Automatizaciones

Asegúrate de que el JSON sea válido y parseable.