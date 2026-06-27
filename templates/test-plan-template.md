# Test Plan Template for Enterprise Applications

## 📋 Propósito

Este template establece el estándar para la planificación, ejecución y gestión de pruebas en aplicaciones enterprise del contexto ERLN (retail masivo, 19,556 sucursales, misión crítica 24/7). Su objetivo es garantizar que cada release cumpla con los **SLAs de 99.99%**, los requerimientos de **PCI-DSS**, y los estándares de calidad corporativos, minimizando el riesgo de defectos en producción.

**Principios de Testing:**
- ✅ **Shift-Left Testing**: Detectar defectos lo más temprano posible en el ciclo de vida.
- ✅ **Testing Pyramid**: Mayor cobertura en unit tests, menor en E2E (evitar "ice cream cone").
- ✅ **Risk-Based Testing**: Priorizar pruebas basadas en impacto de negocio y probabilidad de fallo.
- ✅ **Automation-First**: Automatizar todo lo repetible, manual solo lo exploratorio.
- ✅ **Production-Like Environments**: Ambientes de prueba deben reflejar producción (infra, datos, configuración).
- ❌ **Prohibido**: Saltar pruebas de performance "porque ya pasó en QA".
- ❌ **Prohibido**: Usar datos reales de producción en ambientes no productivos (PCI-DSS violation).
- ❌ **Prohibido**: Aprobar releases con defectos CRITICAL o HIGH abiertos.

---

## 🎯 Alcance del Plan de Pruebas

### 1.1 Identificación del Proyecto
| Campo | Valor |
|-------|-------|
| **Nombre del Proyecto** | [Ej. Migración Módulo de Pagos a Microservicios] |
| **Versión/Release** | [Ej. v2.1.0] |
| **Dominio de Negocio** | [Ej. Pagos, Inventario, Clientes] |
| **Arquitecto Responsable** | [Nombre] |
| **QA Lead** | [Nombre] |
| **Fecha de Inicio** | [YYYY-MM-DD] |
| **Fecha de Go-Live** | [YYYY-MM-DD] |
| **SLA Objetivo** | 99.99% (máximo 4.38 minutos downtime/mes) |

### 1.2 Alcance (In-Scope)
- [ ] Funcionalidades nuevas desarrolladas en este release.
- [ ] Módulos refactorizados o migrados (Strangler Fig).
- [ ] Integraciones con sistemas externos (APIs de terceros, bancos, proveedores).
- [ ] Cambios en base de datos Oracle 12c (nuevas tablas, índices, stored procedures).
- [ ] Configuración de Kong Enterprise (nuevas rutas, plugins, rate limiting).
- [ ] Cambios en infraestructura (Kubernetes, WebLogic, redes).

### 1.3 Fuera de Alcance (Out-of-Scope)
- [ ] Funcionalidades no modificadas en este release (regresión limitada a áreas afectadas).
- [ ] Sistemas legacy no integrados directamente.
- [ ] Pruebas de usabilidad (si aplica, se manejan en otro plan).
- [ ] Pruebas de compatibilidad de navegadores antiguos (IE11 no soportado).

---

## 🏗️ Estrategia de Pruebas (Testing Pyramid)

### 2.1 Pirámide de Pruebas

```
        /\
       /  \      E2E / UI Tests (10%)
      /----\     - Selenium, RestAssured
     /      \    - Flujos críticos de negocio
    /--------\   
   /          \  Integration / Contract Tests (20%)
  /------------\ - Testcontainers, Pact
 /              \- APIs, DB, message brokers
/----------------\
/                  \ Unit Tests (70%)
/--------------------\ - JUnit 5, Mockito
                      - Cobertura ≥ 80%
```

### 2.2 Distribución por Tipo de Prueba

| Nivel | Herramienta | Cobertura Mínima | Ejecución | Responsable |
|-------|-------------|------------------|-----------|-------------|
| **Unit Tests** | JUnit 5 + Mockito | 80% código | Cada commit (CI) | Developers |
| **Integration Tests** | Testcontainers + Spring Boot Test | Módulos críticos | Cada PR (CI) | Developers + QA |
| **Contract Tests** | Pact (consumer-driven) | APIs expuestas | Cada PR (CI) | Developers + Consumer Teams |
| **Component Tests** | Spring Boot Test + Mocks | Servicio aislado | Cada PR (CI) | Developers |
| **E2E / Smoke Tests** | Selenium / RestAssured | Flujos críticos | Pre-prod (CD) | QA Automation |
| **Performance Tests** | JMeter / Gatling | Escenarios pico | Pre-release | Performance Team |
| **Security Tests** | OWASP ZAP + Checkmarx | Todos los endpoints | Pre-release | Security Team |
| **Chaos Tests** | Gremlin / Litmus | Servicios críticos | Trimestral (staging) | SRE + QA |
| **Exploratory Tests** | Manual | Áreas de alto riesgo | Pre-release | QA Manual |

### 2.3 Criterios de Aceptación por Nivel

#### Unit Tests
- ✅ Cobertura de código ≥ 80% (JaCoCo).
- ✅ Mutación testing ≥ 60% (PIT).
- ✅ Tiempo de ejecución < 5 minutos.
- ✅ Sin dependencias externas (mockeadas).

#### Integration Tests
- ✅ Base de datos Oracle 12c embebida (Testcontainers).
- ✅ Message brokers (RabbitMQ/Kafka) embebidos.
- ✅ Validación de transacciones distribuidas (Sagas).
- ✅ Tiempo de ejecución < 15 minutos.

#### Contract Tests
- ✅ Consumer-driven contracts definidos (Pact).
- ✅ Provider verifica cumplimiento en cada build.
- ✅ Pact broker centralizado (versionado).
- ✅ Breaking changes detectados automáticamente.

#### E2E Tests
- ✅ Flujos críticos de negocio cubiertos (ej. proceso de pago, aprobación de crédito).
- ✅ Datos de prueba aislados (no interfieren con otros tests).
- ✅ Tiempo de ejecución < 30 minutos.
- ✅ Ejecución en ambiente staging productivo.

---

## 🧪 Tipos de Pruebas Detalladas

### 3.1 Pruebas Funcionales

#### 3.1.1 Pruebas de Caja Negra
- **Objetivo**: Validar que el sistema cumple con los requerimientos de negocio.
- **Técnicas**:
  - Equivalence partitioning (dividir inputs en clases equivalentes).
  - Boundary value analysis (probar límites de rangos).
  - Decision table testing (combinaciones de condiciones).
  - State transition testing (cambios de estado en entidades).
- **Herramientas**: Test cases en Jira/Xray, ejecución manual o automatizada.

#### 3.1.2 Pruebas de Regresión
- **Objetivo**: Validar que cambios no rompieron funcionalidad existente.
- **Estrategia**:
  - **Regresión completa**: Solo para releases mayores (cada 3 meses).
  - **Regresión selectiva**: Basada en impacto de cambios (análisis de dependencias).
  - **Regresión automatizada**: Suite de E2E tests ejecutada en cada release candidate.
- **Herramientas**: Selenium, RestAssured, JUnit.

#### 3.1.3 Pruebas de Humo (Smoke Tests)
- **Objetivo**: Validar que las funcionalidades críticas están operativas después de un despliegue.
- **Alcance**:
  - Health checks de todos los microservicios.
  - Endpoints críticos (ej. login, proceso de pago, consulta de inventario).
  - Integraciones con sistemas externos (conexión a bancos, proveedores).
- **Ejecución**: Automática post-deploy en pipeline CI/CD.
- **Criterio de éxito**: 100% de smoke tests pasando. Si falla, rollback automático.

### 3.2 Pruebas No Funcionales

#### 3.2.1 Pruebas de Performance

**Objetivo**: Validar que el sistema cumple con los SLAs de latencia y throughput bajo carga.

**Escenarios de Prueba**:
| Escenario | Descripción | Carga | Duración | SLA Objetivo |
|-----------|-------------|-------|----------|--------------|
| **Baseline** | Carga normal de operación | 100 users | 30 min | p99 < 500ms |
| **Load Test** | Carga esperada en pico (ej. Black Friday) | 1,000 users | 1 hora | p99 < 1s, error rate < 0.1% |
| **Stress Test** | Llevar el sistema al límite | 5,000 users | 30 min | Graceful degradation, no crash |
| **Soak Test** | Carga sostenida para detectar memory leaks | 500 users | 24 horas | Memory usage estable |
| **Spike Test** | Aumento súbito de carga | 0 → 2,000 users en 1 min | 15 min | Auto-scaling funciona |

**Herramientas**: JMeter, Gatling, k6.

**Métricas a Monitorear**:
- Latencia (p50, p95, p99).
- Throughput (requests/second).
- Error rate (%).
- CPU, memory, disk I/O (Kubernetes metrics).
- JVM metrics (heap, GC, threads).
- Database metrics (Oracle AWR, active sessions, wait events).

**Criterios de Aceptación**:
- ✅ p99 latency < 500ms (servicios críticos).
- ✅ Error rate < 0.1% bajo carga normal.
- ✅ Throughput cumple con capacidad planificada.
- ✅ No memory leaks (soak test 24h estable).
- ✅ Auto-scaling funciona (spike test).

#### 3.2.2 Pruebas de Seguridad

**Objetivo**: Validar que el sistema cumple con PCI-DSS y políticas de seguridad corporativa.

**Tipos de Pruebas**:
| Tipo | Herramienta | Frecuencia | Responsable |
|------|-------------|------------|-------------|
| **SAST** (Static Application Security Testing) | Checkmarx, SonarQube Security | Cada commit (CI) | Developers |
| **SCA** (Software Composition Analysis) | Snyk, OWASP Dependency-Check | Cada PR (CI) | Developers |
| **DAST** (Dynamic Application Security Testing) | OWASP ZAP | Pre-release | Security Team |
| **Penetration Testing** | Manual + herramientas | Trimestral / Pre-go-live | External Security Firm |
| **Secrets Detection** | GitLeaks, TruffleHog | Cada commit (CI) | Developers |
| **Container Image Scanning** | Trivy, Snyk Container | Cada build (CI) | DevOps |

**Validaciones PCI-DSS**:
- ✅ PAN enmascarado en logs y respuestas de API (`**** **** **** 1234`).
- ✅ CVV/CVC NUNCA almacenado (validado en code review y DAST).
- ✅ Tokenización de PAN implementada (reemplazo con token de proveedor certificado).
- ✅ TLS 1.2+ en todos los endpoints (cipher suites fuertes).
- ✅ TDE habilitado en Oracle 12c para datos sensibles.
- ✅ Audit trail activado (Oracle Audit Vault).

**Criterios de Aceptación**:
- ✅ 0 vulnerabilidades CRITICAL o HIGH (SAST, SCA, DAST).
- ✅ Penetration test aprobado sin hallazgos críticos.
- ✅ PCI-DSS SAQ completado y firmado.
- ✅ Security review aprobada por CISO.

#### 3.2.3 Pruebas de Resiliencia (Chaos Engineering)

**Objetivo**: Validar que el sistema se degrada elegantemente ante fallos.

**Experimentos de Chaos**:
| Experimento | Herramienta | Hipótesis | Criterio de Éxito |
|-------------|-------------|-----------|-------------------|
| **Matar pods del microservicio** | Gremlin / Litmus | Circuit breaker (Resilience4j) se abre, Kong rutea a fallback | Error rate < 1%, recuperación automática en < 30s |
| **Introducir latencia de red** | Gremlin | Timeouts y retries funcionan (exponential backoff) | p99 latency < 2s, no cascading failures |
| **Saturar CPU/memoria** | Gremlin | Auto-scaling se activa, bulkhead aísla recursos | No OOM kills, otros servicios no afectados |
| **Cortar conexión a base de datos** | Gremlin | Graceful degradation, mensajes en cola para retry | No data loss, recuperación automática |
| **Corromper respuestas de dependencias** | WireMock / Gremlin | Fallback methods retornan datos por defecto | Usuario ve mensaje de degradación, no error 500 |
| **Simular fallo de Kong Enterprise** | Chaos Mesh | DNS failover a instancia standby | Downtime < 5 segundos |

**Frecuencia**: Trimestral en ambiente staging.

**Criterios de Aceptación**:
- ✅ Todos los experimentos pasan sin violar SLA.
- ✅ Circuit breakers (Resilience4j) funcionan correctamente.
- ✅ Fallbacks retornan respuestas válidas (degradación elegante).
- ✅ Auto-scaling y auto-healing de Kubernetes funcionan.
- ✅ Post-mortem documentado con action items.

#### 3.2.4 Pruebas de Disponibilidad (Disaster Recovery)

**Objetivo**: Validar que el sistema se recupera ante desastres dentro del RTO/RPO.

**Escenarios**:
| Escenario | RTO Objetivo | RPO Objetivo | Frecuencia |
|-----------|--------------|--------------|------------|
| **Fallo de zona de disponibilidad (AZ)** | < 30 minutos | < 5 minutos | Semestral |
| **Fallo de región (DR site)** | < 2 horas | < 15 minutos | Anual |
| **Corrupción de base de datos** | < 1 hora | < 5 minutos (Oracle Data Guard) | Trimestral |
| **Ataque DDoS** | < 15 minutos (mitigación) | N/A | Trimestral |

**Validaciones**:
- ✅ Oracle Data Guard failover funciona (standby → primary).
- ✅ Kubernetes cluster multi-AZ configurado.
- ✅ Backups restaurados exitosamente (prueba trimestral).
- ✅ DNS failover a DR site funciona.
- ✅ Kong Enterprise HA configurado (active-active o active-passive).

---

## 🌍 Ambientes de Prueba

### 4.1 Estrategia de Ambientes

| Ambiente | Propósito | Datos | Infraestructura | Acceso |
|----------|-----------|-------|-----------------|--------|
| **DEV** | Desarrollo y unit tests | Sintéticos | Kubernetes namespace compartido | Developers |
| **SIT** (System Integration Testing) | Integration tests, contract tests | Sintéticos + subset anonimizado | Kubernetes namespace dedicado | Developers + QA |
| **QA** | E2E tests, regression, exploratory | Subset anonimizado de prod | Réplica de prod (50% capacidad) | QA Team |
| **STAGING** (Pre-Prod) | Performance, security, UAT, chaos | Anonimizado de prod (masking) | Idéntico a prod (100% capacidad) | QA + Business Users |
| **PROD** | Producción | Reales | Producción | Ops Team (on-call) |

### 4.2 Gestión de Datos de Prueba

**Principios**:
- ✅ **Nunca usar datos reales de producción** en ambientes no productivos (PCI-DSS violation).
- ✅ **Data masking** obligatorio para PII y datos de tarjeta (Oracle Data Redaction).
- ✅ **Synthetic data generation** para casos de prueba (herramienta: Faker, Mockaroo).
- ✅ **Data subsetting** para ambientes QA/staging (subset representativo de prod).
- ✅ **Data isolation** por ejecución de tests (no interferencia entre suites).

**Herramientas**:
- **Oracle Data Redaction**: Mascarado dinámico en queries.
- **Delphix**: Virtualización y masking de datos.
- **Faker (Java)**: Generación de datos sintéticos.
- **DbUnit**: Setup/teardown de datos para integration tests.

**Flujo de Datos**:
```
PROD (datos reales) 
  → Data Extraction (subset 10%) 
  → Data Masking (PII, PAN, passwords) 
  → Data Loading (QA/STAGING)
  → Data Refresh (semanal)
```

### 4.3 Configuración de Ambientes

**Kubernetes Namespaces**:
- `dev-[service-name]`
- `sit-[service-name]`
- `qa-[service-name]`
- `staging-[service-name]`
- `prod-[service-name]`

**ConfigMaps y Secrets**:
- ConfigMaps versionados en Git (Kustomize overlays por ambiente).
- Secrets en HashiCorp Vault (inyección vía sidecar o CSI driver).
- Feature flags en LaunchDarkly/Unleash (configuración por ambiente).

**Base de Datos Oracle 12c**:
- Esquemas separados por ambiente.
- TDE habilitado en todos los ambientes (datos sensibles).
- Oracle Data Guard configurado en STAGING y PROD.

---

## 📋 Criterios de Entrada y Salida

### 5.1 Criterios de Entrada (Entry Criteria)

#### Para Inicio de Pruebas de Integración (SIT)
- [ ] Unit tests pasando con cobertura ≥ 80%.
- [ ] Código revisado y aprobado (pull request merged).
- [ ] SonarQube Quality Gate verde (0 vulnerabilities, 0 bugs).
- [ ] API contracts definidos y publicados (OpenAPI 3.0).
- [ ] Ambiente SIT desplegado y estable.
- [ ] Datos de prueba disponibles (sintéticos o anonimizados).
- [ ] Test cases documentados y aprobados.

#### Para Inicio de Pruebas E2E (QA)
- [ ] Integration tests pasando.
- [ ] Contract tests pasando (Pact broker actualizado).
- [ ] Ambiente QA desplegado con versión candidata a release.
- [ ] Datos de prueba anonimizados cargados.
- [ ] Integraciones con sistemas externos configuradas y validadas.
- [ ] Smoke tests pasando.

#### Para Inicio de Pruebas de Performance (STAGING)
- [ ] E2E tests pasando en QA.
- [ ] Ambiente STAGING idéntico a PROD (capacidad, configuración).
- [ ] Datos de prueba anonimizados cargados (volumen realista).
- [ ] Herramientas de load testing configuradas (JMeter/Gatling).
- [ ] Monitoreo habilitado (Prometheus, Grafana, Splunk, Oracle AWR).
- [ ] Baseline de performance documentado (release anterior).

#### Para Inicio de UAT (User Acceptance Testing)
- [ ] Pruebas funcionales y de regresión pasando.
- [ ] Performance tests pasando (SLA cumplido).
- [ ] Security scans limpios (0 CRITICAL/HIGH).
- [ ] Ambiente STAGING estable.
- [ ] Business users disponibles y capacitados.
- [ ] Casos de prueba de negocio aprobados por product owner.

### 5.2 Criterios de Salida (Exit Criteria)

#### Para Aprobación de Release a Producción
- [ ] **Unit tests**: Cobertura ≥ 80%, mutación ≥ 60%.
- [ ] **Integration tests**: 100% pasando.
- [ ] **Contract tests**: 100% pasando (Pact broker verde).
- [ ] **E2E tests**: 100% de flujos críticos pasando.
- [ ] **Performance tests**: SLA cumplido (p99 < 500ms, error rate < 0.1%).
- [ ] **Security tests**: 0 vulnerabilidades CRITICAL/HIGH, PCI-DSS validado.
- [ ] **Chaos tests**: Experimentos críticos pasando (resiliencia validada).
- [ ] **UAT**: Sign-off de business owner.
- [ ] **Defectos**: 0 CRITICAL, 0 HIGH abiertos. MEDIUM/LOW con plan de mitigación aprobado.
- [ ] **Documentation**: Runbooks, API docs, architecture diagrams actualizados.
- [ ] **Rollback plan**: Documentado y probado en STAGING.
- [ ] **CAB approval**: Change Advisory Board aprobó el release.

---

## 👥 Roles y Responsabilidades

### 6.1 Matriz RACI

| Actividad | Arquitecto | QA Lead | Dev Lead | QA Automation | Security | DevOps | Business Owner |
|-----------|------------|---------|----------|---------------|----------|--------|----------------|
| **Definir estrategia de pruebas** | A | R | C | C | C | I | I |
| **Escribir test cases** | I | A | C | R | C | I | C |
| **Automatizar tests** | I | A | C | R | I | C | I |
| **Ejecutar tests manuales** | I | A | I | R | I | I | C |
| **Ejecutar tests de performance** | C | A | I | R | I | C | I |
| **Ejecutar tests de seguridad** | C | I | I | I | R | C | I |
| **Ejecutar chaos tests** | A | C | C | R | C | R | I |
| **Gestionar defectos** | I | A | R | R | C | I | I |
| **Aprobar release** | R | C | C | I | C | I | A |
| **Decidir go/no-go** | R | R | R | I | C | C | A |

**Leyenda**: R = Responsible (ejecuta), A = Accountable (aprueba), C = Consulted (consulta), I = Informed (informa).

### 6.2 Responsabilidades Clave

#### Arquitecto de Aplicaciones
- Definir estrategia de pruebas alineada con arquitectura.
- Validar que pruebas cubren riesgos arquitectónicos (resiliencia, escalabilidad, seguridad).
- Aprobar criterios de entrada/salida.
- Participar en go/no-go decisions.

#### QA Lead
- Planificar y coordinar todas las actividades de pruebas.
- Definir criterios de entrada/salida.
- Gestionar defectos y priorizar correcciones.
- Reportar métricas de calidad a stakeholders.
- Aprobar release desde perspectiva de calidad.

#### Development Lead
- Asegurar que unit e integration tests están implementados.
- Corregir defectos reportados por QA.
- Participar en code reviews para validar calidad.
- Aprobar release desde perspectiva técnica.

#### QA Automation Engineer
- Automatizar test cases (E2E, regression, smoke).
- Mantener suite de automatización (actualizar, refactorizar).
- Ejecutar tests automatizados en pipeline CI/CD.
- Reportar resultados y métricas de automatización.

#### Security Engineer
- Ejecutar SAST, SCA, DAST.
- Coordinar penetration testing externo.
- Validar cumplimiento PCI-DSS.
- Aprobar release desde perspectiva de seguridad.

#### DevOps Engineer
- Provisionar y mantener ambientes de prueba.
- Integrar tests en pipeline CI/CD.
- Gestionar datos de prueba (masking, subsetting).
- Soportar ejecución de chaos tests.

#### Business Owner
- Definir casos de prueba de negocio (UAT).
- Ejecutar UAT y validar funcionalidad.
- Aprobar release desde perspectiva de negocio.
- Comunicar impacto a usuarios finales.

---

## 📊 Métricas y KPIs de Calidad

### 7.1 Métricas de Ejecución de Pruebas

| Métrica | Fórmula | Objetivo | Frecuencia de Reporte |
|---------|---------|----------|----------------------|
| **Test Case Execution Rate** | (Test cases ejecutados / Test cases planificados) × 100 | 100% antes de go-live | Diario durante ejecución |
| **Test Case Pass Rate** | (Test cases pasando / Test cases ejecutados) × 100 | ≥ 98% | Diario durante ejecución |
| **Defect Detection Percentage (DDP)** | (Defectos encontrados en QA / Defectos totales) × 100 | ≥ 90% (menos del 10% escapa a prod) | Por release |
| **Defect Leakage** | (Defectos encontrados en PROD / Defectos totales) × 100 | ≤ 5% | Por release |
| **Automation Coverage** | (Test cases automatizados / Test cases totales) × 100 | ≥ 70% para regression | Mensual |
| **Automation Stability** | (Ejecuciones exitosas / Ejecuciones totales) × 100 | ≥ 95% (flaky tests < 5%) | Semanal |

### 7.2 Métricas de Defectos

| Métrica | Fórmula | Objetivo | Frecuencia de Reporte |
|---------|---------|----------|----------------------|
| **Defect Density** | Defectos / KLOC (miles de líneas de código) | < 1 defecto / KLOC | Por release |
| **Defect Severity Distribution** | % CRITICAL, HIGH, MEDIUM, LOW | 0 CRITICAL, 0 HIGH abiertos | Diario durante ejecución |
| **Defect Age** | Promedio de días que un defecto permanece abierto | < 3 días para CRITICAL, < 5 días para HIGH | Diario |
| **Defect Rejection Rate** | (Defectos rechazados / Defectos reportados) × 100 | ≤ 10% (indica calidad de reporting) | Por release |
| **Defect Rework Rate** | (Defectos reabiertos / Defectos corregidos) × 100 | ≤ 5% (indica calidad de fixes) | Por release |

### 7.3 Métricas de Performance

| Métrica | Herramienta | Objetivo | Frecuencia |
|---------|-------------|----------|------------|
| **p50 Latency** | Prometheus / Grafana | < 200ms | Continuo en prod |
| **p95 Latency** | Prometheus / Grafana | < 400ms | Continuo en prod |
| **p99 Latency** | Prometheus / Grafana | < 500ms | Continuo en prod |
| **Error Rate** | Prometheus / Grafana | < 0.1% | Continuo en prod |
| **Throughput** | Prometheus / Grafana | ≥ capacidad planificada | Continuo en prod |
| **Availability** | Prometheus + Pingdom | 99.99% (4.38 min downtime/mes) | Mensual |

### 7.4 Dashboards de Calidad

**Dashboard 1: Quality Overview (QA Lead)**
- Test execution progress (ejecutados vs planificados).
- Pass/fail rate por tipo de prueba.
- Defect trend (abiertos vs cerrados).
- Defect severity distribution.
- Top 10 defectos por impacto.

**Dashboard 2: Performance Monitoring (SRE / DevOps)**
- Latency percentiles (p50, p95, p99) en tiempo real.
- Error rate por endpoint.
- Throughput (requests/second).
- Resource utilization (CPU, memory, disk, network).
- Database metrics (active sessions, wait events).

**Dashboard 3: Security Posture (Security Team)**
- Vulnerability trend (SAST, SCA, DAST).
- Vulnerabilities por severidad (CRITICAL, HIGH, MEDIUM, LOW).
- Mean time to remediate (MTTR) por severidad.
- Compliance score (PCI-DSS requirements).
- Secrets detection alerts.

**Dashboard 4: Business Impact (Business Owner)**
- Availability (uptime %).
- Transaction success rate.
- Revenue impact (transacciones procesadas).
- Customer experience metrics (NPS, CSAT).
- Defect leakage a producción.

---

## 🐛 Gestión de Defectos

### 8.1 Severidad y Prioridad

#### Severidad (Impacto Técnico)
| Severidad | Definición | Ejemplo | SLA de Corrección |
|-----------|------------|---------|-------------------|
| **CRITICAL** | Sistema caído o funcionalidad crítica completamente rota, sin workaround | No se pueden procesar pagos, base de datos corrupta | < 4 horas |
| **HIGH** | Funcionalidad crítica degradada, workaround difícil o inexistente | Latencia > 5s en proceso de pago, pérdida de datos | < 8 horas |
| **MEDIUM** | Funcionalidad no crítica rota, workaround existe | Reporte no se genera, pero proceso de pago funciona | < 3 días |
| **LOW** | Problema cosmético o menor, no afecta funcionalidad | Typo en UI, color incorrecto | < 7 días |

#### Prioridad (Urgencia de Negocio)
| Prioridad | Definición | Criterio |
|-----------|------------|----------|
| **P1 - Immediate** | Debe corregirse ya, bloquea release | CRITICAL severity + impacto en revenue o SLA |
| **P2 - High** | Debe corregirse en este release | HIGH severity o MEDIUM con alto impacto de negocio |
| **P3 - Medium** | Puede corregirse en próximo release | MEDIUM severity con workaround |
| **P4 - Low** | Se corrige cuando haya capacidad | LOW severity, cosmético |

### 8.2 Flujo de Vida de Defectos

```
[New] → [Triaged] → [Assigned] → [In Progress] → [Fixed] → [Ready for QA] → [Verified] → [Closed]
                                                                                              ↓
                                                                                         [Reopened] (si falla validación)
```

**Estados**:
- **New**: Defecto reportado, pendiente de triage.
- **Triaged**: Severidad y prioridad asignadas por QA Lead.
- **Assigned**: Asignado a desarrollador responsable.
- **In Progress**: Desarrollador trabajando en la corrección.
- **Fixed**: Corrección implementada, lista para validación.
- **Ready for QA**: Desplegado en ambiente de pruebas, QA puede validar.
- **Verified**: QA validó que el defecto está corregido.
- **Closed**: Defecto cerrado, no requiere más acción.
- **Reopened**: QA validó que el defecto persiste o regresó.
- **Deferred**: Se decide corregir en futuro release (requiere aprobación de Business Owner).
- **Rejected**: No es defecto (diseño correcto, duplicado, no reproducible).

### 8.3 Herramientas de Gestión de Defectos

**Jira + Xray**:
- **Jira**: Tracking de defectos, workflows, reportes.
- **Xray**: Gestión de test cases, ejecución, trazabilidad a requerimientos.

**Integraciones**:
- **Jira ↔ GitHub**: Vincular defectos a pull requests y commits.
- **Jira ↔ Jenkins**: Actualizar estado de defectos automáticamente desde pipeline.
- **Jira ↔ Splunk**: Vincular defectos a logs y traces para debugging.
- **Jira ↔ ServiceNow**: Escalar defectos CRITICAL a incident management.

### 8.4 Defect Triage Meetings

**Frecuencia**: Diario durante ejecución de pruebas, semanal en mantenimiento.

**Participantes**: QA Lead, Dev Lead, Arquitecto, Product Owner, Security (si aplica).

**Agenda**:
1. Revisar defectos nuevos (asignar severidad y prioridad).
2. Revisar defectos bloqueados (dependencias, falta de información).
3. Revisar defectos CRITICAL/HIGH (plan de acción inmediato).
4. Revisar métricas de defectos (tendencia, aging).
5. Decidir go/no-go para release (si aplica).

---

## ⚠️ Riesgos y Mitigaciones

### 9.1 Matriz de Riesgos de Pruebas

| ID | Riesgo | Probabilidad (1-5) | Impacto (1-5) | Score | Mitigación | Contingencia |
|----|--------|--------------------|---------------|-------|------------|--------------|
| R1 | Ambientes de prueba inestables o no disponibles | 3 | 4 | 12 | Ambientes gestionados por DevOps, monitoreo 24/7, SLA de disponibilidad 99.9% | Usar ambiente alternativo o mockear dependencias |
| R2 | Datos de prueba insuficientes o corruptos | 2 | 4 | 8 | Data refresh semanal, validación automática de integridad, synthetic data generation | Generar datos on-demand con scripts de emergencia |
| R3 | Defectos CRITICAL descubiertos tarde en el ciclo | 3 | 5 | 15 | Shift-left testing, code reviews estrictos, SonarQube gates | Extender timeline de pruebas, reducir alcance de release |
| R4 | Performance degradation no detectada en QA | 2 | 5 | 10 | Performance tests en cada release, baseline comparison, monitoring en prod | Rollback automático si SLA se viola post-deploy |
| R5 | Vulnerabilidades de seguridad descubiertas en producción | 2 | 5 | 10 | SAST/SCA/DAST en CI/CD, penetration testing trimestral, bug bounty program | Parche de emergencia vía hotfix, comunicación a clientes si aplica |
| R6 | Automatización de tests frágil (flaky tests) | 4 | 3 | 12 | Code reviews de tests, retry mechanisms, isolate flaky tests, root cause analysis | Ejecutar tests fallidos manualmente, priorizar fix de flaky tests |
| R7 | Dependencias externas no disponibles para pruebas | 3 | 3 | 9 | Mocks/stubs para dependencias externas (WireMock), contract tests (Pact) | Deshabilitar funcionalidad dependiente vía feature flag |
| R8 | Equipo de pruebas insuficiente o con falta de skills | 2 | 4 | 8 | Cross-training, documentation, automation para reducir effort manual | Contratar QA temporal, priorizar automatización de pruebas críticas |

### 9.2 Risk-Based Testing Strategy

**Priorización de Pruebas Basada en Riesgo**:

| Risk Level | Criterios | Estrategia de Pruebas |
|------------|-----------|----------------------|
| **Alto** | Funcionalidad crítica de negocio (pagos, aprobaciones), alto tráfico, cambios complejos, integraciones con sistemas externos | Pruebas exhaustivas: unit, integration, E2E, performance, security, chaos. Cobertura ≥ 90%. |
| **Medio** | Funcionalidad importante pero no crítica, tráfico moderado, cambios de complejidad media | Pruebas estándar: unit, integration, E2E para flujos principales. Cobertura ≥ 80%. |
| **Bajo** | Funcionalidad secundaria, bajo tráfico, cambios simples, sin integraciones externas | Pruebas básicas: unit, smoke tests. Cobertura ≥ 70%. |

---

## ✅ Checklist de Go-Live

### 10.1 Pre-Requisitos de Calidad

#### Pruebas Funcionales
- [ ] Unit tests: Cobertura ≥ 80%, mutación ≥ 60%.
- [ ] Integration tests: 100% pasando.
- [ ] Contract tests: 100% pasando (Pact broker verde).
- [ ] E2E tests: 100% de flujos críticos pasando.
- [ ] Regression tests: 100% pasando (suite completa o selectiva según impacto).
- [ ] Smoke tests: 100% pasando (ejecutados post-deploy en STAGING).

#### Pruebas No Funcionales
- [ ] Performance tests: SLA cumplido (p99 < 500ms, error rate < 0.1%, throughput ≥ capacidad planificada).
- [ ] Security tests: 0 vulnerabilidades CRITICAL/HIGH (SAST, SCA, DAST).
- [ ] Penetration test: Aprobado sin hallazgos críticos.
- [ ] PCI-DSS validation: SAQ completado, TDE habilitado, PAN enmascarado, CVV no almacenado.
- [ ] Chaos tests: Experimentos críticos pasando (resiliencia validada).
- [ ] Disaster recovery: Failover probado (RTO < 30 min, RPO < 5 min).

#### UAT (User Acceptance Testing)
- [ ] Casos de prueba de negocio ejecutados y aprobados.
- [ ] Business owner sign-off obtenido.
- [ ] Feedback de usuarios incorporado (si aplica).

### 10.2 Gestión de Defectos

- [ ] 0 defectos CRITICAL abiertos.
- [ ] 0 defectos HIGH abiertos.
- [ ] Defectos MEDIUM/LOW con plan de mitigación aprobado (deferred a próximo release).
- [ ] Defect leakage < 5% (histórico de releases anteriores).

### 10.3 Documentación y Comunicación

- [ ] Test plan documentado y aprobado.
- [ ] Test cases documentados en Jira/Xray.
- [ ] Test execution report generado (métricas, resultados).
- [ ] Defect report generado (defectos abiertos, cerrados, deferred).
- [ ] Runbooks actualizados (procedimientos operativos, troubleshooting).
- [ ] API documentation actualizada (Swagger UI, Postman collections).
- [ ] Architecture diagrams actualizados (C4 model, sequence diagrams).
- [ ] Release notes preparadas (nuevas funcionalidades, known issues, migration steps).
- [ ] Communication plan ejecutado (stakeholders, usuarios, equipos de soporte notificados).

### 10.4 Despliegue y Rollback

- [ ] Deployment plan documentado (pasos, responsables, timeline).
- [ ] Rollback plan documentado y probado en STAGING.
- [ ] Rollback criteria definidos (si error rate > X% o latency > Y, ejecutar rollback).
- [ ] War room establecido (bridge call con arquitectos, devs, ops, security, business).
- [ ] CAB (Change Advisory Board) aprobó el release.
- [ ] Ventana de mantenimiento asignada.
- [ ] Feature flags configurados (para activar/desactivar funcionalidades sin redeploy).

### 10.5 Monitoreo y Soporte

- [ ] Dashboards de monitoreo configurados (Prometheus, Grafana, Splunk).
- [ ] Alertas configuradas (P1, P2, P3) con runbooks asociados.
- [ ] On-call rotation establecido (quién recibe paging, escalation policy).
- [ ] Hypercare plan definido (monitoreo intensivo por 48-72h post-deploy).
- [ ] Support team capacitado (conoce nuevas funcionalidades, known issues).

---

## 📚 Referencias y Frameworks

- **ISTQB Foundation Level**: International Software Testing Qualifications Board.
- **Agile Testing**: Lisa Crispin & Janet Gregory, "Agile Testing" (2009).
- **Continuous Delivery**: Jez Humble & David Farley, "Continuous Delivery" (2010).
- **Testing Microservices**: Angie Jones, "Testing Microservices" (2023).
- **Chaos Engineering**: Casey Rosenthal & Nora Jones, "Chaos Engineering" (2020).
- **Site Reliability Engineering**: Google SRE Book (2016).
- **PCI-DSS v4.0**: Payment Card Industry Data Security Standard.
- **OWASP Testing Guide**: Open Web Application Security Project.

---

**Última actualización:** 2025-02-05  
**Versión:** 1.0  
**Mantenido por:** QA Guild & Enterprise Architecture Office  
**Aprobado por:** CTO & Architecture Review Board