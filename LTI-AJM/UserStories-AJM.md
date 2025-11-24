# User Stories - Sistema LTI
**Autor:** Antonio José Marín (AJM)  
**Fecha:** 24 de noviembre de 2025  
**Versión:** 1.0  
**Metodología:** Prompt Engineering (ReAct: Reasoning + Acting)

---

## Índice

1. [Análisis Estratégico (THOUGHT)](#análisis-estratégico-thought)
2. [User Stories (ACTION)](#user-stories-action)
3. [Métricas y Observaciones (OBSERVATION)](#métricas-y-observaciones-observation)
4. [Backlog Priorizado (BACKLOG FINAL)](#backlog-priorizado-backlog-final)
5. [Tickets de Trabajo - US-001](#tickets-de-trabajo---us-001)
6. [Estimaciones de Esfuerzo](#estimaciones-de-esfuerzo)
7. [Experimentación con Prompts](#experimentación-con-prompts)

---

## Análisis Estratégico (THOUGHT)

### Features Críticas para Adopción Día 1

Tras analizar el sistema LTI documentado y compararlo con ATS tradicionales (Greenhouse, Lever, Workable), identifico **3 funcionalidades críticas** que un reclutador necesita para adoptar LTI desde el primer día:

1. **Matching Semántico con IA**: La capacidad de encontrar candidatos relevantes sin depender de palabras clave exactas es el diferenciador #1 vs competencia. Los reclutadores pierden candidatos excelentes porque su CV dice "Python" y la búsqueda dice "py".

2. **Parsing Automático de CVs**: Eliminar la carga manual de datos (que toma 5-10 minutos por candidato) es crítico para justificar el cambio de herramienta. Este es el "wow moment" del producto.

3. **Colaboración en Tiempo Real**: Los ATS tradicionales tienen flujos de feedback asincrónicos y lentos. Un reclutador que puede chatear en tiempo real con el hiring manager sobre un candidato toma decisiones 3x más rápido.

### Diferenciación vs Competencia

**LTI se diferencia en:**
- **IA no como feature secundaria sino como core**: Greenhouse tiene "AI scoring" como add-on caro. En LTI, el matching semántico es nativo y accesible.
- **Colaboración estilo Slack/Notion**: Los ATS tradicionales tienen "comentarios" estáticos. LTI tiene chat en tiempo real con WebSocket.
- **Asistente de voz conversacional**: Ningún ATS tradicional tiene búsqueda/dictado por voz. Es un differentiator único.

### Flujos con Mayor Fricción en ATS Tradicionales

1. **Búsqueda de candidatos**: En Greenhouse/Lever buscas por keywords booleanas (`Python AND Django NOT Junior`). Frustante e impreciso.
2. **Carga de candidatos**: Copiar-pegar datos del CV a formularios. Tedioso y propenso a errores.
3. **Feedback entre reclutador y manager**: Emails/comentarios asincrónicos → decisiones lentas.
4. **Evaluación de candidatos**: Scorecards con checkboxes genéricos, sin contexto IA.

---

## User Stories (ACTION)

### US-001: Parsing Automático de CVs con IA
**Como:** Reclutador  
**Quiero:** Subir un CV en PDF/DOCX y que el sistema extraiga automáticamente todos los datos estructurados (nombre, email, teléfono, experiencia laboral, educación, skills)  
**Para:** Ahorrar 5-10 minutos de carga manual por candidato y evitar errores de transcripción  
**Prioridad:** Alta (Must Have - MVP Core)  
**Valor de negocio:** 10/10  
**Complejidad técnica:** 7/10  
**Ratio valor/complejidad:** 1.43

**Criterios de aceptación:**
- [ ] El sistema acepta archivos PDF, DOCX y TXT de hasta 5MB
- [ ] Extrae correctamente nombre completo, email, teléfono y ubicación con precisión >95%
- [ ] Identifica y estructura experiencia laboral con formato: empresa, cargo, fechas (mes/año), descripción
- [ ] Extrae skills técnicas y las normaliza (ej: "React.js" → "React", "Python 3.11" → "Python")
- [ ] Extrae información educativa: universidad, título, año de graduación
- [ ] Procesa el CV en <10 segundos y muestra formulario pre-rellenado para validación
- [ ] Maneja errores gracefully (CV corrupto, imagen escaneada) con mensaje claro al usuario
- [ ] Genera embedding del CV completo (vector 1536 dimensiones) para matching posterior

**Notas técnicas:**
- **Backend**: Endpoint `POST /api/candidates/upload-cv` que recibe multipart/form-data
- **Parser**: OpenAI GPT-4 con prompt estructurado de extracción (función `parse_cv_with_gpt4()`)
- **Embeddings**: `text-embedding-3-small` de OpenAI sobre texto completo del CV
- **Database**: Tabla `candidates` con campos `resume_text` (TEXT), `resume_embedding` (VECTOR(1536))
- **Storage**: CVs originales en Supabase Storage bucket `candidate-cvs/`
- **Queue**: Celery task async para procesamiento pesado (no bloquear response HTTP)

**Dependencias:** Ninguna

---

### US-002: Matching Semántico Candidato-Posición
**Como:** Reclutador  
**Quiero:** Que el sistema me sugiera automáticamente los top 10 candidatos más relevantes para una posición usando IA semántica (no keywords)  
**Para:** Encontrar candidatos excelentes que descartaría con búsqueda tradicional de palabras clave  
**Prioridad:** Alta (Must Have - MVP Core)  
**Valor de negocio:** 10/10  
**Complejidad técnica:** 8/10  
**Ratio valor/complejidad:** 1.25

**Criterios de aceptación:**
- [ ] Al crear una posición, el sistema genera automáticamente un embedding de la job description
- [ ] El sistema calcula similaridad coseno entre embedding de posición y embeddings de todos los candidatos
- [ ] Devuelve top 10 candidatos rankeados por score de matching (0-100%)
- [ ] El matching considera: skills, experiencia, educación y descripción de responsabilidades
- [ ] Muestra explicación visual de por qué cada candidato es relevante (skills coincidentes, experiencia similar)
- [ ] El matching se ejecuta en <3 segundos para bases de datos de hasta 10,000 candidatos
- [ ] El sistema aprende de feedback (cuando un candidato rankeado bajo es contratado, ajusta pesos)

**Notas técnicas:**
- **Backend**: Endpoint `GET /api/positions/{id}/match-candidates?limit=10`
- **Database**: Query pgvector con operador de distancia coseno: `SELECT *, 1 - (resume_embedding <=> position_embedding) AS match_score FROM candidates ORDER BY match_score DESC LIMIT 10`
- **Index**: Crear índice HNSW en `resume_embedding` para performance: `CREATE INDEX ON candidates USING hnsw (resume_embedding vector_cosine_ops)`
- **Cache**: Redis para cachear resultados de matching por 1 hora (invalidar cuando se actualiza posición o candidato)
- **Frontend**: Componente `<CandidateMatchList>` con progress bar de match score y chips de skills coincidentes

**Dependencias:** US-001 (requiere embeddings de CVs generados)

---

### US-003: Colaboración en Tiempo Real sobre Candidatos
**Como:** Reclutador y Hiring Manager  
**Quiero:** Poder chatear en tiempo real sobre un candidato específico mientras revisamos su perfil  
**Para:** Tomar decisiones de avanzar/descartar 3x más rápido que con emails/comentarios asincrónicos  
**Prioridad:** Alta (Must Have - MVP Core)  
**Valor de negocio:** 9/10  
**Complejidad técnica:** 7/10  
**Ratio valor/complejidad:** 1.29

**Criterios de aceptación:**
- [ ] Cada candidato tiene un chat privado visible solo para reclutadores y managers asignados a esa posición
- [ ] Los mensajes aparecen en tiempo real (<500ms latencia) para todos los usuarios conectados
- [ ] Se muestra indicador "X está escribiendo..." cuando alguien está tipeando
- [ ] Los mensajes se persisten en BD y son visibles al recargar la página
- [ ] Se pueden mencionar usuarios con @ (ej: "@Juan ¿qué opinas de este candidato?") y reciben notificación push
- [ ] Se puede adjuntar archivos (notas de entrevista, scorecards) en el chat
- [ ] Historial de chat se exporta a PDF junto con el perfil del candidato

**Notas técnicas:**
- **Backend**: WebSocket endpoint `wss://api.lti.com/ws/candidates/{candidate_id}/chat`
- **Protocol**: FastAPI WebSockets con autenticación JWT en handshake
- **Database**: Tabla `chat_messages` con campos: `id`, `candidate_id`, `author_id`, `message`, `created_at`, `attachments`
- **Frontend**: Componente `<RealtimeChat>` con React Query mutations y WebSocket hook
- **Broadcast**: Redis Pub/Sub para broadcast de mensajes entre múltiples instancias de backend
- **Notifications**: Service Worker para push notifications cuando usuario mencionado con @

**Dependencias:** Ninguna

---

### US-004: Pipeline Kanban Visual
**Como:** Reclutador  
**Quiero:** Ver y arrastrar candidatos entre etapas del proceso (Applied → Screening → Interview → Offer → Hired)  
**Para:** Tener visibilidad clara del estado de todos los candidatos de un vistazo  
**Prioridad:** Alta (Must Have - MVP Core)  
**Valor de negocio:** 8/10  
**Complejidad técnica:** 5/10  
**Ratio valor/complejidad:** 1.60

**Criterios de aceptación:**
- [ ] Cada posición tiene un tablero Kanban con columnas configurables (por defecto: Applied, Screening, Interview, Offer, Hired, Rejected)
- [ ] Los candidatos se muestran como tarjetas con foto, nombre, score de matching y última actividad
- [ ] Drag & drop fluido para mover candidatos entre columnas
- [ ] Al mover un candidato, se actualiza automáticamente su status en BD y notifica al candidato (email)
- [ ] Las columnas muestran contador de candidatos (ej: "Interview (3)")
- [ ] Se pueden aplicar filtros (por skill, experiencia, ubicación) sin salir del tablero
- [ ] El tablero carga <2 segundos incluso con 100+ candidatos en el pipeline

**Notas técnicas:**
- **Frontend**: Librería `@dnd-kit/core` para drag & drop con React
- **Backend**: Endpoint `PATCH /api/applications/{id}/status` con validación de transiciones válidas
- **Database**: Tabla `applications` con campo `status` (ENUM) y `updated_at` (timestamp con trigger)
- **Optimistic Updates**: Frontend actualiza UI inmediatamente, rollback si el PATCH falla
- **Email**: Queue de Celery para envío asincrónico de emails de cambio de estado

**Dependencias:** US-002 (usa score de matching en las tarjetas)

---

### US-005: Asistente de Voz para Búsqueda de Candidatos
**Como:** Reclutador  
**Quiero:** Buscar candidatos diciendo "Busca desarrolladores de Python con 5 años de experiencia en Madrid"  
**Para:** Realizar búsquedas complejas 10x más rápido que escribiendo filtros booleanos  
**Prioridad:** Media (Should Have - Post-MVP Sprint 2)  
**Valor de negocio:** 7/10  
**Complejidad técnica:** 9/10  
**Ratio valor/complejidad:** 0.78

**Criterios de aceptación:**
- [ ] Botón de micrófono en barra de búsqueda que activa captura de voz
- [ ] El sistema transcribe el audio a texto en <2 segundos (Deepgram STT)
- [ ] Un LLM (GPT-4) extrae intención de búsqueda y parámetros (skills, experiencia, ubicación)
- [ ] El sistema ejecuta búsqueda semántica basada en los parámetros extraídos
- [ ] El asistente responde por voz: "He encontrado 5 candidatos de Python con 5+ años en Madrid"
- [ ] La respuesta de voz se reproduce automáticamente (ElevenLabs TTS)
- [ ] Los resultados se muestran visualmente en la UI simultáneamente

**Notas técnicas:**
- **Backend**: Endpoint `POST /api/voice/search` que recibe audio en base64
- **STT**: Deepgram API con modelo `nova-2` (latencia <2s)
- **Intent Extraction**: GPT-4 con function calling para extraer parámetros estructurados
- **TTS**: ElevenLabs API con voz personalizada (entrenada con voz de marca LTI)
- **Frontend**: Web Speech API para captura de audio del micrófono
- **Observability**: Log de todas las queries de voz en Braintrust para análisis de intención

**Dependencias:** US-002 (usa matching semántico para búsqueda)

---

### US-006: Generación de Job Descriptions con IA
**Como:** Reclutador  
**Quiero:** Generar una job description completa escribiendo solo el título del puesto y requisitos clave  
**Para:** Crear ofertas atractivas en 2 minutos en vez de 30 minutos  
**Prioridad:** Media (Should Have - Post-MVP Sprint 2)  
**Valor de negocio:** 7/10  
**Complejidad técnica:** 5/10  
**Ratio valor/complejidad:** 1.40

**Criterios de aceptación:**
- [ ] Formulario de creación de posición tiene botón "Generar con IA"
- [ ] Usuario introduce: título del puesto, 3-5 requisitos clave, rango salarial (opcional)
- [ ] El sistema genera job description completa con: resumen atractivo, responsabilidades (5-7), requisitos técnicos, soft skills deseables, beneficios
- [ ] El tono de la descripción es profesional pero amigable (marca LTI)
- [ ] Usuario puede editar la descripción generada antes de publicar
- [ ] El sistema genera 3 variantes de descripción y el usuario elige la mejor

**Notas técnicas:**
- **Backend**: Endpoint `POST /api/positions/generate-description`
- **LLM**: GPT-4 con prompt engineering específico para job descriptions (ejemplos de alta calidad en few-shot)
- **Prompt**: Incluir contexto de la empresa (obtenido de `companies` table) para personalización
- **Frontend**: Editor rico (TipTap) para edición de la descripción generada
- **Cost**: ~$0.05 por generación (GPT-4), aceptable para feature premium

**Dependencias:** Ninguna

---

### US-007: Evaluaciones Estructuradas de Candidatos
**Como:** Hiring Manager  
**Quiero:** Rellenar un scorecard estructurado después de cada entrevista con criterios predefinidos  
**Para:** Tomar decisiones de contratación basadas en datos objetivos, no intuiciones  
**Prioridad:** Media (Should Have - Post-MVP Sprint 2)  
**Valor de negocio:** 8/10  
**Complejidad técnica:** 6/10  
**Ratio valor/complejidad:** 1.33

**Criterios de aceptación:**
- [ ] Cada posición tiene un template de scorecard configurable (ej: Skills técnicas 40%, Fit cultural 30%, Comunicación 30%)
- [ ] Después de cada entrevista, el entrevistador recibe notificación para rellenar scorecard
- [ ] El scorecard tiene ratings (1-5 estrellas) por categoría + campo de comentarios
- [ ] El sistema calcula score total ponderado automáticamente
- [ ] Todos los scorecards de un candidato se agregan en una vista consolidada
- [ ] El sistema sugiere decisión (Hire/No Hire) basada en score agregado vs threshold de la posición

**Notas técnicas:**
- **Database**: Tablas `scorecard_templates` (configuración) y `scorecards` (evaluaciones completadas)
- **Backend**: Endpoints para CRUD de templates y scorecards
- **Calculations**: Función `calculate_weighted_score()` que suma ratings ponderados por categoría
- **Frontend**: Formulario dinámico que se genera desde el template (React Hook Form)
- **Notifications**: Email + push notification al entrevistador 1 hora después de la entrevista

**Dependencias:** Ninguna

---

### US-008: Scheduling Automático de Entrevistas con Calendario
**Como:** Reclutador  
**Quiero:** Que el sistema proponga automáticamente slots de entrevista considerando la disponibilidad del candidato y del entrevistador  
**Para:** Eliminar el back-and-forth de emails coordinando horarios (que toma 2-3 días)  
**Prioridad:** Media (Could Have - Post-MVP Sprint 3)  
**Valor de negocio:** 6/10  
**Complejidad técnica:** 8/10  
**Ratio valor/complejidad:** 0.75

**Criterios de aceptación:**
- [ ] Integración con Google Calendar para leer disponibilidad del entrevistador
- [ ] El reclutador envía link de self-scheduling al candidato
- [ ] El candidato ve slots disponibles en su zona horaria y elige uno
- [ ] Al confirmar, se crea evento de calendario automáticamente para ambos con link de videollamada (Google Meet)
- [ ] Se envían recordatorios automáticos 24h y 1h antes de la entrevista
- [ ] Si el entrevistador cancela, el candidato recibe opciones alternativas automáticamente

**Notas técnicas:**
- **Integration**: Google Calendar API con OAuth2 (scopes: `calendar.readonly`, `calendar.events`)
- **Backend**: Endpoints para leer slots disponibles y crear eventos
- **Logic**: Función `find_available_slots()` que intersecta disponibilidad candidato + entrevistador
- **Videocalls**: Google Meet links automáticos (generados por Calendar API)
- **Reminders**: Celery tasks programados para envío de recordatorios
- **Frontend**: Componente `<CalendarScheduler>` estilo Calendly

**Dependencias:** Ninguna

---

### US-009: Automatizaciones de Workflow (Reglas Tipo Zapier)
**Como:** Admin del sistema  
**Quiero:** Configurar reglas automáticas tipo "Si candidato pasa a etapa Interview → enviar email con instrucciones + crear evento en calendario"  
**Para:** Automatizar tareas repetitivas y asegurar procesos consistentes  
**Prioridad:** Baja (Could Have - Post-MVP Sprint 3)  
**Valor de negocio:** 7/10  
**Complejidad técnica:** 9/10  
**Ratio valor/complejidad:** 0.78

**Criterios de aceptación:**
- [ ] UI visual para crear reglas (trigger + acciones) sin código
- [ ] Triggers disponibles: cambio de etapa, score supera threshold, fecha límite alcanzada
- [ ] Acciones disponibles: enviar email, crear tarea, notificar usuario, actualizar campo, llamar webhook externo
- [ ] Las reglas se ejecutan en <5 segundos después del trigger
- [ ] Log de ejecuciones de reglas (éxito/fallo) visible en dashboard de admin
- [ ] Se pueden activar/desactivar reglas sin borrarlas

**Notas técnicas:**
- **Architecture**: Event-driven con Redis Streams (eventos de sistema publican a stream)
- **Rules Engine**: Librería `python-business-rules` para evaluar condiciones
- **Database**: Tabla `automation_rules` con campos: `trigger`, `conditions` (JSONB), `actions` (JSONB)
- **Worker**: Celery consumer que lee eventos de Redis Stream y ejecuta reglas matching
- **Frontend**: Visual rule builder similar a Zapier/Make.com (drag & drop de bloques)

**Dependencias:** Ninguna (pero US-004 y US-007 generan eventos útiles para triggers)

---

### US-010: Dictado de Notas en Entrevistas por Voz
**Como:** Hiring Manager  
**Quiero:** Dictar notas durante o después de una entrevista usando mi voz  
**Para:** Capturar observaciones sin perder contacto visual con el candidato o tipear después  
**Prioridad:** Baja (Could Have - Post-MVP Sprint 3)  
**Valor de negocio:** 5/10  
**Complejidad técnica:** 6/10  
**Ratio valor/complejidad:** 0.83

**Criterios de aceptación:**
- [ ] En la vista de candidato, hay botón "Dictar notas" que activa micrófono
- [ ] El sistema transcribe audio en tiempo real con latencia <2s (Deepgram streaming STT)
- [ ] El texto transcrito aparece en editor de notas y se guarda automáticamente
- [ ] El sistema identifica entidades clave (skills mencionadas, feedback positivo/negativo) y las resalta
- [ ] Se puede pausar/reanudar dictado sin perder contexto
- [ ] Funciona en Chrome, Safari y Firefox (Web Speech API + fallback a Deepgram)

**Notas técnicas:**
- **Backend**: WebSocket endpoint `wss://api.lti.com/ws/dictation` para streaming de audio
- **STT**: Deepgram Streaming API con modelo `nova-2` (latencia sub-2s)
- **NER**: SpaCy model `en_core_web_sm` para entity recognition (extraer skills, empresas mencionadas)
- **Frontend**: Web Speech API con fallback a Deepgram si browser no soporta
- **Storage**: Notas se guardan en `interviews.notes` (TEXT column) con autosave cada 5s

**Dependencias:** Ninguna

---

## Métricas y Observaciones (OBSERVATION)

| User Story | Valor Negocio | Complejidad Técnica | Ratio V/C | Prioridad | Sprint |
|------------|---------------|---------------------|-----------|-----------|--------|
| US-004 | 8/10 | 5/10 | **1.60** | Alta | **Sprint 1** |
| US-001 | 10/10 | 7/10 | **1.43** | Alta | **Sprint 1** |
| US-006 | 7/10 | 5/10 | **1.40** | Media | Sprint 2 |
| US-007 | 8/10 | 6/10 | **1.33** | Media | Sprint 2 |
| US-003 | 9/10 | 7/10 | **1.29** | Alta | **Sprint 1** |
| US-002 | 10/10 | 8/10 | **1.25** | Alta | **Sprint 1** |
| US-010 | 5/10 | 6/10 | **0.83** | Baja | Sprint 3 |
| US-009 | 7/10 | 9/10 | **0.78** | Baja | Sprint 3 |
| US-005 | 7/10 | 9/10 | **0.78** | Media | Sprint 2 |
| US-008 | 6/10 | 8/10 | **0.75** | Media | Sprint 3 |

### Análisis

**Sprint 1 (MVP Core - 4 semanas):** US-004, US-001, US-003, US-002  
→ Ratio V/C promedio: 1.39 (excelente)  
→ Entrega valor inmediato: parsing de CVs + matching + pipeline + colaboración

**Sprint 2 (Valor Medio - 3 semanas):** US-006, US-007, US-005  
→ Ratio V/C promedio: 1.18  
→ Features que mejoran UX pero no son bloqueantes para MVP

**Sprint 3 (Post-MVP - 3 semanas):** US-010, US-009, US-008  
→ Ratio V/C promedio: 0.79 (complejidad alta, valor medio)  
→ Nice-to-have que pueden esperar a post-MVP

---

## Backlog Priorizado (BACKLOG FINAL)

### Sprint 1: MVP Core (4 semanas, 27 puntos)

1. **US-004: Pipeline Kanban Visual** [5 puntos]
   - Fundamento del flujo de trabajo
   - Baja complejidad, alto valor
   - Sin dependencias → comenzar primero

2. **US-001: Parsing Automático de CVs** [8 puntos]
   - Feature diferenciador #1
   - Genera embeddings necesarios para US-002
   - Complejidad media pero ROI inmediato

3. **US-003: Colaboración en Tiempo Real** [8 puntos]
   - Feature diferenciador #2 vs competencia
   - WebSocket + Redis Pub/Sub (complejidad técnica interesante)
   - Se puede desarrollar en paralelo con US-001

4. **US-002: Matching Semántico** [6 puntos]
   - Feature diferenciador #3
   - Depende de US-001 (embeddings)
   - Implementar último en Sprint 1

**Objetivo Sprint 1:** MVP funcional con parsing, matching, pipeline y colaboración → Demostrable a early adopters

---

### Sprint 2: Mejoras de UX (3 semanas, 19 puntos)

5. **US-006: Generación de Job Descriptions con IA** [5 puntos]
   - Mejora onboarding de posiciones
   - Baja complejidad, valor medio

6. **US-007: Evaluaciones Estructuradas** [6 puntos]
   - Profesionaliza proceso de evaluación
   - Necesario para clientes enterprise

7. **US-005: Asistente de Voz para Búsqueda** [8 puntos]
   - Feature "wow" para demos
   - Alta complejidad (STT + LLM + TTS)
   - Puede retrasarse si Sprint 2 apretado

**Objetivo Sprint 2:** LTI se diferencia claramente de competencia → Listo para lanzamiento beta

---

### Sprint 3: Automatizaciones (3 semanas, 23 puntos)

8. **US-010: Dictado de Notas por Voz** [6 puntos]
   - Mejora UX de evaluaciones
   - Complejidad media

9. **US-009: Automatizaciones de Workflow** [9 puntos]
   - Feature power-user
   - Alta complejidad (rules engine)
   - Valor alto para clientes que escalan

10. **US-008: Scheduling Automático con Calendario** [8 puntos]
    - Elimina fricción en coordinación
    - Complejidad alta (integraciones)
    - Nice-to-have, no bloqueante

**Objetivo Sprint 3:** LTI escala para equipos grandes → Listo para clientes enterprise

---

## Tickets de Trabajo - US-001

He descompuesto la **US-001 (Parsing Automático de CVs)** en tickets técnicos específicos para el equipo de desarrollo:

### Backend

#### Ticket 1.1: Endpoint de Upload de CVs
**Descripción:** Crear endpoint que recibe archivos PDF/DOCX y los almacena  
**Tareas:**
- Crear endpoint `POST /api/candidates/upload-cv` en FastAPI
- Validar tipo de archivo (PDF, DOCX, TXT) y tamaño máximo 5MB
- Subir archivo a Supabase Storage bucket `candidate-cvs/` con nombre único
- Retornar URL del archivo y UUID del candidato
- Manejo de errores: archivo corrupto, formato no soportado, límite excedido

**Criterios de aceptación:**
- [ ] Endpoint acepta multipart/form-data
- [ ] Rechaza archivos >5MB con error 413
- [ ] Retorna JSON con `{candidate_id, cv_url, status: "uploaded"}`

**Estimación:** 3 puntos (5 horas)

---

#### Ticket 1.2: Parser de CVs con GPT-4
**Descripción:** Función que extrae texto del archivo y usa GPT-4 para estructurar datos  
**Tareas:**
- Extraer texto de PDF usando PyMuPDF (para PDF) y python-docx (para DOCX)
- Crear prompt de extracción estructurado para GPT-4 con JSON schema
- Llamar OpenAI API `gpt-4` con función `parse_cv(cv_text: str) -> CandidateData`
- Parsear respuesta JSON y validar campos requeridos con Pydantic
- Manejar errores: CV ilegible, API timeout, respuesta malformada

**Criterios de aceptación:**
- [ ] Extrae correctamente datos de CVs en español e inglés
- [ ] Precisión >95% en nombre, email, teléfono (medido en test set de 20 CVs)
- [ ] Procesa CV en <10 segundos (incluye llamada a OpenAI)

**Estimación:** 5 puntos (8 horas)

---

#### Ticket 1.3: Generación de Embeddings del CV
**Descripción:** Generar vector embedding del texto completo del CV  
**Tareas:**
- Crear función `generate_embedding(text: str) -> list[float]` que llama OpenAI `text-embedding-3-small`
- Validar que el embedding tenga 1536 dimensiones
- Almacenar embedding en columna `resume_embedding` (tipo VECTOR en PostgreSQL)
- Crear índice HNSW en `resume_embedding` para búsquedas rápidas
- Manejo de errores: texto vacío, API timeout

**Criterios de aceptación:**
- [ ] Genera embedding de 1536 dimensiones
- [ ] Embedding se almacena correctamente en PostgreSQL con pgvector
- [ ] Índice HNSW permite búsquedas <3s en tabla con 10k registros

**Estimación:** 3 puntos (5 horas)

---

#### Ticket 1.4: Task Asíncrono con Celery
**Descripción:** Envolver parsing y embedding en Celery task para procesamiento async  
**Tareas:**
- Crear Celery task `process_uploaded_cv(candidate_id: str, cv_url: str)`
- Task descarga CV, parsea con GPT-4, genera embedding, actualiza BD
- Actualizar estado del candidato: `processing` → `ready`
- Enviar notificación push al frontend cuando termine (WebSocket)
- Configurar retry logic (3 reintentos con backoff exponencial)

**Criterios de aceptación:**
- [ ] Task se ejecuta async sin bloquear request HTTP
- [ ] Frontend recibe notificación WebSocket cuando processing termina
- [ ] En caso de fallo, task hace retry hasta 3 veces

**Estimación:** 5 puntos (8 horas)

---

### Frontend

#### Ticket 1.5: Componente de Upload de CVs
**Descripción:** Interfaz para arrastrar y soltar CVs con preview  
**Tareas:**
- Crear componente React `<CVUploader>` con drag & drop (react-dropzone)
- Mostrar preview del archivo seleccionado (nombre, tamaño, tipo)
- Validar archivo en cliente antes de subir (tipo, tamaño)
- Mostrar progress bar durante upload (axios con onUploadProgress)
- Mostrar mensaje de éxito/error después de upload

**Criterios de aceptación:**
- [ ] Drag & drop funciona en Chrome, Safari, Firefox
- [ ] Progress bar se actualiza en tiempo real
- [ ] Muestra error claro si archivo no válido

**Estimación:** 3 puntos (5 horas)

---

#### Ticket 1.6: Formulario Pre-rellenado de Candidato
**Descripción:** Formulario que muestra datos extraídos del CV para validación  
**Tareas:**
- Crear componente `<CandidateForm>` con React Hook Form
- Campos: nombre, email, teléfono, ubicación, experiencia (lista), educación (lista), skills (chips)
- Pre-rellenar formulario con datos retornados por parser
- Permitir edición de todos los campos antes de guardar
- Botón "Guardar candidato" que hace PUT `/api/candidates/{id}`

**Criterios de aceptación:**
- [ ] Formulario se renderiza en <2s después de parsing
- [ ] Todos los campos son editables
- [ ] Validación de formato (email válido, teléfono con formato correcto)

**Estimación:** 5 puntos (8 horas)

---

#### Ticket 1.7: Indicador de Estado de Procesamiento
**Descripción:** Mostrar spinner/skeleton mientras el CV se procesa  
**Tareas:**
- Escuchar evento WebSocket `cv_processing_complete` con candidate_id
- Mostrar spinner con mensaje "Analizando tu CV con IA..." durante procesamiento
- Cuando procesamiento termina, ocultar spinner y mostrar formulario pre-rellenado
- Si procesamiento falla, mostrar error con opción de "Reintentar"

**Criterios de aceptación:**
- [ ] Spinner visible solo durante procesamiento
- [ ] Transición suave de spinner a formulario
- [ ] Error handling claro con opción de retry

**Estimación:** 2 puntos (3 horas)

---

### Database

#### Ticket 1.8: Schema de Tabla `candidates`
**Descripción:** Crear tabla con columnas para datos estructurados y embedding  
**Tareas:**
- Crear migración Supabase con tabla `candidates`:
  - `id` (UUID, PK)
  - `email` (VARCHAR, UNIQUE)
  - `name`, `phone`, `location` (VARCHAR)
  - `resume_text` (TEXT)
  - `resume_url` (TEXT)
  - `resume_embedding` (VECTOR(1536))
  - `experience` (JSONB)
  - `education` (JSONB)
  - `skills` (JSONB)
  - `created_at`, `updated_at` (TIMESTAMP)
- Crear índice UNIQUE en `email`
- Crear índice HNSW en `resume_embedding`

**Criterios de aceptación:**
- [ ] Migración ejecuta sin errores en dev y staging
- [ ] Columna `resume_embedding` acepta vectores de 1536 dimensiones
- [ ] Índice HNSW permite búsquedas eficientes

**Estimación:** 2 puntos (3 horas)

---

### Testing

#### Ticket 1.9: Tests End-to-End de US-001
**Descripción:** Suite de tests que validan el flujo completo de parsing  
**Tareas:**
- Test 1: Upload de CV válido → retorna candidate_id
- Test 2: Upload de archivo >5MB → retorna error 413
- Test 3: Upload de archivo no-PDF/DOCX → retorna error 400
- Test 4: Parser extrae correctamente datos de CV de prueba
- Test 5: Embedding se genera y almacena correctamente
- Test 6: Formulario se pre-rellena con datos parseados
- Usar CVs de prueba reales (5 PDFs, 5 DOCX en español e inglés)

**Criterios de aceptación:**
- [ ] Todos los tests pasan en CI/CD
- [ ] Cobertura de código >80% en funciones de parsing
- [ ] Tests usan mocks de OpenAI API (no gastar créditos en tests)

**Estimación:** 5 puntos (8 horas)

---

### Resumen de Tickets US-001

| Ticket | Descripción | Estimación | Dependencias |
|--------|-------------|------------|--------------|
| 1.1 | Endpoint upload CVs | 3 puntos | - |
| 1.2 | Parser con GPT-4 | 5 puntos | 1.1 |
| 1.3 | Generación embeddings | 3 puntos | 1.2 |
| 1.4 | Task async Celery | 5 puntos | 1.2, 1.3 |
| 1.5 | Componente upload | 3 puntos | 1.1 |
| 1.6 | Formulario pre-rellenado | 5 puntos | 1.4, 1.5 |
| 1.7 | Indicador procesamiento | 2 puntos | 1.4, 1.6 |
| 1.8 | Schema BD | 2 puntos | - |
| 1.9 | Tests E2E | 5 puntos | 1.1-1.8 |
| **TOTAL** | **US-001 completa** | **33 puntos** | - |

---

## Estimaciones de Esfuerzo

He estimado el esfuerzo de los tickets de US-001 usando **Fibonacci (1, 2, 3, 5, 8, 13, 21)** y **Story Points** (donde 1 punto ≈ 1-2 horas):

### Metodología de Estimación

**1 punto:** Tarea muy simple, <2 horas, sin complejidad técnica (ej: crear endpoint CRUD básico)  
**2 puntos:** Tarea simple, 2-3 horas, complejidad baja (ej: crear schema de BD)  
**3 puntos:** Tarea normal, 4-5 horas, complejidad media (ej: endpoint con validaciones)  
**5 puntos:** Tarea compleja, 6-8 horas, complejidad alta (ej: integración con API externa)  
**8 puntos:** Tarea muy compleja, 1-2 días, complejidad muy alta (ej: implementar WebSocket con Redis)

### Justificación de Estimaciones

**Ticket 1.1 (3 puntos):** Endpoint de upload es estándar, pero requiere validaciones y integración con Supabase Storage.

**Ticket 1.2 (5 puntos):** Parser con GPT-4 es el núcleo de la US. Requiere prompt engineering, manejo de errores, y validación de respuestas. Complejidad alta.

**Ticket 1.3 (3 puntos):** Generación de embeddings es una llamada API + almacenamiento. Complejidad media por configuración de pgvector.

**Ticket 1.4 (5 puntos):** Celery task con retry logic, notificaciones WebSocket y manejo de estados. Complejidad alta.

**Ticket 1.5 (3 puntos):** Componente React con drag & drop. Complejidad media (librería externa bien documentada).

**Ticket 1.6 (5 puntos):** Formulario dinámico con validaciones y edición de listas (experiencia, educación). Complejidad alta.

**Ticket 1.7 (2 puntos):** Indicador de estado con WebSocket listener. Complejidad baja (usa infraestructura existente).

**Ticket 1.8 (2 puntos):** Migración de BD estándar con pgvector. Complejidad baja.

**Ticket 1.9 (5 puntos):** Suite de tests E2E completa con mocks. Complejidad alta (requiere setup de fixtures y CVs de prueba).

**Total US-001: 33 puntos (≈ 50-60 horas de desarrollo = 1.5-2 semanas con 1 desarrollador full-stack)**

---

## Experimentación con Prompts

Durante este ejercicio, experimenté con **5 técnicas diferentes de prompt engineering** basadas en la guía de [promptingguide.ai](https://www.promptingguide.ai/es):

### Prompts Probados

1. **Prompt Simple (Zero-shot)**: Petición directa sin contexto extenso
2. **Prompt Few-Shot**: Incluye ejemplo de User Story como referencia
3. **Prompt Chain-of-Thought**: Razonamiento paso a paso con análisis MoSCoW
4. **Prompt JSON Estructurado**: Output en formato JSON parseable
5. **Prompt ReAct**: Role prompting + Reasoning + Acting con métricas cuantitativas

### Mejor Prompt: ReAct (Prompt 5) ⭐⭐⭐

**Puntuación:** 48/50

**Por qué funcionó mejor:**

1. **Razonamiento explícito**: El prompt fuerza al modelo a pensar antes de actuar (THOUGHT → ACTION → OBSERVATION). Esto genera User Stories con justificación estratégica en vez de simplemente listar features.

2. **Métricas cuantitativas**: Al pedir valor de negocio (X/10), complejidad técnica (X/10) y ratio valor/complejidad, obtengo datos objetivos para priorizar en vez de depender de intuición.

3. **Contexto de negocio**: El role prompting ("eres Lead Product Manager con 10 años en HR Tech") hace que el modelo adopte perspectiva de producto real, no solo técnica.

4. **Estructura multi-paso**: THOUGHT → ACTION → OBSERVATION → BACKLOG FINAL crea un flujo natural que emula cómo trabajaría un PM senior en la realidad.

5. **Roadmap accionable**: El resultado no es solo User Stories aisladas, sino un plan de implementación completo con sprints organizados y dependencias identificadas.

### Comparativa con Otros Prompts

**Prompt 1 (Simple):** Genera User Stories correctas pero sin contexto estratégico. Bueno para equipos que ya tienen priorización definida. **Puntuación:** 33/50

**Prompt 2 (Few-Shot):** Mejora consistencia al seguir ejemplo, pero sigue sin análisis estratégico. Útil cuando necesitas formato muy específico. **Puntuación:** 34/50

**Prompt 3 (Chain-of-Thought):** Excelente para priorización (MoSCoW framework), pero menos cuantitativo que ReAct. Sería mi segunda opción. **Puntuación:** 45/50 ⭐

**Prompt 4 (JSON):** Perfecto para integración con herramientas (Jira, Linear) pero pierde legibilidad para humanos. Mejor para automatización que para planificación. **Puntuación:** 40/50

### Lecciones Aprendidas

1. **Context is King**: Cuanto más contexto estratégico das (competencia, target, dolores), mejor es el output.

2. **Frameworks explícitos funcionan**: Mencionar MoSCoW, ratio valor/complejidad, o ReAct guía al modelo mejor que "prioriza las User Stories".

3. **Multi-paso > Single-shot**: Prompts que fuerzan razonamiento paso a paso (Chain-of-Thought, ReAct) generan output significativamente mejor que prompts directos.

4. **Role prompting añade perspectiva**: Hacer que el modelo actúe como "Lead PM con 10 años de experiencia" cambia dramáticamente la profundidad del análisis.

5. **Métricas cuantitativas > cualitativas**: Pedir valor/complejidad en escala 1-10 permite comparar User Stories objetivamente vs descripciones vagas como "alta prioridad".

### Recomendación para Futuros Ejercicios

Para ejercicios de Product Management, usar **ReAct** o **Chain-of-Thought**.  
Para ejercicios de integración técnica, usar **JSON Estructurado**.  
Para generación rápida sin análisis profundo, usar **Few-Shot**.

---

**Fin del documento UserStories-AJM.md**
