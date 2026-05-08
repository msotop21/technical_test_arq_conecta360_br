Conecta360 – Propuesta de Arquitectura Tecnológica

Sistema Integral de Atención Ciudadana (SIAC)

📌 Descripción General

Conecta360 es un Sistema Integral de Atención Ciudadana diseñado para centralizar, modernizar y dar trazabilidad a la gestión de solicitudes, quejas, reclamos, sugerencias y servicios ciudadanos, habilitando una experiencia omnicanal, eficiente y auditable.

Este repositorio documenta la Propuesta de Arquitectura Tecnológica, que sirve como marco de referencia para el diseño, desarrollo, integración, despliegue y evolución de la plataforma.

🎯 Objetivos de la Arquitectura
Centralizar todos los canales de atención (web, móvil, redes sociales, call center).
Garantizar trazabilidad completa del ciclo de vida de los casos.
Facilitar la integración con sistemas internos y externos (incluyendo legados).
Asegurar escalabilidad, alta disponibilidad y resiliencia.
Establecer gobierno de APIs, seguridad y observabilidad end-to-end.
Proveer una base tecnológica sostenible y alineada a estándares.
🏗️ Alcance de la Propuesta

La arquitectura cubre:

Componentes de aplicación y datos
Integración y mensajería
Seguridad e identidad
Gobierno de APIs
Observabilidad y monitoreo
Infraestructura y operación

Dirigida a:

Negocio
Arquitectura Empresarial y Tecnológica
Desarrollo
Ciberseguridad
Operaciones
Proveedores tecnológicos
🧩 Stack Tecnológico
Componente	Tecnología	Justificación	Alternativa
Backend	NestJS	Arquitectura modular, orientada a microservicios	Spring Boot
Orquestación BPM	Camunda 8	Modelado visual de procesos complejos	Bizagi
Base de Datos Principal	PostgreSQL	Robustez transaccional, JSONB, open source	MySQL / Oracle
Base No Relacional	MongoDB	Logs y estructuras JSON variables	—
BI / Analítica	Apache Superset + BigQuery	Bajo costo operativo, integración BI	Power BI Premium
Mensajería	Apache Kafka	Alta capacidad y persistencia de eventos	RabbitMQ
Cache / Sesiones	Redis	Baja latencia y alto rendimiento	Memcached
Frontend	React.js + Next.js	SEO, carga progresiva	Angular / Vue
Identidad (SSO)	Keycloak	OIDC y SAML, open source	Auth0
IA / Chatbot	FastAPI + LangChain	Clasificación automática de tickets	—
API Gateway	Kong	Gobierno, seguridad y escalabilidad	AWS API Gateway
Observabilidad	Prometheus + Grafana + Loki	Observabilidad sin costo por volumen	Datadog
CI/CD	GitLab CI	Automatización y seguridad integrada	Azure Pipelines
🧠 Patrones Arquitectónicos

La solución adopta los siguientes patrones:

Layered Architecture
Organización clara y gobernabilidad tecnológica.
Microservicios
Servicios independientes por dominio (casos, ciudadanos, notificaciones, analítica).
Event-Driven Architecture
Comunicación asíncrona mediante Kafka para absorber picos de carga.
CQRS
Escrituras transaccionales y lecturas analíticas desacopladas.
Outbox Pattern
Garantiza consistencia entre base de datos y mensajería.
Adapter / Sidecar
Integración con sistemas legados sin APIs.
API Gateway
Punto único de entrada, control de tráfico y seguridad.
📐 Diagramas de Arquitectura

Los diagramas incluidos en este proyecto cubren:

Diagrama de Contexto
Diagrama de Contenedores
Diagrama de Componentes de Alto Nivel
Arquitectura BPM
Alta Disponibilidad Geográfica
Arquitectura de Identidad
Arquitectura de Base de Datos

Los diagramas se mantienen en formatos compatibles con herramientas abiertas.

🔌 Diseño de APIs
Convenciones Generales
Base URL: https://api.conecta360.gov.do/v1
Autenticación: Authorization: Bearer <JWT>
Estilo: REST + eventos asíncronos
📄 REST – Casos
Crear Caso

POST /cases

{
  "citizen_id": "uuid",
  "category": "alumbrado_publico",
  "subcategory": "lampara_apagada",
  "description": "Farola sin luz",
  "location": { "lat": 9.9281, "lng": -84.0907 },
  "channel": "web"
}
Consultar Caso

GET /cases/{case_number}

Respuesta incluye estado, eventos, responsable y SLA.

📊 REST – KPIs

GET /dashboard/kpis

Disponible para roles supervisor / administrador.

📡 Eventos Asíncronos (Kafka)
cases.created

Evento emitido al registrar un nuevo caso ciudadano.

cases.status_changed

Evento emitido cuando cambia el estado de un caso.

Los esquemas siguen el estándar AsyncAPI 2.6.

⏱️ Línea de Tiempo del Proyecto

La línea de tiempo incluida es referencial, con el objetivo de ilustrar una ejecución ideal del proyecto, no necesariamente ajustada a una planificación real.

📁 Estructura Recomendada del Repositorio
conecta360-architecture/
│
├── README.md
├── docs/
│   ├── arquitectura/
│   ├── diagramas/
│   ├── seguridad/
│   └── integraciones/
│
├── api/
│   ├── openapi/
│   └── asyncapi/
│
├── bpm/
│   └── procesos/
│
└── referencias/
📚 Referencias y Herramientas
Draw.io (Diagrams.net)
Mermaid Live
PlantUML
