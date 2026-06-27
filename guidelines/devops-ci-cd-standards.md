# DevOps & CI/CD Standards

## 📋 Propósito

Este documento establece los estándares obligatorios de **Integración Continua, Entrega Continua y Operaciones (DevOps)** para todos los microservicios y aplicaciones enterprise del caso ERLN. Su objetivo es garantizar despliegues seguros, repetibles, auditables y con rollback automático, alineados con los SLAs de **99.99%** y los requerimientos de cumplimiento **PCI-DSS** y **SOX**.

**Objetivos de Negocio:**
- Reducir el **MTTR (Mean Time to Recovery)** a < 15 minutos mediante rollbacks automatizados.
- Acelerar el **Time-to-Market** de nuevas funcionalidades de semanas a días.
- Garantizar **trazabilidad completa** (quién, qué, cuándo, por qué) para auditorías PCI-DSS y SOX.
- Estandarizar la promoción entre ambientes para eliminar el "drift de configuración".

---

## 🌳 Estrategia de Ramificación (Git Branching)

### Modelo: GitFlow Enterprise (Modificado)

Dado el contexto de misión crítica y la necesidad de releases coordinados con el ARB, se adopta **GitFlow modificado** en lugar de Trunk-Based Development.

| Rama | Propósito | Protección | Merge Target |
|------|-----------|------------|--------------|
| `main` | Código en producción (siempre desplegable) | Branch protection, requiere 2 approvers + CI verde | - |
| `release/*` | Preparación de release (hardening, UAT) | Solo hotfixes y bug fixes críticos | `main` + `develop` |
| `develop` | Integración de features para el próximo release | CI obligatorio, SonarQube gate | `release/*` |
| `feature/*` | Desarrollo de nuevas funcionalidades | CI básico | `develop` |
| `hotfix/*` | Parches críticos a producción | Aprobación de Change Advisory Board (CAB) | `main` + `develop` |

### Reglas de Protección de Ramas
- ✅ **Branch Protection Rules** habilitadas en `main` y `develop`.
- ✅ **Pull Requests obligatorios** (nunca push directo).
- ✅ **Mínimo 2 approvers**: 1 Tech Lead + 1 Security Champion (para cambios en módulos PCI).
- ✅ **CI Pipeline verde** como condición obligatoria para merge.
- ✅ **Linear history** requerida (squash & merge) para facilitar auditoría.
- ❌ **Prohibido:** Forzar push a ramas protegidas, incluso para administradores.

---

## 🔄 CI/CD Pipeline Estándar

### Herramientas del Stack
- **CI/CD Orchestrator:** Jenkins (enterprise license) o GitLab CI (self-hosted).
- **Code Repository:** Bitbucket Server / GitLab Self-Managed.
- **Artifact Repository:** JFrog Artifactory (para JARs, Docker images, Helm charts).
- **Container Registry:** Harbor (enterprise) o ACR/GCR si aplica cloud híbrido.
- **Secrets Management:** HashiCorp Vault integrado al pipeline.
- **IaC:** Terraform + Ansible.

### Fases del Pipeline (Stage-Gate Model)

```
[Commit] → [Build] → [Test] → [Security] → [Package] → [Deploy-Dev] → [Deploy-QA] → [Deploy-Staging] → [Deploy-Prod]
```

### Detalle de Stages

#### 1. Build & Compile
- **Java 8** con Maven/Gradle wrapper (versión fijada en el repo).
- Compilación con flags de seguridad: `-Xlint:all -Werror`.
- Generación de JAR/WAR ejecutable (Spring Boot fat-jar o WAR para WebLogic).

#### 2. Unit & Integration Tests
- **Cobertura mínima:** 80% de cobertura de código (JaCoCo).
- **Mutación testing:** PIT para validar calidad de tests (mínimo 60% mutation score).
- **Testcontainers** para pruebas de integración con Oracle 12c embebido.
- Tiempo máximo de ejecución: 15 minutos (fail-fast).

#### 3. Static Analysis & Quality Gates (SonarQube)
- **SonarQube Enterprise** con Quality Profile corporativo.
- **Quality Gate obligatorio:**
  - 0 Vulnerabilidades
  - 0 Bugs
  - 0 Code Smells críticos/bloqueantes
  - Cobertura ≥ 80%
  - Duplicación < 3%
  - Debt Ratio < 5%
- ❌ **Bloqueo automático** del pipeline si el Quality Gate falla.

#### 4. Security Scanning (Shift-Left)
- **SAST (Static Application Security Testing):** Checkmarx / SonarQube Security.
- **SCA (Software Composition Analysis):** Snyk / OWASP Dependency-Check.
  - Bloqueo si hay vulnerabilidades **CRITICAL** o **HIGH** sin excepción aprobada por CISO.
- **Secrets Detection:** GitLeaks / TruffleHog para detectar credenciales en el código.
- **Container Image Scanning:** Trivy / Snyk Container sobre la imagen Docker.

#### 5. Package & Sign
- Construcción de imagen Docker con **distroless base** o **Red Hat UBI** (no Alpine por FIPS compliance).
- **Firma de imagen** con Cosign / Notary para garantizar integridad.
- Push a Harbor/Artifactory con tag inmutable (SHA del commit, nunca `latest` en prod).
- Generación de **Helm Chart** versionado y empaquetado.

#### 6. Deploy a Ambientes
- Cada ambiente tiene su propio **values.yaml** de Helm.
- Despliegue vía **ArgoCD** (GitOps) o Jenkins con Helm upgrade.
- **Health checks** post-deploy obligatorios (readiness + liveness).
- **Smoke tests** automatizados contra endpoints críticos.

---

## 🏗️ Estrategias de Despliegue

### Producción: Blue-Green Deployment (Obligatorio para servicios críticos)

| Característica | Implementación |
|----------------|----------------|
| **Ambientes paralelos** | Dos sets idénticos de pods (Blue = activo, Green = nuevo) |
| **Switch de tráfico** | Kong Enterprise re-enrutamiento vía service mesh o DNS |
| **Rollback** | Instantáneo (< 30 seg) re-enrutando a Blue |
| **Costo** | 2x infraestructura durante el despliegue (justificado por SLA 99.99%) |

### Staging/QA: Canary Deployment
- Liberación progresiva: 5% → 25% → 50% → 100% del tráfico.
- **Prometheus + Grafana** monitorean error rate y latency durante cada fase.
- **Auto-rollback** si error rate > 1% o p99 latency > SLA definido.

### Reglas de Despliegue
- ✅ **Despliegues en ventana de cambio aprobada** (CAB) para servicios PCI.
- ✅ **Feature Flags** (LaunchDarkly / Unleash) para desacoplar deploy de release.
- ✅ **Database migrations** ejecutadas con **Liquibase** en modo forward-compatible (nunca romper backwards compatibility).
- ❌ **Prohibido:** Despliegues en viernes por la tarde o días festivos (salvo hotfix P1 con CAB).
- ❌ **Prohibido:** Despliegues manuales en producción (todo vía pipeline).

---

## 🗄️ Gestión de Configuración y Secretos

### Principio: Configuration as Code
- Toda configuración no sensible en **ConfigMaps** de Kubernetes versionados en Git.
- Diferenciación por ambiente vía **Kustomize overlays** o **Helm values**.

### Secretos (HashiCorp Vault)
- **Inyección vía sidecar** (Vault Agent) o **CSI Driver**.
- **Nunca** en variables de entorno de manifiestos K8s.
- **Rotación automática** cada 90 días para credenciales de BD.
- **Dynamic Secrets** de Oracle DB: credenciales de un solo uso con TTL de 24h.

### Variables de Entorno Críticas
| Variable | Origen | Rotación |
|----------|--------|----------|
| DB credentials | Vault (Dynamic Secrets) | Automática (24h TTL) |
| API Keys (Kong, terceros) | Vault (KV v2) | 90 días |
| Certificados TLS | Vault PKI / cert-manager | 90 días |
| Feature Flags | LaunchDarkly / ConfigMap | On-demand |

---

## 📊 Observabilidad Integrada al Pipeline

### Telemetría Obligatoria por Servicio
Cada microservicio debe exponer al desplegar:

1. **Métricas (Prometheus):**
   - RED metrics: Rate, Errors, Duration (p50, p95, p99).
   - JVM metrics: heap, GC, threads.
   - Custom business metrics (ej. transacciones/segundo).

2. **Logs (Splunk):**
   - Formato JSON estructurado.
   - Campos obligatorios: `traceId`, `spanId`, `service`, `level`, `timestamp`, `userId` (anonimizado).
   - **Correlation ID** propagado vía headers HTTP (`X-Correlation-ID`).

3. **Traces (OpenTelemetry → Splunk/Jaeger):**
   - Sampling rate: 100% en QA, 10% en Prod (ajustable).
   - Propagación W3C Trace Context.

### SLOs por Ambiente
| Métrica | Dev | QA | Staging | Prod (Crítico) |
|---------|-----|-----|---------|----------------|
| Availability | Best effort | 99% | 99.9% | **99.99%** |
| p99 Latency | N/A | < 2s | < 1s | < 500ms |
| Error Rate | N/A | < 5% | < 1% | < 0.1% |

---

## 🔒 Cumplimiento y Gobernanza (PCI-DSS / SOX)

### Segregación de Duties (SoD)
- El desarrollador que escribe el código **NO puede** desplegar a producción.
- El pipeline es ejecutado por una **service account** con permisos mínimos.
- Aprobación de cambio en ServiceNow requerida para deploy a Prod.

### Auditoría y Trazabilidad
- Cada despliegue genera un **Deployment Record** con:
  - Commit SHA, autor, PR link.
  - Resultados de SonarQube, SAST, SCA.
  - Aprobaciones (CAB, Security).
  - Timestamp y ejecutor (service account).
- Retención de logs de pipeline: **1 año mínimo** (3 años para PCI).

### Change Advisory Board (CAB)
- Reunión semanal para aprobar cambios a producción.
- Requerimientos para aprobación:
  - ✅ Plan de rollback documentado.
  - ✅ Pruebas de regresión ejecutadas.
  - ✅ Impacto en SLA evaluado.
  - ✅ Ventana de mantenimiento acordada.
  - ✅ Comunicación a stakeholders.

---

## 🧪 Estrategia de Pruebas (Testing Pyramid)

| Nivel | Herramienta | Cobertura Mínima | Ejecución |
|-------|-------------|------------------|-----------|
| **Unit Tests** | JUnit 5 + Mockito | 80% código | Cada commit |
| **Integration Tests** | Testcontainers + Spring Boot Test | Módulos críticos | Cada PR |
| **Contract Tests** | Pact (consumer-driven) | APIs expuestas | Cada PR |
| **E2E / Smoke Tests** | Selenium / RestAssured | Flujos críticos de negocio | Pre-prod |
| **Performance Tests** | JMeter / Gatling | Escenarios pico | Pre-release |
| **Chaos Engineering** | Gremlin / Litmus | Servicios críticos | Trimestral en staging |

### Reglas de Pruebas
- ✅ **Contract tests** obligatorios antes de romper APIs públicas.
- ✅ **Performance baseline** comparado contra release anterior (regresión < 5%).
- ❌ **Prohibido:** Saltar pruebas de integración "porque ya pasaron en local".
- ❌ **Prohibido:** Datos reales de producción en ambientes de QA (usar data masking).

---

## 🏛️ Infrastructure as Code (IaC)

### Herramientas
- **Terraform:** Para provisión de infraestructura (OCI, on-prem, redes).
- **Ansible:** Para configuración de servidores y WebLogic clusters.
- **Helm:** Para despliegue de aplicaciones en Kubernetes.
- **Crossplane** (opcional): Para control plano de infraestructura vía K8s API.

### Principios de IaC
- ✅ **Todo** infraestructura versionada en Git (no clicks en consola).
- ✅ **State files** en backend remoto (S3 + DynamoDB locking / Terraform Cloud).
- ✅ **Plan review** obligatorio antes de `apply` (PR approval).
- ✅ **Módulos reutilizables** aprobados por el equipo de plataforma.
- ❌ **Prohibido:** Cambios manuales en producción ("cowboy admin").

---

## 🚨 Gestión de Incidentes y Rollback

### Criterios de Rollback Automático
- Error rate > 1% en los primeros 5 minutos post-deploy.
- p99 latency > 2x del baseline histórico.
- Health check falla consecutivamente (3 intentos).
- Alerta P1 disparada en Splunk/PagerDuty.

### Procedimiento de Rollback
1. **Detección automática** (ArgoCD rollback / Kong traffic shift).
2. **Notificación** a canal de Slack `#incidents` y on-call.
3. **Post-mortem** obligatorio en las siguientes 48h (formato blameless).
4. **Action items** registrados en Jira con SLA de resolución.

### Disaster Recovery
- **RPO (Recovery Point Objective):** < 5 minutos (Oracle Data Guard).
- **RTO (Recovery Time Objective):** < 30 minutos.
- **Backup testing** trimestral con simulacro de failover.

---

## ✅ Checklist de Go-Live (Definition of Done para Pipeline)

### Pre-Desarrollo
- [ ] Repositorio creado con branch protection rules.
- [ ] Pipeline CI/CD configurado y funcionando en Dev.
- [ ] SonarQube project con Quality Gate corporativo.
- [ ] Vault path creado para secretos del servicio.
- [ ] Helm chart base generado desde template corporativo.

### Pre-Producción
- [ ] Cobertura de tests ≥ 80% (JaCoCo).
- [ ] SonarQube Quality Gate verde (0 vulnerabilities, 0 bugs).
- [ ] SAST y SCA sin hallazgos CRITICAL/HIGH.
- [ ] Imagen Docker escaneada y firmada.
- [ ] Contract tests con consumidores ejecutados.
- [ ] Performance test ejecutado y baseline documentado.
- [ ] Runbook operativo creado (runbook.md en el repo).
- [ ] Dashboards de Prometheus/Grafana configurados.
- [ ] Alertas de Splunk configuradas (P1, P2).
- [ ] Plan de rollback documentado y probado en staging.
- [ ] Aprobación de CAB y Security obtenida.
- [ ] Comunicación a stakeholders enviada.

---

## 📚 Referencias y Frameworks

- [DORA Metrics](https://dora.dev/research/) - Para medir desempeño del equipo (Deployment Frequency, Lead Time, MTTR, Change Failure Rate).
- [The Phoenix Project](https://itrevolution.com/the-phoenix-project/) - Principios de Three Ways.
- [Accelerate (Forsgren et al.)](https://itrevolution.com/accelerate-book/) - Evidencia científica de capacidades DevOps.
- [PCI-DSS v4.0 Requirement 6.4](https://www.pcisecuritystandards.org/) - Change control procedures.
- [Google SRE Book](https://sre.google/sre-book/table-of-contents/) - SLOs, error budgets, toil reduction.

---

**Última actualización:** 2025-01-20
**Versión:** 1.0
**Mantenido por:** Platform Engineering Guild
**Aprobado por:** VP of Engineering & Architecture Review Board