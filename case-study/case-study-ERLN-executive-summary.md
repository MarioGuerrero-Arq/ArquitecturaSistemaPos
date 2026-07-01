# ERLN Retail Platform Modernization — Executive Case Study

## 📋 Resumen Ejecutivo

Este documento presenta el **caso de estudio enterprise** de la modernización arquitectónica de la plataforma de pagos de ERLN, el minorista más grande de México con **19,556 sucursales** y operación **misión crítica 24/7**. El caso demuestra la aplicación práctica de principios de arquitectura enterprise, toma de decisiones basada en datos (TCO/ROI), y gestión de riesgos en un entorno regulado por **PCI-DSS**, con un SLA de **99.99%**.

> **Nota de anonimización**: Las cifras de negocio son representativas del sector retail masivo mexicano. La arquitectura, decisiones técnicas y patrones aplicados son **100% reales y reproducibles**.

---

## 🎯 El Reto de Negocio

### Contexto
ERLN procesaba más de **2.5 millones de transacciones diarias** de pagos de servicios, recargas y ventas propias a través de un **monolito legacy** construido sobre **Oracle Forms + PL/SQL** desplegado en **WebLogic**. El sistema presentaba:

| Problema | Impacto de Negocio |
|----------|---------------------|
| **Downtime mensual promedio** | 43.8 horas (SLA 99.5%) → $6.2M USD/año en pérdidas |
| **Time-to-market** | 45 días por feature → pérdida de oportunidades competitivas |
| **Costo de mantenimiento** | 5 FTEs dedicados solo a sostener legacy → $450K USD/año |
| **Riesgo PCI-DSS** | 10% probabilidad de fallo en auditoría → multa potencial $500K USD |
| **Escalabilidad** | Imposibilidad de absorber picos de Black Friday (3x carga normal) |

### Mandato del Negocio
El CTO y el Architecture Review Board (ARB) aprobaron un programa de modernización con los siguientes objetivos no negociables:

1. **SLA 99.99%** (máximo 4.38 minutos de downtime mensual).
2. **Time-to-market < 15 días** por feature crítica.
3. **Cumplimiento PCI-DSS v4.0** antes de Q3 2026.
4. **TCO a 3 años < $2M USD** con ROI demostrable.
5. **Cero interrupción** durante la migración (zero-downtime migration).

---

## 📖 Historia STAR — Migración del Dominio de Pagos

### **S**ituación (Contexto)
El monolito legacy de ERLN, con más de **1.2 millones de líneas de código PL/SQL** y **15 años de evolución**, sostenía el 100% de las transacciones de pagos en 19,556 sucursales. El sistema había alcanzado sus límites físicos y técnicos:
- Cada cambio requería ventanas de mantenimiento de 6+ horas.
- Los despliegues fallaban el 35% de las veces (change failure rate).
- El equipo de 25 desarrolladores operaba en modo "reactivo", apagando incendios.

### **T**area (Responsabilidad)
Como **Arquitecto de Aplicaciones Principal**, fui responsable de:
- Diseñar la arquitectura target y la estrategia de migración.
- Liderar la aprobación de 4 ADRs críticos ante el ARB.
- Coordinar 6 equipos cross-funcionales (48 personas).
- Garantizar el cumplimiento de SLA 99.99% y PCI-DSS durante la transición.
- Reportar directamente al CTO y al comité de arquitectura.

### **A**cción (Estrategia y Ejecución)

#### 1. Estrategia de Migración (ADR-001: Strangler Fig)
Decisión estratégica de **migración incremental** en lugar de "big bang rewrite":
- **Branch by Abstraction** para convivir monolito + microservicios.
- **Anti-Corruption Layer** para traducir entre dominios legacy y nuevos.
- **Secuencia de extracción** priorizada por valor de negocio: Pagos → Inventario → Clientes → Promociones.

#### 2. API Gateway Enterprise (ADR-002: Kong Enterprise vs Oracle APICS)
Análisis comparativo con selección de **Kong Enterprise** basado en:
- **TCO 3 años**: $1.13M vs $1.85M (Oracle APICS) → **39% de ahorro**.
- **Time-to-market**: 6 meses vs 12 meses.
- **SLA soportado**: 99.99% nativo.
- **Plugins de seguridad PCI-DSS**: JWT, rate limiting, WAF, bot detection out-of-the-box.

#### 3. Resiliencia Distribuida (ADR-003: Resilience4j vs Hystrix)
Implementación de patrones de resiliencia con **Resilience4j**:
- Circuit Breaker, Retry, Bulkhead, Rate Limiter en cada microservicio.
- Fallbacks con degradación elegante (no errores 500 al usuario).
- Hystrix descartado por estar en mantenimiento desde 2020 (riesgo de soporte).

#### 4. Alineación Cross-Team (ADR-004: Cross-Team Alignment Framework)
Estructura organizacional tipo **Team Topologies**:
- **Stream-aligned teams** por dominio de negocio (Pagos, Inventario, Clientes).
- **Platform team** responsable de Kubernetes, Kong, observabilidad.
- **Enabling team** (arquitectura) acompañando la adopción de prácticas.
- **Guilds** técnicos transversales (Security, DevOps, Data).

#### 5. Stack Tecnológico Enterprise
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

### **R**esultado (Impacto Cuantificable)

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

#### Cumplimiento y Gobernanza
- ✅ **PCI-DSS v4.0** validado por QSA externo (0 hallazgos críticos).
- ✅ **Zero downtime** durante migración (Strangler Fig + blue-green).
- ✅ **11 Go/No-Go Gates** aprobados por ARB sin excepciones.
- ✅ **Zero data loss** en migración de 15 años de datos transaccionales.

---

## 🏛️ Arquitectura Target

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

## 🎓 Decisiones Arquitectónicas Clave (ADRs)

| ADR | Decisión | Alternativa Descartada | Justificación de Negocio |
|-----|----------|------------------------|---------------------------|
| **ADR-001** | Strangler Fig Pattern | Big Bang Rewrite | Reduce riesgo de $2M a $170K, permite rollback en < 5 min |
| **ADR-002** | Kong Enterprise | Oracle APICS | Ahorro de $670K en 3 años, time-to-market 6 meses vs 12 |
| **ADR-003** | Resilience4j | Hystrix | Hystrix en mantenimiento desde 2020, riesgo de soporte enterprise |
| **ADR-004** | Cross-Team Alignment | Estructura funcional tradicional | Alinea Conway's Law, reduce dependencias entre equipos en 70% |

Cada ADR incluye:
- Análisis de alternativas con scoring ponderado.
- TCO comparativo a 3 años.
- Matriz de riesgos cuantificada.
- Criterios de revisión (revisit date).

---

## 📚 Artefactos de Gobernanza Generados

El portafolio completo incluye los siguientes artefactos enterprise:

### 📘 Architecture Decision Records (ADRs)
1. `001-strangler-fig-migration.md` — Estrategia de migración incremental.
2. `002-API-Gateway-Implementation.md` — Selección de Kong Enterprise.
3. `003-Circuit-Breaker-Pattern.md` — Resiliencia con Resilience4j.
4. `004-Cross-Team-Alignment-Framework.md` — Alineación organizacional.

### 📗 Guidelines y Baselines
5. `api-design-guidelines.md` — Estándares de diseño de APIs (OpenAPI 3.0, versioning, idempotencia).
6. `security-baseline.md` — Zero-Trust baseline con cumplimiento PCI-DSS.
7. `devops-ci-cd-standards.md` — Estándares de CI/CD, GitFlow, Blue-Green.

### 📙 Templates Operativos
8. `estimation-template.md` — Estimación con PERT, TCO, ROI, matriz de riesgos.
9. `m2s-checklist.md` — Checklist de 11 fases para migración monolito a microservicios.
10. `test-plan-template.md` — Plan de pruebas enterprise (testing pyramid, chaos, PCI-DSS).

### 📕 Manifiesto
11. `README.md` — Manifiesto del arquitecto (filosofía, principios, valores).

Todos los artefactos siguen el principio de **"decisión técnica → justificación de negocio → mitigación de riesgos"**, alineados con frameworks enterprise como **TOGAF ADM**, **COBIT 2019** y **DORA Metrics**.

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

### 🎯 Principios Rectores Validados

> **"No hay decisiones técnicas, solo decisiones de negocio con implicaciones técnicas."**

Cada decisión arquitectónica fue evaluada con tres lentes:
1. **Valor de negocio**: ¿Aumenta revenue, reduce costos o mitiga riesgos?
2. **TCO a 3-5 años**: ¿Es sostenible operacional y financieramente?
3. **Riesgo residual**: ¿Podemos dormir tranquilos con esta decisión?

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

## 🏆 Impacto Organizacional

### Evolución Cultural
- **De reactivo a proactivo**: El equipo pasó de "apagar incendios" a "prevenir incendios".
- **Engineering excellence**: Adopción de prácticas como code reviews obligatorios, pair programming, blameless post-mortems.
- **Developer happiness**: eNPS del equipo aumentó de 32 a 68 (medición interna anual).

### Capacidades Instaladas
- **6 equipos stream-aligned** autónomos (pueden desplegar a producción sin dependencias).
- **Platform team** maduro con SLAs internos de 99.95%.
- **Internal Developer Portal** (Backstage) con self-service para 120+ desarrolladores.
- **Golden paths** documentados para nuevas funcionalidades (time-to-hello-world < 1 día).

---

## 📞 Contacto y Profundización

Este case study es parte de un **portafolio de arquitectura** disponible en GitHub. Los artefactos completos (ADRs, guidelines, templates) están documentados con el nivel de detalle esperado en una organización enterprise real.

**Para profundizar en**:
- **Estrategia de migración** → Ver `001-strangler-fig-migration.md`
- **Análisis comparativo de API Gateways** → Ver `002-api-gateway-implementation.md`
- **Gestión de riesgos técnicos** → Ver `estimation-template.md` (sección de matriz de riesgos)
- **Checklist operativo de migración** → Ver `m2s-checklist.md` (11 fases con Go/No-Go gates)
- **Cumplimiento PCI-DSS** → Ver `security-baseline.md` (Zero-Trust baseline)

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

**Última actualización:** 2026-06-30  
**Versión:** 1.1  
**Autor:** Mario Guerrero — Enterprise Application Architect  
**Contexto:** Caso de estudio enterprise basado en experiencia real anonimizada  
**Licencia:** Portafolio profesional — Uso para entrevistas técnicas y referencias profesionales