# Monolith to Microservices (M2S) Migration Checklist

## 📋 Propósito

Este checklist establece el estándar obligatorio para la migración de aplicaciones monolíticas a arquitecturas de microservicios en el contexto ERLN (retail masivo, 19,556 sucursales, misión crítica 24/7). Su objetivo es garantizar que cada migración se ejecute de manera **controlada, medible y reversible**, minimizando el riesgo operacional y asegurando el cumplimiento de **PCI-DSS** y los **SLAs de 99.99%**.

**Principios de Migración:**
- ✅ **Strangler Fig Pattern** (ADR-001): Migración incremental, nunca "big bang".
- ✅ **Branch by Abstraction**: Convivencia controlada entre monolito y microservicios.
- ✅ **API-First Design**: Contratos definidos antes de implementar.
- ✅ **Database per Service**: Cada microservicio es dueño de sus datos.
- ✅ **Zero Downtime**: Migración sin interrupción del servicio.
- ❌ **Prohibido**: Reescribir el monolito desde cero ("rewrite trap").
- ❌ **Prohibido**: Migrar sin rollback plan documentado y probado.
- ❌ **Prohibido**: Crear "microservicios distribuidos" (distributed monolith).

---

## 🎯 Fase 1: Assessment y Domain Decomposition

### 1.1 Análisis del Monolito Existente
- [ ] **Inventario de módulos** documentado (funcionalidad, dependencias, líneas de código).
- [ ] **Análisis de dependencias** realizado (herramienta: Structure101, jQAssistant, o SonarQube Dependency Matrix).
- [ ] **Identificación de "hotspots"** (código con alta complejidad ciclomática o deuda técnica).
- [ ] **Mapeo de integraciones externas** (APIs de terceros, sistemas legacy, batch jobs).
- [ ] **Análisis de tráfico y uso** (métricas de WebLogic/Splunk: qué módulos se usan más, picos de carga).
- [ ] **Identificación de datos sensibles** (PCI, PII) y su ubicación en el monolito.

### 1.2 Domain-Driven Design (DDD)
- [ ] **Event Storming** realizado con stakeholders de negocio y equipo técnico.
- [ ] **Bounded Contexts** identificados y delimitados (ej. Pagos, Inventario, Clientes, Promociones).
- [ ] **Ubiquitous Language** documentado para cada bounded context (glosario de términos).
- [ ] **Context Maps** creados (relaciones entre bounded contexts: Partnership, Customer-Supplier, Conformist).
- [ ] **Aggregates y Entities** definidos para cada contexto.
- [ ] **Domain Events** identificados (eventos de negocio que cruzan boundaries).

### 1.3 Criterios de Elegibilidad para Migración
- [ ] **Priorización de dominios** basada en:
  - Valor de negocio (impacto en revenue o customer experience).
  - Frecuencia de cambios (dominios con alta actividad de desarrollo).
  - Complejidad técnica (deuda técnica, riesgo de fallo).
  - Independencia de despliegue (posibilidad de liberar sin afectar otros dominios).
- [ ] **Matriz de decisión** completada (¿migrar ahora, refactorizar, o dejar en monolito?).
- [ ] **Aprobación del ARB** obtenida para el dominio seleccionado como "primer corte" (strangler slice).

**Go/No-Go Gate 1**: ✅ Domain decomposition aprobado por ARB y stakeholders de negocio.

---

## 🔄 Fase 2: Estrategia de Migración (Strangler Fig)

### 2.1 Definición del Patrón de Migración
- [ ] **Strangler Fig Pattern** seleccionado como estrategia oficial (ADR-001).
- [ ] **Secuencia de extracción** definida (qué funcionalidades se migran primero).
- [ ] **Branch by Abstraction** planificado (capa de abstracción para desacoplar monolito).
- [ ] **Anti-Corruption Layer (ACL)** diseñada (adaptadores para traducir entre monolito y microservicios).
- [ ] **Feature Flags** configurados (LaunchDarkly/Unleash) para controlar tráfico entre monolito y microservicio.

### 2.2 Plan de Convivencia
- [ ] **Ruta de migración** documentada (monolito → ACL → microservicio → decommission).
- [ ] **Dual-write strategy** definida (si aplica, escribir en ambos sistemas durante transición).
- [ ] **Data synchronization** planificada (CDC con Debezium/Oracle GoldenGate para mantener consistencia).
- [ ] **Fallback mechanism** diseñado (si el microservicio falla, regresar al monolito automáticamente).
- [ ] **Timeline de convivencia** definido (máximo 6 meses antes de decommission del módulo en monolito).

### 2.3 Gestión de Riesgos
- [ ] **Matriz de riesgos** actualizada con riesgos específicos de migración.
- [ ] **Rollback plan** documentado y probado en staging (cómo regresar al monolito en < 5 minutos).
- [ ] **Data rollback strategy** definida (cómo revertir cambios de datos si la migración falla).
- [ ] **Communication plan** creado (stakeholders, usuarios, equipos de soporte informados).

**Go/No-Go Gate 2**: ✅ Estrategia de migración aprobada, rollback plan probado en staging.

---

## 🏗️ Fase 3: Diseño del Microservicio

### 3.1 API Design (API-First)
- [ ] **OpenAPI 3.0 specification** creada y revisada por consumidores.
- [ ] **Contract testing** definido (Pact consumer-driven contracts).
- [ ] **Versioning strategy** definida (URI versioning `/v1/resource` o header versioning).
- [ ] **Idempotency** garantizada para operaciones POST/PUT (header `Idempotency-Key`).
- [ ] **HATEOAS** evaluado (si aplica, links embebidos para navegabilidad).
- [ ] **API Guidelines** revisadas y cumplidas (ver `api-design-guidelines.md`).

### 3.2 Data Ownership
- [ ] **Database per Service** diseñado (nueva base de datos Oracle 12c o schema aislado).
- [ ] **Data migration plan** creado (extracción desde monolito, transformación, carga).
- [ ] **Saga pattern** definido para transacciones distribuidas (orquestación vs coreografía).
- [ ] **Event sourcing** evaluado (si aplica para audit trail o replay capability).
- [ ] **CQRS** evaluado (si hay asimetría lectura/escritura).

### 3.3 Resiliencia y Patrones
- [ ] **Circuit Breaker** configurado con **Resilience4j** (ADR-003).
  - [ ] Failure rate threshold definido (ej. 50%).
  - [ ] Slow call rate threshold definido (ej. 100 calls > 5s).
  - [ ] Wait duration in open state configurado (ej. 30s).
  - [ ] Fallback method implementado (degradación elegante).
- [ ] **Retry pattern** configurado (exponential backoff con jitter).
- [ ] **Bulkhead pattern** evaluado (aislamiento de recursos para servicios críticos).
- [ ] **Timeout configurado** (conect timeout 2s, read timeout 5s).
- [ ] **Rate Limiting** definido (cuotas por consumidor vía Kong Enterprise).

### 3.4 Cross-Team Alignment
- [ ] **Data Mesh principles** aplicados (si aplica, dominio como producto).
- [ ] **SLA definido** con consumidores (disponibilidad, latencia, throughput).
- [ ] **On-call rotation** establecido (quién soporta el microservicio 24/7).
- [ ] **Documentation portal** actualizado (Swagger UI, runbooks, architecture diagrams).
- [ ] **Cross-team alignment framework** aplicado (ADR-004: guilds, chapters, sync meetings).

**Go/No-Go Gate 3**: ✅ Diseño aprobado en ARB, contratos firmados con consumidores.

---

## 💻 Fase 4: Implementación (Java 8 + Spring Boot)

### 4.1 Stack Tecnológico
- [ ] **Java 8** como versión base (no Java 11+ por compatibilidad con WebLogic 12c).
- [ ] **Spring Boot 2.x** (última versión compatible con Java 8).
- [ ] **Spring Data JPA** para acceso a datos (Oracle 12c).
- [ ] **Spring Cloud** (si aplica, para service discovery, config server).
- [ ] **Resilience4j** integrado (circuit breaker, retry, rate limiter, bulkhead).
- [ ] **Lombok** para reducir boilerplate (con cuidado en equals/hashCode).
- [ ] **MapStruct** para mapeo de DTOs (no BeanCopier por type-safety).

### 4.2 Calidad de Código
- [ ] **SonarQube Quality Gate** configurado (0 vulnerabilities, 0 bugs, 80% cobertura).
- [ ] **Checkstyle/SpotBugs** integrados en pipeline (estilo de código corporativo).
- [ ] **Architecture tests** escritos (ArchUnit para validar dependencias entre capas).
- [ ] **Unit tests** con JUnit 5 + Mockito (cobertura ≥ 80%).
- [ ] **Integration tests** con Testcontainers (Oracle 12c embebido, Redis, RabbitMQ).
- [ ] **Contract tests** con Pact (consumer-driven).

### 4.3 Configuración y Secretos
- [ ] **Externalized configuration** (application.yml en ConfigMap de Kubernetes).
- [ ] **Secretos en Vault** (HashiCorp Vault, no en application.yml).
- [ ] **Profile-based config** (dev, qa, staging, prod).
- [ ] **Feature flags** integrados (LaunchDarkly SDK o Unleash).
- [ ] **Health checks** implementados (`/actuator/health`, `/actuator/info`).

### 4.4 Logging y Tracing
- [ ] **Structured logging** (JSON format con Logback/Log4j2).
- [ ] **Correlation ID** propagado (MDC con `X-Correlation-ID`).
- [ ] **OpenTelemetry** integrado (traces, metrics, logs).
- [ ] **Sensitive data masking** en logs (PAN, PII, passwords).
- [ ] **Log levels** configurados por ambiente (INFO en prod, DEBUG en dev).

**Go/No-Go Gate 4**: ✅ Código revisado, SonarQube verde, tests pasando.

---

## 🗄️ Fase 5: Data Migration (Oracle 12c)

### 5.1 Estrategia de Migración de Datos
- [ ] **Data mapping documentado** (tabla origen en monolito → tabla destino en microservicio).
- [ ] **Data transformation rules** definidas (limpieza, enriquecimiento, normalización).
- [ ] **Data validation rules** creadas (checksums, row counts, business rules).
- [ ] **Migration window** definida (ventana de mantenimiento aprobada por CAB).
- [ ] **Rollback plan para datos** documentado (backup antes de migrar, script de reversa).

### 5.2 Técnicas de Migración
- [ ] **Bulk load** para datos históricos (Oracle SQL*Loader, Data Pump).
- [ ] **CDC (Change Data Capture)** para sincronización incremental (Debezium + Kafka, Oracle GoldenGate).
- [ ] **Dual-write pattern** implementado (si aplica, escribir en monolito y microservicio simultáneamente).
- [ ] **Shadow traffic** configurado (enviar tráfico de lectura al microservicio para validar sin afectar usuarios).
- [ ] **Data freeze window** planificado (período sin cambios durante cutover final).

### 5.3 Validación y Consistencia
- [ ] **Data reconciliation scripts** ejecutados (comparar monolito vs microservicio).
- [ ] **Checksum validation** realizado (hash de filas críticas).
- [ ] **Business rules validation** ejecutado (ej. saldos de cuentas, inventarios).
- [ ] **Performance testing** de queries (índices creados, execution plans revisados).
- [ ] **Data retention policy** definida (cuánto tiempo mantener datos en monolito después de migrar).

### 5.4 Seguridad de Datos
- [ ] **TDE (Transparent Data Encryption)** habilitado en Oracle 12c para datos PCI/PII.
- [ ] **Data masking** aplicado en ambientes no productivos (Oracle Data Redaction).
- [ ] **Access control** configurado (roles Oracle con least privilege).
- [ ] **Audit trail** activado (Oracle Audit Vault para rastrear accesos a datos sensibles).

**Go/No-Go Gate 5**: ✅ Datos migrados y validados, reconciliación exitosa, rollback probado.

---

## 🌐 Fase 6: API Gateway y Routing (Kong Enterprise)

### 6.1 Configuración de Kong Enterprise
- [ ] **Route creado** en Kong para el nuevo microservicio (path, methods, hosts).
- [ ] **Service registrado** en Kong (upstream URL, health checks).
- [ ] **Plugins de seguridad** habilitados (ADR-002):
  - [ ] `jwt` o `oauth2` para autenticación.
  - [ ] `rate-limiting` configurado (quotas por consumer).
  - [ ] `ip-restriction` para APIs administrativas.
  - [ ] `cors` configurado restrictivamente.
  - [ ] `bot-detection` o WAF integration.
- [ ] **Plugins de transformación** configurados (request/response transformer si cambia formato).
- [ ] **Plugins de observabilidad** habilitados (prometheus, zipkin, logging).

### 6.2 Traffic Management
- [ ] **Canary release** configurado (5% → 25% → 50% → 100% del tráfico al microservicio).
- [ ] **A/B testing** habilitado (si aplica, para validar nueva implementación con usuarios reales).
- [ ] **Traffic mirroring** configurado (shadow traffic para validar sin afectar producción).
- [ ] **Fallback routing** definido (si el microservicio falla, Kong rutea al monolito automáticamente).
- [ ] **Circuit breaker en Kong** configurado (si el microservicio está caído, no saturar con retries).

### 6.3 Testing de Integración
- [ ] **End-to-end tests** ejecutados a través de Kong (simulando tráfico real).
- [ ] **Load testing** realizado (JMeter/Gatling) para validar que Kong no es bottleneck.
- [ ] **Chaos testing** ejecutado (matar pods del microservicio, validar que Kong hace fallback).
- [ ] **Security testing** realizado (pen-test a través de Kong, validar que plugins bloquean ataques).

**Go/No-Go Gate 6**: ✅ Kong configurado, traffic management probado, fallback funcionando.

---

## 🧪 Fase 7: Testing Integral

### 7.1 Testing Pyramid
- [ ] **Unit tests** ≥ 80% cobertura (JUnit 5 + Mockito).
- [ ] **Integration tests** con Testcontainers (Spring Boot Test + Oracle 12c embebido).
- [ ] **Contract tests** con Pact (consumer-driven, ejecutados en pipeline).
- [ ] **Component tests** (probar microservicio aislado con dependencias mockeadas).
- [ ] **End-to-end tests** (Selenium/RestAssured para flujos críticos de negocio).

### 7.2 Performance Testing
- [ ] **Baseline establecido** (métricas del monolito: latency p99, throughput, error rate).
- [ ] **Load testing** ejecutado (JMeter/Gatling) con escenarios de pico (ej. Black Friday).
- [ ] **Stress testing** realizado (llevar el sistema al límite, validar graceful degradation).
- [ ] **Soak testing** ejecutado (carga sostenida por 24h, detectar memory leaks).
- [ ] **Spike testing** realizado (aumentos súbitos de carga, validar auto-scaling).
- [ ] **Performance regression** validado (nuevo microservicio no es > 10% más lento que monolito).

### 7.3 Chaos Engineering
- [ ] **Chaos experiments** definidos (Gremlin/Litmus):
  - [ ] Matar pods del microservicio (validar circuit breaker y fallback).
  - [ ] Introducir latencia de red (validar timeouts y retries).
  - [ ] Saturar CPU/memoria (validar auto-scaling y bulkheads).
  - [ ] Cortar conexión a base de datos (validar graceful degradation).
  - [ ] Corromper respuestas de dependencias (validar manejo de errores).
- [ ] **GameDay** realizado (simulacro de incidente con equipo de operaciones).
- [ ] **Post-mortem** documentado (lecciones aprendidas, action items).

### 7.4 Security Testing
- [ ] **SAST** ejecutado (Checkmarx/SonarQube Security, 0 vulnerabilities CRITICAL/HIGH).
- [ ] **SCA** ejecutado (Snyk/OWASP Dependency-Check, 0 vulnerabilities CRITICAL/HIGH).
- [ ] **DAST** ejecutado (OWASP ZAP contra endpoints expuestos).
- [ ] **Penetration testing** realizado por equipo de seguridad externo.
- [ ] **PCI-DSS validation** completado (si el microservicio procesa datos de tarjeta).

**Go/No-Go Gate 7**: ✅ Todos los tests pasando, performance baseline cumplido, security scan limpio.

---

## 📊 Fase 8: Observability (Splunk + Prometheus)

### 8.1 Métricas (Prometheus + Grafana)
- [ ] **RED metrics** expuestas (Rate, Errors, Duration p50/p95/p99).
- [ ] **JVM metrics** expuestas (heap, GC, threads, class loading).
- [ ] **Business metrics** expuestas (transacciones/segundo, usuarios activos, etc.).
- [ ] **Resilience4j metrics** expuestas (circuit breaker state, retry count, bulkhead availability).
- [ ] **Custom metrics** definidas (KPIs específicos del dominio).
- [ ] **Grafana dashboards** creados:
  - [ ] Overview del microservicio (SLA, error rate, latency).
  - [ ] Drill-down por endpoint (performance por API).
  - [ ] JVM health (memory, GC, threads).
  - [ ] Business metrics (transacciones, revenue).

### 8.2 Logs (Splunk)
- [ ] **Structured logging** en JSON (campos: timestamp, level, service, traceId, spanId, message).
- [ ] **Correlation ID** propagado en todos los logs (trazabilidad end-to-end).
- [ ] **Sensitive data masking** validado (PAN, PII, passwords enmascarados).
- [ ] **Log aggregation** configurado (Fluentd/Filebeat → Splunk).
- [ ] **Splunk dashboards** creados:
  - [ ] Error analysis (top errors, error rate por endpoint).
  - [ ] Security events (intentos de acceso fallidos, anomalías).
  - [ ] Audit trail (quién hizo qué, cuándo).

### 8.3 Traces (OpenTelemetry → Splunk/Jaeger)
- [ ] **Distributed tracing** habilitado (OpenTelemetry SDK).
- [ ] **Trace propagation** configurado (W3C Trace Context, headers `traceparent`, `tracestate`).
- [ ] **Sampling strategy** definida (100% en QA, 10% en Prod, 100% para errores).
- [ ] **Span instrumentation** agregado (HTTP calls, DB queries, message broker).
- [ ] **Trace visualization** configurada (Jaeger UI o Splunk APM).

### 8.4 Alertas (PagerDuty/ServiceNow)
- [ ] **P1 alerts** configuradas (críticas, paging 24/7):
  - [ ] Error rate > 1% en 5 minutos.
  - [ ] p99 latency > 2s en 5 minutos.
  - [ ] Circuit breaker abierto.
  - [ ] Health check fallando.
- [ ] **P2 alerts** configuradas (altas, business hours):
  - [ ] Error rate > 0.5% en 15 minutos.
  - [ ] p95 latency > 1s en 15 minutos.
  - [ ] Memory usage > 80%.
- [ ] **Runbooks** documentados para cada alerta (cómo diagnosticar, cómo mitigar).
- [ ] **On-call rotation** configurado (quién recibe paging, escalation policy).

**Go/No-Go Gate 8**: ✅ Dashboards creados, alertas configuradas, runbooks documentados.

---

## 🔒 Fase 9: Security & Compliance (PCI-DSS, Zero-Trust)

### 9.1 Identidad y Acceso
- [ ] **OAuth 2.0 / OIDC** integrado (Entra ID o proveedor corporativo).
- [ ] **JWT validation** configurado (validar signature, expiration, issuer, audience).
- [ ] **RBAC/ABAC** implementado (roles y permisos definidos, least privilege).
- [ ] **MFA** habilitado para accesos administrativos.
- [ ] **Service-to-service auth** configurado (mTLS o client credentials flow).

### 9.2 Protección de Datos
- [ ] **TLS 1.2+** habilitado en todos los endpoints (cipher suites fuertes).
- [ ] **TDE** activado en Oracle 12c para datos PCI/PII.
- [ ] **Tokenización** implementada para PAN (reemplazar número de tarjeta con token).
- [ ] **Data masking** en logs y respuestas de API (PAN enmascarado: `**** **** **** 1234`).
- [ ] **CVV/CVC** NUNCA almacenado (validado en code review y pen-test).

### 9.3 API Security
- [ ] **Kong Enterprise plugins** habilitados (auth, rate limiting, WAF, bot detection).
- [ ] **Input validation** implementado (Bean Validation, reject malicious payloads).
- [ ] **Output encoding** configurado (prevenir XSS).
- [ ] **SQL injection prevention** validado (prepared statements, no string concatenation).
- [ ] **Security headers** inyectados por Kong (HSTS, X-Frame-Options, CSP).

### 9.4 Compliance
- [ ] **PCI-DSS SAQ** completado (Self-Assessment Questionnaire para el microservicio).
- [ ] **Audit trail** activado (Oracle Audit Vault, logs de auditoría inmutables).
- [ ] **Data residency** validado (datos de tarjeta permanecen en México).
- [ ] **Retention policy** definida (logs retenidos 1 año, datos PCI según política).
- [ ] **Security review** aprobada por CISO (revisión de arquitectura, code, infra).

**Go/No-Go Gate 9**: ✅ Security review aprobada, PCI-DSS validado, Zero-Trust implementado.

---

## 🚀 Fase 10: Cutover y Go-Live

### 10.1 Pre-Requisitos
- [ ] **Todos los Go/No-Go Gates** aprobados (Fases 1-9).
- [ ] **Change Advisory Board (CAB)** aprobó el cambio (ventana de mantenimiento asignada).
- [ ] **Communication plan** ejecutado (stakeholders, usuarios, equipos de soporte notificados).
- [ ] **War room** establecido (bridge call con arquitectos, devs, ops, security, business).
- [ ] **Rollback plan** revisado y listo para ejecutar (documentado, probado, asignaciones claras).

### 10.2 Ejecución del Cutover
- [ ] **Data freeze** iniciado (detener cambios en monolito para el módulo migrado).
- [ ] **Final data sync** ejecutado (CDC catch-up, validar reconciliación).
- [ ] **Smoke tests** ejecutados contra microservicio en producción (validar health checks).
- [ ] **Canary release** iniciado (5% del tráfico al microservicio, monitorear 15 min).
- [ ] **Gradual traffic shift** ejecutado (25% → 50% → 100%, monitorear métricas en cada paso).
- [ ] **Fallback testing** realizado (forzar fallo del microservicio, validar que Kong rutea al monolito).
- [ ] **Business validation** completado (usuarios clave validan flujos críticos en producción).

### 10.3 Post-Cutover
- [ ] **Hypercare period** iniciado (monitoreo intensivo por 48-72h).
- [ ] **War room** activo (bridge call permanente durante hypercare).
- [ ] **Daily standups** realizados (revisar métricas, incidentes, feedback de usuarios).
- [ ] **Rollback decision criteria** definidos (si error rate > X% o latency > Y, ejecutar rollback).
- [ ] **Success criteria** validados (SLA cumplido, business KPIs estables, sin incidentes P1/P2).

### 10.4 Rollback (si aplica)
- [ ] **Rollback decision** tomada (si criteria cumplido, ejecutar rollback inmediatamente).
- [ ] **Traffic shift** revertido (Kong rutea 100% al monolito).
- [ ] **Data rollback** ejecutado (si aplica, revertir cambios de datos).
- [ ] **Post-mortem** agendado (dentro de 48h, blameless, action items claros).
- [ ] **Stakeholders comunicados** (incident report enviado, next steps definidos).

**Go/No-Go Gate 10**: ✅ Cutover exitoso, hypercare completado, success criteria cumplidos.

---

## 🗑️ Fase 11: Decommissioning del Monolito

### 11.1 Validación Post-Migración
- [ ] **Stability period** completado (mínimo 30 días sin incidentes relacionados al módulo migrado).
- [ ] **Data consistency** validada (monolito y microservicio muestran mismos datos).
- [ ] **Performance baseline** cumplido (microservicio igual o mejor que monolito).
- [ ] **User feedback** positivo (sin quejas de usuarios sobre funcionalidad migrada).
- [ ] **Business sign-off** obtenido (dueño de negocio confirma que funcionalidad opera correctamente).

### 11.2 Decommissioning del Código
- [ ] **Código del módulo** eliminado del monolito (branch by abstraction removida).
- [ ] **Dead code** limpiado (imports, clases, métodos que ya no se usan).
- [ ] **Tests del módulo** eliminados del monolito (migrados al microservicio).
- [ ] **Documentation actualizada** (diagramas de arquitectura, runbooks, API docs).
- [ ] **Code review** realizado (validar que no quedaron referencias al módulo migrado).

### 11.3 Decommissioning de Datos
- [ ] **Backup final** de tablas del monolito (retener según política, ej. 1 año).
- [ ] **Tablas eliminadas** del monolito (después de backup y validación).
- [ ] **Foreign keys** removidas (si había referencias desde otras tablas del monolito).
- [ ] **Stored procedures/functions** eliminadas (si eran exclusivas del módulo migrado).
- [ ] **Data dictionary actualizado** (reflejar nueva estructura de datos).

### 11.4 Decommissioning de Infraestructura
- [ ] **WebLogic cluster** redimensionado (si el monolito consume menos recursos).
- [ ] **Oracle 12c schemas** limpiados (usuarios, roles, tablespaces que ya no se usan).
- [ ] **Monitoring alerts** actualizadas (remover alertas del módulo migrado en monolito).
- [ ] **Cost allocation** actualizado (chargeback al equipo dueño del microservicio).

### 11.5 Celebración y Retrospectiva
- [ ] **Retrospectiva** realizada (qué salió bien, qué salió mal, lecciones aprendidas).
- [ ] **Knowledge sharing** session (presentar experiencia a otros equipos que harán migración).
- [ ] **Documentation actualizada** (playbook de migración con lecciones aprendidas).
- [ ] **Celebración del equipo** (reconocer esfuerzo, team building, comida).

**Go/No-Go Gate 11**: ✅ Monolito decommissionado, equipo celebró, lecciones aprendidas documentadas.

---

## ✅ Definition of Done (DoD) por Fase

| Fase | DoD Criteria | Aprobador |
|------|--------------|-----------|
| **Fase 1** | Domain decomposition aprobado, bounded contexts definidos | ARB + Business Owner |
| **Fase 2** | Strangler Fig plan aprobado, rollback plan probado | ARB + Tech Lead |
| **Fase 3** | Diseño aprobado, contratos firmados con consumidores | ARB + Consumer Teams |
| **Fase 4** | Código implementado, SonarQube verde, tests pasando | Tech Lead + QA |
| **Fase 5** | Datos migrados y validados, reconciliación exitosa | DBA + Data Owner |
| **Fase 6** | Kong configurado, traffic management probado | Platform Team + Security |
| **Fase 7** | Todos los tests pasando, performance baseline cumplido | QA + Performance Team |
| **Fase 8** | Dashboards creados, alertas configuradas, runbooks documentados | SRE + Ops Team |
| **Fase 9** | Security review aprobada, PCI-DSS validado | CISO + Security Team |
| **Fase 10** | Cutover exitoso, hypercare completado | CAB + Business Owner |
| **Fase 11** | Monolito decommissionado, lecciones aprendidas documentadas | ARB + Tech Lead |

---

## 🚫 Anti-Patterns a Evitar

### 1. Distributed Monolith
- **Síntoma**: Microservicios que no se pueden desplegar independientemente, dependencias circulares.
- **Prevención**: Validar independencia de despliegue en Fase 3, contract tests estrictos.

### 2. Big Bang Migration
- **Síntoma**: Intentar migrar todo el monolito de una vez, proyectos de 2+ años.
- **Prevención**: Strangler Fig obligatorio, migrar un bounded context a la vez.

### 3. Shared Database
- **Síntoma**: Múltiples microservicios accediendo a la misma base de datos.
- **Prevención**: Database per Service, validar en Fase 3 y code reviews.

### 4. Nano-services
- **Síntoma**: Microservicios demasiado pequeños (1-2 endpoints), overhead operacional alto.
- **Prevención**: Right-sizing en Fase 1, validar que cada servicio tiene cohesión alta.

### 5. Ignoring Organizational Structure
- **Síntoma**: Crear microservicios que no alinean con estructura de equipos (Conway's Law).
- **Prevención**: Cross-team alignment framework (ADR-004), cada equipo dueño de sus servicios.

### 6. No Rollback Plan
- **Síntoma**: Migrar sin plan de reversa, si falla, quedarse atorado.
- **Prevención**: Rollback plan obligatorio en Fase 2, probado en staging.

### 7. Premature Optimization
- **Síntoma**: Sobre-diseñar, agregar complejidad innecesaria (event sourcing, CQRS) sin necesidad.
- **Prevención**: YAGNI (You Aren't Gonna Need It), empezar simple, evolucionar después.

### 8. Ignoring Data Migration
- **Síntoma**: Subestimar complejidad de migrar datos, descubrir problemas en cutover.
- **Prevención**: Data migration plan detallado en Fase 5, reconciliación exhaustiva.

### 9. No Observability
- **Síntoma**: Desplegar microservicio sin métricas, logs, traces, alertas.
- **Prevención**: Observability checklist en Fase 8, Go/No-Go gate obligatorio.

### 10. Skipping Security
- **Síntoma**: Migrar rápido, dejar security para "después", descubrir vulnerabilidades en prod.
- **Prevención**: Security-by-design en Fase 9, PCI-DSS validado antes de go-live.

---

## 📚 Referencias y Frameworks

- **Strangler Fig Pattern**: Martin Fowler, "Strangler Fig Application" (2004).
- **Domain-Driven Design**: Eric Evans, "Domain-Driven Design: Tackling Complexity in the Heart of Software" (2003).
- **Building Microservices**: Sam Newman, "Building Microservices" (2nd Edition, 2021).
- **Monolith to Microservices**: Sam Newman, "Monolith to Microservices" (2019).
- **Microservices Patterns**: Chris Richardson, "Microservices Patterns" (2018).
- **Production-Ready Microservices**: Susan Fowler, "Production-Ready Microservices" (2016).
- **Release It!**: Michael Nygard, "Release It!: Design and Deploy Production-Ready Software" (2nd Edition, 2018).
- **Team Topologies**: Matthew Skelton & Manuel Pais, "Team Topologies" (2019).

---

**Última actualización:** 2025-01-30  
**Versión:** 1.0  
**Mantenido por:** Enterprise Architecture Office  
**Aprobado por:** CTO & Architecture Review Board