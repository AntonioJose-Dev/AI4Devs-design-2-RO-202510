Técnica: Few-shot learning + contexto extenso

text
Actúa como Product Manager senior especializado en HR Tech y ATS.

# CONTEXTO DEL SISTEMA LTI

LTI (Leading Talent Intelligence) es un ATS de nueva generación para startups tech (20-500 empleados) que combina:
- Matching semántico de candidatos con IA (OpenAI embeddings + pgvector)
- Colaboración en tiempo real (WebSocket)
- Asistente de voz conversacional (OpenAI Realtime API)
- Automatizaciones inteligentes (LangChain)

Stack técnico: FastAPI, React+TS, Supabase (PostgreSQL+pgvector), OpenAI, Deepgram, ElevenLabs.

## FUNCIONALIDADES PRINCIPALES:

1. **Gestión de Candidatos:**
   - Parsing automático de CVs con IA (extracción de skills, experiencia)
   - Matching semántico candidato-posición (similaridad coseno)
   - Scoring automatizado
   - Historial completo de interacciones

2. **Gestión de Posiciones:**
   - Creación de job descriptions
   - Generación automática de embeddings
   - Publicación multi-canal
   - Pipeline configurable

3. **Colaboración en Tiempo Real:**
   - Chat interno entre reclutadores y hiring managers
   - Feedback colaborativo sobre candidatos
   - Notificaciones push en tiempo real

4. **Asistente de Voz IA:**
   - Búsqueda de candidatos por voz
   - Dictado de notas en entrevistas
   - Resumen automático de conversaciones

5. **Automatizaciones:**
   - Workflows configurables
   - Emails personalizados con IA
   - Scheduling automático de entrevistas

## EJEMPLO DE USER STORY (referencia):

**ID:** US-001
**Título:** Parsing automático de CV con IA
**Como:** Reclutador
**Quiero:** Subir un CV en PDF y que el sistema extraiga automáticamente nombre, email, teléfono, skills, experiencia laboral y educación
**Para:** Ahorrar tiempo de carga manual de datos y evitar errores humanos
**Prioridad:** Alta
**Criterios de aceptación:**
- [ ] El sistema acepta archivos PDF, DOCX y TXT de hasta 5MB
- [ ] Extrae correctamente nombre completo, email y teléfono con 95%+ precisión
- [ ] Identifica y estructura experiencia laboral (empresa, cargo, fechas, descripción)
- [ ] Extrae lista de skills técnicas mencionadas
- [ ] Procesa el CV en menos de 10 segundos
- [ ] Permite revisión y edición manual de los datos extraídos
**Notas técnicas:**
- Usar OpenAI GPT-4 con prompt estructurado para extracción
- Almacenar texto completo en `candidates.resume_text`
- Generar embedding del CV con `text-embedding-3-small`
- Implementar validación de campos (formato email, teléfono)

---

# TAREA:

Genera 10 User Stories COMPLETAS para LTI siguiendo EXACTAMENTE la plantilla del ejemplo anterior.

Distribuye las User Stories así:
- 3 User Stories de Gestión de Candidatos
- 2 User Stories de Gestión de Posiciones
- 2 User Stories de Colaboración en Tiempo Real
- 2 User Stories de Asistente de Voz
- 1 User Story de Automatizaciones

Cada User Story debe tener:
- ID único secuencial (US-001, US-002...)
- Título descriptivo y conciso
- Formato "Como... Quiero... Para..."
- Prioridad justificada (Alta si es core MVP, Media si mejora UX, Baja si es nice-to-have)
- Mínimo 4 criterios de aceptación específicos y medibles
- Notas técnicas con referencias al stack (APIs, tablas de BD, componentes)

Genera las 10 User Stories ahora.