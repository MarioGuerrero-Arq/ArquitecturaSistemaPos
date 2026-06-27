# 🏗️ Enterprise Architecture Portfolio — Mario Guerrero

<div align="center">

![Enterprise Architecture](https://img.shields.io/badge/Architecture-Enterprise-blue?style=for-the-badge)
![Java 8](https://img.shields.io/badge/Java-8-orange?style=for-the-badge)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-2.x-green?style=for-the-badge)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Production-326CE5?style=for-the-badge)
![Oracle 12c](https://img.shields.io/badge/Oracle-12c-F80000?style=for-the-badge)
![PCI-DSS](https://img.shields.io/badge/PCI--DSS-v4.0-red?style=for-the-badge)

**Arquitecto de Aplicaciones Senior | 13+ años en Enterprise Software**
**Especialización: Modernización de Sistemas Críticos | Retail Masivo | Fintech**

[Ver Case Study ERLN](#-case-study-ERLN-retail-platform-modernization) | [Ver ADRs](#-architecture-decision-records-adrs) | [Ver Guidelines](#-guidelines--baselines) | [Ver Templates](#-operational-templates)

</div>

---

## 📜 Manifiesto del Arquitecto

> **"No hay decisiones técnicas, solo decisiones de negocio con implicaciones técnicas."**

Como Arquitecto de Aplicaciones, mi rol es **traducir objetivos de negocio en soluciones técnicas sostenibles**, balanceando tres dimensiones críticas:

1. **Valor de Negocio**: Cada decisión debe aumentar revenue, reducir costos o mitigar riesgos.
2. **TCO a 3-5 años**: Las soluciones deben ser financieramente sostenibles (CAPEX + OPEX).
3. **Riesgo Residual**: Las decisiones deben permitir "dormir tranquilo" (SLA, compliance, security).

### Principios Rectores

✅ **Pragmatismo sobre Perfección**: Soluciones "suficientemente buenas" que entregan valor hoy, no arquitecturas de ivory tower.

✅ **Datos sobre Opiniones**: Cada ADR incluye TCO comparativo, matriz de riesgos y ROI proyectado.

✅ **Seguridad y Compliance by Design**: PCI-DSS, Zero-Trust, y auditoría SOX no son afterthoughts.

✅ **Observabilidad como Requisito**: Si no puedes medirlo, no puedes mejorarlo (RED metrics, distributed tracing).

✅ **Evolución sobre Revolución**: Strangler Fig sobre Big Bang, migraciones incrementales con rollback en < 5 min.

❌ **Rechazado**: Open-source sin soporte enterprise en sistemas críticos.

❌ **Rechazado**: Vendor lock-in extremo sin estrategia de salida.

❌ **Rechazado**: Trade-offs técnicos presentados como "problemas" sin plan de mitigación.

---

## 🎯 Case Study: ERLN Retail Platform Modernization

### Contexto

**ERLN** es el minorista más grande de México con **19,556 sucursales** y operación **misión crítica 24/7**. Procesaba más de **2.5 millones de transacciones diarias** de pagos de servicios, recargas y ventas propias a través de un **monolito legacy** construido sobre **Oracle Forms + PL/SQL** desplegado en **WebLogic**.

### El Reto

| Problema | Impacto de Negocio |
|----------|---------------------|
| **Downtime mensual** | 43.8 horas (SLA 99.5%) → $6.2M USD/año en pérdidas |
| **Time-to-market** | 45 días por feature → pérdida de oportunidades competitivas |
| **Costo de mantenimiento** | 5 FTEs dedicados solo a sostener legacy → $450K USD/año |
| **Riesgo PCI-DSS** | 10% probabilidad de fallo en auditoría → multa potencial $500K USD |
| **Escalabilidad** | Imposibilidad de absorber picos de Black Friday (3x carga normal) |

### Mandato del Negocio

El CTO y el Architecture Review Board (ARB) aprobaron un programa de modernización con objetivos no negociables:

1. ✅ **SLA 99.99%** (máximo 4.38 minutos de downtime mensual).
2. ✅ **Time-to-market < 15 días** por feature crítica.
3. ✅ **Cumplimiento PCI-DSS v4.0** antes de Q3 2026.
4. ✅ **TCO a 3 años < $2M USD** con ROI demostrable.
5. ✅ **Cero interrupción** durante la migración (zero-downtime migration).

### Mi Rol

Como **Arquitecto de Aplicaciones Principal**, fui responsable de:
- Diseñar la arquitectura target y la estrategia de migración.
- Liderar la aprobación de 4 ADRs críticos ante el ARB.
- Coordinar 6 equipos cross-funcionales (48 personas).
- Garantizar el cumplimiento de SLA 99.99% y PCI-DSS durante la transición.
- Reportar directamente al CTO y al comité de arquitectura.

### Resultados Cuantificables

#### Métricas Técnicas
| Métrica | Antes | Después | Mejora |
|---------|-------|---------|--------|
| **SLA** | 99.5% (43.8h downtime/mes) | **99.99%** (4.38 min/mes) | **90% reducción downtime** |
| **Deployment Frequency** | 1/mes | **12/semana** | **48x más rápido** |
| **Lead Time for Changes** | 45 días | **4 días** | **11x más rápido** |
| **Change Failure Rate** | 35% | **3%** | **91% reducción** |
| **MTTR** | 8 horas | **12 minutos** | **40x más rápido** |

#### Métricas de Negocio
| Métrica | Valor Anual |
|---------|-------------|
| **Pérdidas evitadas por downtime** | $520K USD |
| **Revenue anticipado por time-to-market** | $300K USD |
| **Reducción de effort operativo** | $180K USD (3 FTEs reasignados) |
| **Evitación de multas PCI-DSS** | $45K USD (risk-adjusted) |
| **Beneficio anual total** | **$1,045K USD** |

#### Análisis Financiero (TCO 3 años)
| Concepto | Valor |
|----------|-------|
| **Inversión total (EV)** | $1,789K USD |
| **Beneficios anuales** | $1,045K USD |
| **ROI anual** | **47.8%** |
| **Payback period** | **2.1 años** |
| **Ahorro vs alternativa Oracle APICS** | $670K USD (3 años) |

### Historia STAR Completa

📄 **[Ver Executive Case Study Completo](./case-study/case-study-ERLN-executive-summary.md)**

---

## 📚 Portafolio de Artefactos

Este repositorio contiene **11 artefactos enterprise** que demuestran experiencia real en arquitectura de aplicaciones, organizados en cuatro categorías:

### 📘 Architecture Decision Records (ADRs)

Documentos que capturan decisiones arquitectónicas críticas, incluyendo contexto, alternativas evaluadas, decisión tomada y consecuencias.

| # | ADR | Decisión | Alternativa Descartada | Justificación de Negocio |
|---|-----|----------|------------------------|---------------------------|
| 1 | **[001-strangler-fig-migration.md](./adr/001-strangler-fig-migration.md)** | Strangler Fig Pattern | Big Bang Rewrite | Reduce riesgo de $2M a $170K, permite rollback en < 5 min |
| 2 | **[002-api-gateway-implementation.md](./adr/002-api-gateway-implementation.md)** | Kong Enterprise | Oracle APICS | Ahorro de $670K en 3 años, time-to-market 6 meses vs 12 |
| 3 | **[003-circuit-breaker-pattern.md](./adr/003-circuit-breaker-pattern.md)** | Resilience4j | Hystrix | Hystrix en mantenimiento desde 2020, riesgo de soporte enterprise |
| 4 | **[004-cross-team-alignment-framework.md](./adr/004-Cross-Team-Alignment-Framework.md)** | Cross-Team Alignment | Estructura funcional tradicional | Alinea Conway's Law, reduce dependencias entre equipos en 70% |

### 📗 Guidelines & Baselines

Estándares técnicos y líneas base de seguridad/operaciones aplicables a todos los microservicios y aplicaciones enterprise.

| # | Guideline | Propósito | Alcance |
|---|-----------|-----------|---------|
| 5 | **[api-design-guidelines.md](./guidelines/api-design-guidelines.md)** | Estándares de diseño de APIs | OpenAPI 3.0, versioning, idempotencia, HATEOAS, error handling |
| 6 | **[security-baseline.md](./guidelines/security-baseline.md)** | Zero-Trust baseline con PCI-DSS | Identidad, API security, protección de datos, gestión de secretos |
| 7 | **[devops-ci-cd-standards.md](./guidelines/devops-ci-cd-standards.md)** | Estándares de CI/CD y operaciones | GitFlow, Blue-Green, testing pyramid, observabilidad, IaC |

### 📙 Operational Templates

Plantillas reutilizables para estimación, migración y gestión de calidad en proyectos enterprise.

| # | Template | Propósito | Uso |
|---|----------|-----------|-----|
| 8 | **[estimation-template.md](./templates/estimation-template.md)** | Estimación con PERT, TCO, ROI | Análisis financiero de iniciativas, matriz de riesgos cuantificada |
| 9 | **[m2s-checklist.md](./templates/m2s-checklist.md)** | Checklist de migración monolito a microservicios | 11 fases con Go/No-Go gates, Strangler Fig, data migration |
| 10 | **[test-plan-template.md](./templates/test-plan-template.md)** | Plan de pruebas enterprise | Testing pyramid, chaos engineering, PCI-DSS validation |

### 📕 Executive Summary

Resumen ejecutivo del case study para presentaciones ante ARB, comités ejecutivos y entrevistas técnicas.

| # | Documento | Propósito | Audiencia |
|---|-----------|-----------|-----------|
| 11 | **[case-study-ERLN-executive-summary.md](./case-study/case-study-ERLN-executive-summary.md)** | Executive brief del caso ERLN | CTO, ARB, reclutadores, entrevistadores técnicos |

---

## 🛠️ Stack Tecnológico

### Core Platform
| Capa | Tecnología | Justificación |
|------|------------|---------------|
| **Lenguaje** | Java 8 | Compatibilidad con WebLogic 12c, equipo experimentado |
| **Framework** | Spring Boot 2.x | Estándar enterprise, ecosistema maduro |
| **Base de datos** | Oracle 12c + TDE | Licencias existentes, cumplimiento PCI-DSS |
| **API Gateway** | Kong Enterprise | Mejor TCO, plugins PCI-DSS nativos |
| **Orquestación** | Kubernetes (on-prem) | Data residency México, control total |
| **Resiliencia** | Resilience4j | Estándar moderno, activo en comunidad |
| **Observabilidad** | Splunk + Prometheus | Estándar corporativo, integración con SOAR |
| **Secretos** | HashiCorp Vault | Rotación automática, dynamic secrets Oracle |

### DevOps & CI/CD
- **CI/CD**: Jenkins / GitLab CI
- **Artifact Repository**: JFrog Artifactory
- **Container Registry**: Harbor
- **GitOps**: ArgoCD
- **IaC**: Terraform + Ansible + Helm
- **Code Quality**: SonarQube Enterprise
- **Security Scanning**: Checkmarx (SAST), Snyk (SCA), Trivy (Container)

### Testing & Quality
- **Unit/Integration**: JUnit 5, Mockito, Testcontainers
- **Contract Testing**: Pact (consumer-driven)
- **E2E**: Selenium, RestAssured
- **Performance**: JMeter, Gatling
- **Chaos Engineering**: Gremlin, Litmus

---

## 📊 Arquitectura Target

### Diagrama Conceptual (C4 - Container Level)

```
┌─────────────────────────────────────────────────────────────────────┐
│                         CANAL DE ENTRADA                             │
│  [POS Sucursales]  [Mobile App]  [Web]  [3rd Party APIs]            │
└────────────────────────────┬────────────────────────────────────────┘
                             │ HTTPS / TLS 1.2+
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   KONG ENTERPRISE (API Gateway)                      │
│  [JWT Auth] [Rate Limiting] [WAF] [Bot Detection] [CORS] [Logging]  │
└────────────────────────────┬────────────────────────────────────────┘
                             │
        ┌────────────────────┼────────────────────┐
        ▼                    ▼                    ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  PAYMENT     │    │  INVENTORY   │    │  CUSTOMER    │
│  SERVICE     │    │  SERVICE     │    │  SERVICE     │
│  (Spring     │    │  (Spring     │    │  (Spring     │
│   Boot)      │    │   Boot)      │    │   Boot)      │
│              │    │              │    │              │
│ [Resilience4j│    │ [Resilience4j│    │ [Resilience4j│
│  Circuit     │    │  Circuit     │    │  Circuit     │
│  Breaker]    │    │  Breaker]    │    │  Breaker]    │
└──────┬───────┘    └──────┬───────┘    └──────┬───────┘
       │                   │                   │
       ▼                   ▼                   ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Oracle 12c  │    │  Oracle 12c  │    │  Oracle 12c  │
│  (TDE ON)    │    │  (TDE ON)    │    │  (TDE ON)    │
│  PAYMENT_DB  │    │  INV_DB      │    │  CUST_DB     │
└──────────────┘    └──────────────┘    └──────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                    CAPA TRANSVERSAL                                  │
│  [HashiCorp Vault]  [Splunk]  [Prometheus]  [Grafana]  [PagerDuty]  │
│  [LaunchDarkly]     [ArgoCD]  [Jenkins]     [SonarQube] [Snyk]      │
└─────────────────────────────────────────────────────────────────────┘
```

### Principios Arquitectónicos Aplicados

1. **Database per Service**: Cada microservicio dueño de sus datos (Oracle 12c schema aislado).
2. **API-First**: Contratos OpenAPI 3.0 firmados antes de implementar.
3. **Zero-Trust Security**: mTLS service-to-service, JWT en edge, Vault para secretos.
4. **Observability by Design**: RED metrics, structured logging, distributed tracing.
5. **Chaos-Ready**: Circuit breakers, fallbacks, bulkheads en cada servicio.
6. **GitOps**: Todo infraestructura y configuración versionada en Git.

---

## 💡 Lecciones Aprendidas

### ✅ Lo que funcionó

1. **Strangler Fig sobre Big Bang**: La migración incremental permitió validar hipótesis temprano y pivotar sin costos hundidos.
2. **ARB como aliado, no como obstáculo**: Involucrar al ARB desde el día 1 con análisis TCO/ROI robustos aceleró aprobaciones.
3. **Security-by-design**: Integrar PCI-DSS desde el diseño evitó retrabajos costosos (ahorro estimado de $400K).
4. **Cross-team alignment framework**: Los guilds técnicos resolvieron el 80% de las dependencias sin escalamiento ejecutivo.
5. **Chaos engineering trimestral**: Los GameDays detectaron 3 puntos únicos de fallo antes de que impactaran producción.

### ⚠️ Lo que tuvimos que ajustar

1. **Subestimamos la migración de datos**: Oracle PL/SQL embebido en forms requirió 40% más effort del planificado.
   - **Mitigación**: Incorporamos DBAs especializados en el equipo desde el inicio.
2. **Resistencia al cambio de equipos legacy**: 30% del equipo requería upskilling en Java/Spring.
   - **Mitigación**: Programa de training de 8 semanas con pair programming.
3. **Kong Enterprise licensing**: El modelo por nodos requería capacity planning más preciso.
   - **Mitigación**: Buffer del 30% en licensing + renegotiation anual.

---

## 🚀 Roadmap de Evolución (Próximos 18 meses)

### Q3 2026 — Completar Migración de Dominios
- Migrar dominios de Inventario y Clientes (Strangler Fig phase 2).
- Decommissionar 60% del monolito legacy.
- **Meta**: 80% de tráfico en microservicios.

### Q4 2026 — Event-Driven Architecture
- Implementar **Apache Kafka** para domain events.
- Migrar integraciones síncronas a patrones event-driven.
- **Meta**: Reducir acoplamiento entre dominios en 50%.

### Q1 2027 — Data Mesh
- Implementar principios de **Data Mesh** (dominios como producto de datos).
- Self-serve data platform para equipos de negocio.
- **Meta**: Time-to-insight reducido de 2 semanas a 2 días.

### Q2 2027 — AI/ML Integration
- Integrar **ML models** para detección de fraude en tiempo real.
- Personalización de promociones por sucursal.
- **Meta**: Reducción de fraude en 40%, incremento de conversión en 15%.

---

## 📖 Cómo Usar Este Portafolio

### Para Reclutadores y Hiring Managers

Este portafolio demuestra **experiencia enterprise real** en arquitectura de aplicaciones. Los artefactos están organizados para facilitar la evaluación:

1. **Inicio rápido**: Lee el [Executive Case Study](./case-study-ERLN-executive-summary.md) para entender el contexto y resultados.
2. **Profundización técnica**: Revisa los [ADRs](#-architecture-decision-records-adrs) para ver cómo se tomaron decisiones críticas.
3. **Estándares de calidad**: Explora las [Guidelines](#-guidelines--baselines) para entender los estándares aplicados.
4. **Capacidad operativa**: Revisa los [Templates](#-operational-templates) para ver cómo se gestionan proyectos enterprise.

### Para Entrevistas Técnicas

Cada artefacto está diseñado para responder preguntas comunes de entrevistas:

- **"¿Cómo tomas decisiones arquitectónicas?"** → Revisa los ADRs (contexto, alternativas, decisión, consecuencias).
- **"¿Cómo manejas trade-offs técnicos?"** → Revisa el ADR-002 (Kong vs Oracle APICS con TCO comparativo).
- **"¿Cómo garantizas cumplimiento regulatorio?"** → Revisa el [Security Baseline](./guidelines/security-baseline.md) (PCI-DSS, Zero-Trust).
- **"¿Cómo migras sistemas legacy?"** → Revisa el [M2S Checklist](./templates/m2s-checklist.md) (11 fases con Go/No-Go gates).
- **"¿Cómo estimas proyectos enterprise?"** → Revisa el [Estimation Template](./templates/estimation-template.md) (PERT, TCO, ROI, matriz de riesgos).

### Para Arquitectos y Líderes Técnicos

Este portafolio puede servir como **referencia y template** para tus propios proyectos:

- **ADRs**: Usa el formato y estructura para documentar tus decisiones.
- **Guidelines**: Adapta los estándares a tu contexto organizacional.
- **Templates**: Personaliza las plantillas para tus procesos de estimación, migración y testing.
- **Case Study**: Usa la estructura STAR para narrar tus propios casos de éxito.

---

## 🏆 Reconocimientos

- **Premio interno de Innovación Tecnológica 2025** (categoría "Transformación Enterprise").
- **Caso presentado en Oracle Latin America Summit 2025** (migración Oracle 12c + TDE).
- **Benchmark adoptado** por otras unidades de negocio del grupo (ERLN Gas, ERLN Farmacias).

---

## 📞 Contacto

**Mario Guerrero**
Enterprise Application Architect

- 📧 Email: [mario.guerrerom@outlook.com](mailto:mario.guerrerom@outlook.com)
- 💼 LinkedIn: [https://www.linkedin.com/in/mario-guerrero-arc/](https://www.linkedin.com/in/mario-guerrero-arc/)
- 🐙 GitHub: [https://github.com/MarioGuerrero-Arq](https://github.com/MarioGuerrero-Arq)

---

## 📚 Referencias y Frameworks Aplicados

- **TOGAF ADM** — Architecture Development Method (gobernanza arquitectónica).
- **COBIT 2019** — Framework de gobernanza de TI enterprise.
- **DORA Metrics** — Accelerate state report (capacidades DevOps).
- **Team Topologies** — Skelton & Pais (organización de equipos).
- **Building Microservices** — Sam Newman (2nd Edition, 2021).
- **Monolith to Microservices** — Sam Newman (2019).
- **Microservices Patterns** — Chris Richardson (2018).
- **Release It!** — Michael Nygard (2nd Edition, 2018).
- **PCI-DSS v4.0** — Payment Card Industry Data Security Standard.
- **NIST SP 800-207** — Zero Trust Architecture.

---

## ⚖️ Licencia y Uso

Este portafolio es de **uso profesional** para entrevistas técnicas y referencias profesionales. Los datos de negocio son anonimizados pero la arquitectura, decisiones técnicas y patrones aplicados son **100% reales y reproducibles**.

**Nota de anonimización**: Las cifras de negocio son representativas del sector retail masivo mexicano. Los nombres de sistemas internos han sido reemplazados por descripciones genéricas.

---

<div align="center">

**¿Interesado en discutir arquitectura enterprise?**

[Contáctame](mailto:mario.guerrerom@outlook.com) | [Ver LinkedIn](https://www.linkedin.com/in/mario-guerrero-arc/) | [Explorar Portafolio](https://github.com/MarioGuerrero-Arq)

*Última actualización: 2026-06-27 | Versión: 1.0*

</div>