# Estimation Template for Enterprise Architecture Initiatives

## 📋 Propósito

Este template establece el estándar para estimar iniciativas de arquitectura enterprise en el contexto ERLN (retail masivo, misión crítica 24/7). Su objetivo es proporcionar al **Architecture Review Board (ARB)** y a los stakeholders de negocio una visión completa del **TCO (Total Cost of Ownership)**, **ROI (Return on Investment)** y **riesgos financieros** asociados a cualquier decisión arquitectónica.

**Principios de Estimación:**
- ✅ **Transparencia total**: Todo costo, supuesto y riesgo debe ser explícito.
- ✅ **Enfoque en valor**: No estimamos "tecnologías", estimamos "capacidades de negocio".
- ✅ **Three-Point Estimation**: Siempre presentar escenarios (optimista, realista, pesimista).
- ✅ **TCO a 3-5 años**: Considerar CAPEX inicial + OPEX continuo (licencias, soporte, mantenimiento).
- ✅ **Contingencia basada en riesgos**: No usar porcentajes arbitrarios (ej. "agregar 20%").
- ❌ **Prohibido**: Estimaciones de "orden de magnitud" sin desglose de supuestos.
- ❌ **Prohibido**: Ocultar costos de licencias enterprise bajo "open-source".

---

## 🎯 Metodología de Estimación

### 1. Three-Point Estimation (PERT)

Para cada componente de costo, estimar tres escenarios:

| Escenario | Definición | Uso |
|-----------|------------|-----|
| **Optimista (O)** | Todo sale mejor de lo esperado | Límite inferior |
| **Realista (M)** | Escenario más probable | Base de planificación |
| **Pesimista (P)** | Riesgos materializados, bloqueos, dependencias | Límite superior |

**Fórmula PERT (Expected Value):**
```
EV = (O + 4M + P) / 6
```

**Desviación Estándar (riesgo):**
```
σ = (P - O) / 6
```

### 2. T-Shirt Sizing para Iniciativas Tempranas

Cuando la iniciativa está en fase de ideación y no hay suficiente detalle para estimación precisa:

| Size | Esfuerzo (person-months) | Duración (calendar months) | Rango de Costo (USD) |
|------|--------------------------|----------------------------|----------------------|
| **XS** | 1-3 | 1-2 | $10K - $50K |
| **S** | 3-10 | 2-4 | $50K - $150K |
| **M** | 10-30 | 4-8 | $150K - $500K |
| **L** | 30-100 | 8-16 | $500K - $2M |
| **XL** | 100-300 | 16-36 | $2M - $8M |
| **XXL** | >300 | >36 | >$8M |

**Regla**: Ninguna iniciativa >L puede aprobarse sin un desglose detallado (este template).

---

## 💰 Componentes de Costo

### Estructura de Desglose (WBS de Costos)

```
1. CAPEX (Inversión Inicial)
   1.1. Licencias perpetuas (si aplica)
   1.2. Hardware / Infraestructura on-prem
   1.3. Implementación y configuración
   1.4. Migración de datos
   1.5. Capacitación
   1.6. Consultoría externa

2. OPEX (Costos Operativos Anuales)
   2.1. Licencias subscription (SaaS, soporte)
   2.2. Infraestructura cloud / hosting
   2.3. Personal operativo (DevOps, SRE, DBA)
   2.4. Monitoreo y observabilidad (Splunk, Datadog)
   2.5. Seguridad y cumplimiento (auditorías PCI-DSS)
   2.6. Backups y disaster recovery

3. Costos de Riesgo (Contingencia)
   3.1. Riesgos técnicos conocidos
   3.2. Riesgos de integración
   3.3. Riesgos regulatorios
```

### Plantilla de Estimación Detallada

| ID | Componente | Tipo (CAPEX/OPEX) | Optimista (O) | Realista (M) | Pesimista (P) | EV (Expected) | σ (Riesgo) | Supuestos Clave | Responsable |
|----|------------|-------------------|---------------|--------------|---------------|---------------|------------|-----------------|-------------|
| 1.1 | Licencias Kong Enterprise (3 años) | CAPEX | $120K | $150K | $180K | $150K | $10K | 50 nodes, soporte 24/7 | Platform Team |
| 1.2 | Oracle 12c Licenses (existing) | OPEX | $0 | $0 | $0 | $0 | $0 | Ya licenciado | DBA Team |
| 1.3 | Kubernetes Cluster (on-prem) | CAPEX | $80K | $100K | $150K | $105K | $11.6K | 20 worker nodes, 3 masters | Infra Team |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| **TOTAL** | | | **$X** | **$Y** | **$Z** | **$EV** | **$σ** | | |

---

## 📊 Análisis TCO a 3 Años

### Plantilla de TCO

| Año | CAPEX | OPEX | Costos de Riesgo | Total Anual | Acumulado |
|-----|-------|------|------------------|-------------|-----------|
| **Año 1** | $500K | $150K | $50K | $700K | $700K |
| **Año 2** | $0 | $180K | $30K | $210K | $910K |
| **Año 3** | $0 | $200K | $20K | $220K | $1,130K |
| **Total 3 años** | **$500K** | **$530K** | **$100K** | | **$1,130K** |

### Comparativa de Alternativas (si aplica)

| Criterio (peso) | Alternativa A: Kong Enterprise | Alternativa B: Oracle APICS | Alternativa C: APIGEE Edge |
|-----------------|--------------------------------|-----------------------------|----------------------------|
| **TCO 3 años** (30%) | $1.13M ✅ | $1.85M | $1.45M |
| **Time-to-Market** (20%) | 6 meses ✅ | 12 meses | 8 meses |
| **SLA soportado** (20%) | 99.99% ✅ | 99.95% | 99.99% |
| **Cumplimiento PCI-DSS** (15%) | Nativo ✅ | Nativo | Requiere customización |
| **Vendor Lock-in** (15%) | Medio | Alto (Oracle ecosystem) | Alto (Google) |
| **Ponderado Total** | **8.5/10** ✅ | 6.2/10 | 7.1/10 |

**Decisión recomendada**: Alternativa A (Kong Enterprise) - Mejor balance TCO/SLA/compliance.

---

## 📈 Cálculo de ROI y Payback Period

### Beneficios de Negocio (Cuantificables)

| Beneficio | Métrica | Valor Actual | Valor Post-Iniciativa | Ganancia Anual |
|-----------|---------|--------------|----------------------|----------------|
| Reducción de downtime | Horas/año | 43.8h (99.5% SLA) | 4.38h (99.95% SLA) | $520K (evita pérdidas) |
| Aceleración time-to-market | Días/feature | 45 días | 15 días | $300K (revenue anticipado) |
| Reducción de effort operativo | FTEs | 5 FTEs | 2 FTEs | $180K (salarios) |
| Evitación de multas PCI-DSS | Riesgo anual | $500K (probabilidad 10%) | $50K (probabilidad 1%) | $45K |
| **Total Beneficios Anuales** | | | | **$1,045K** |

### Cálculo de ROI

```
ROI = [(Beneficios Anuales - OPEX Anual) / Inversión Total] × 100

ROI = [($1,045K - $210K) / $1,130K] × 100 = 73.9% anual
```

### Payback Period

```
Payback = Inversión Total / Beneficios Netos Anuales
Payback = $1,130K / ($1,045K - $210K) = 1.35 años (≈ 16 meses)
```

**Regla**: Iniciativas enterprise deben tener payback < 3 años. Si > 3 años, requiere justificación estratégica (ej. cumplimiento regulatorio obligatorio).

---

## ⚠️ Matriz de Riesgos Financieros

### Identificación de Riesgos

| ID | Riesgo | Probabilidad (1-5) | Impacto Financiero (1-5) | Score (P×I) | Mitigación | Costo de Mitigación | Residual Risk |
|----|--------|--------------------|--------------------------|-------------|------------|---------------------|---------------|
| R1 | Retraso en migración de Oracle 12c a microservicios | 4 | 4 | 16 | Strangler Fig pattern + paralelización | $50K (consultoría) | 2 (8) |
| R2 | Sobrecosto de licencias Kong por subestimación de nodos | 3 | 3 | 9 | Capacity planning con 30% buffer | $0 (buffer incluido) | 1 (3) |
| R3 | Fallo en auditoría PCI-DSS retrasa go-live | 2 | 5 | 10 | Security champion + pre-audit | $30K (pen-test externo) | 1 (5) |
| R4 | Resistencia al cambio de equipos legacy | 3 | 3 | 9 | Training program + change management | $20K | 2 (6) |
| **Total Risk Exposure** | | | | **44** | | **$100K** | **22** |

### Cálculo de Contingencia

```
Contingencia = Σ (Probabilidad × Impacto Financiero) para cada riesgo

Ejemplo:
R1: 40% × $200K = $80K
R2: 30% × $50K = $15K
R3: 20% × $300K = $60K
R4: 30% × $40K = $12K
Total Contingencia = $167K ≈ $170K
```

**Regla**: La contingencia NO es un "porcentaje mágico". Debe derivarse de la matriz de riesgos cuantificada.

---

## 🎯 Escenarios de Estimación

### Escenario 1: Optimista
- **Supuestos**: Todo sale según plan, sin bloqueos, equipo experimentado.
- **Duración**: 12 meses.
- **Costo**: $950K (CAPEX $450K + OPEX $500K).
- **ROI**: 95% anual.
- **Probabilidad**: 20%.

### Escenario 2: Realista (Base)
- **Supuestos**: Retrasos menores, curva de aprendizaje, dependencias externas.
- **Duración**: 18 meses.
- **Costo**: $1,130K (CAPEX $500K + OPEX $630K).
- **ROI**: 74% anual.
- **Probabilidad**: 60%.

### Escenario 3: Pesimista
- **Supuestos**: Bloqueos técnicos, cambios regulatorios, alta rotación de equipo.
- **Duración**: 30 meses.
- **Costo**: $1,850K (CAPEX $600K + OPEX $1,250K).
- **ROI**: 35% anual.
- **Probabilidad**: 20%.

### Expected Value (PERT)

```
EV_Costo = (950K + 4×1,130K + 1,850K) / 6 = $1,223K
EV_Duración = (12 + 4×18 + 30) / 6 = 19 meses
EV_ROI = (95% + 4×74% + 35%) / 6 = 71.3%
```

**Recomendación al ARB**: Aprobar presupuesto de **$1,223K** con revisión trimestral de avance.

---

## 📝 Supuestos y Restricciones

### Supuestos Clave
1. **Disponibilidad de equipo**: 3 arquitectos, 8 desarrolladores senior, 2 QA, 1 DevOps dedicados al 100%.
2. **Infraestructura existente**: Cluster Kubernetes on-prem ya provisionado (solo costo de expansión).
3. **Licencias Oracle**: Ya licenciadas (no se considera costo incremental).
4. **Ventana de mantenimiento**: Cambios a producción solo en ventanas aprobadas por CAB (no 24/7).
5. **Estabilidad regulatoria**: No cambios mayores en PCI-DSS v4.0 durante el período de implementación.

### Restricciones
1. **Presupuesto máximo aprobado**: $1.5M (CAPEX + OPEX 3 años).
2. **Fecha límite regulatoria**: Cumplimiento PCI-DSS v4.0 antes de Q3 2026.
3. **SLA mínimo**: 99.99% para servicios de pago (no negociable).
4. **Zero downtime**: Migración de servicios críticos sin interrupción (blue-green obligatorio).
5. **Data residency**: Todos los datos de tarjetas deben permanecer en México (no cloud público fuera del país).

---

## ✅ Aprobación y Gobernanza

### Niveles de Aprobación

| Rango de Costo | Nivel de Aprobación Requerido | SLA de Aprobación |
|----------------|-------------------------------|-------------------|
| < $100K | Tech Lead + Engineering Manager | 3 días hábiles |
| $100K - $500K | Director of Engineering + Finance | 5 días hábiles |
| $500K - $2M | VP Engineering + CTO + CFO | 10 días hábiles |
| > $2M | Executive Committee + Board | 30 días hábiles |

### Checklist para Sometimiento al ARB

- [ ] Estimación desglosada por componentes (WBS completo).
- [ ] Análisis TCO a 3 años (mínimo).
- [ ] Cálculo de ROI y payback period documentado.
- [ ] Matriz de riesgos cuantificada (no cualitativa).
- [ ] Tres escenarios presentados (optimista, realista, pesimista).
- [ ] Supuestos y restricciones explícitos.
- [ ] Comparativa de alternativas (si aplica, mínimo 2 opciones).
- [ ] Alineación con strategic roadmap de negocio.
- [ ] Revisión previa con Finance (validación de supuestos financieros).
- [ ] Revisión previa con Security (validación de costos de cumplimiento).

---

## 🧪 Ejemplo Aplicado: Caso ERLN - Migración Strangler Fig

### Contexto
Migración del monolito legacy (Oracle Forms + PL/SQL) a microservicios Java 8 + Spring Boot + Kong Enterprise para el dominio de **Aprobación de Pagos**.

### Estimación Detallada (Extracto)

| ID | Componente | Tipo | O | M | P | EV | σ |
|----|------------|------|---|---|---|----|---|
| 1.1 | Licencias Kong Enterprise (50 nodes, 3 años) | CAPEX | $120K | $150K | $180K | $150K | $10K |
| 1.2 | Desarrollo microservicios (8 FTEs × 12 meses) | CAPEX | $600K | $750K | $1,000K | $775K | $66.7K |
| 1.3 | Migración de datos Oracle 12c | CAPEX | $50K | $80K | $150K | $87K | $16.7K |
| 1.4 | Testing & QA (incluyendo pen-test PCI) | CAPEX | $80K | $120K | $200K | $127K | $20K |
| 2.1 | Operación Kubernetes (2 SREs × 3 años) | OPEX | $180K | $220K | $280K | $223K | $16.7K |
| 2.2 | Monitoreo Splunk (licencia + ingest) | OPEX | $90K | $120K | $150K | $120K | $10K |
| 3.1 | Contingencia (basada en matriz de riesgos) | RIESGO | $100K | $170K | $300K | $180K | $33.3K |
| **TOTAL** | | | **$1,220K** | **$1,610K** | **$2,260K** | **$1,662K** | **$173.4K** |

### TCO 3 Años

| Año | CAPEX | OPEX | Riesgo | Total | Acumulado |
|-----|-------|------|--------|-------|-----------|
| Año 1 | $1,039K | $170K | $100K | $1,309K | $1,309K |
| Año 2 | $0 | $190K | $40K | $230K | $1,539K |
| Año 3 | $0 | $210K | $40K | $250K | $1,789K |
| **Total** | **$1,039K** | **$570K** | **$180K** | | **$1,789K** |

### ROI y Payback

- **Beneficios anuales cuantificados**: $1,045K (reducción downtime, aceleración time-to-market, evitación multas PCI).
- **OPEX anual promedio**: $190K.
- **Inversión total (EV)**: $1,789K.

```
ROI = [($1,045K - $190K) / $1,789K] × 100 = 47.8% anual
Payback = $1,789K / ($1,045K - $190K) = 2.1 años
```

### Escenarios

| Escenario | Duración | Costo Total | ROI | Probabilidad |
|-----------|----------|-------------|-----|--------------|
| Optimista | 14 meses | $1,450K | 62% | 25% |
| Realista | 20 meses | $1,789K | 48% | 55% |
| Pesimista | 32 meses | $2,650K | 22% | 20% |
| **Expected Value** | **21 meses** | **$1,867K** | **45%** | |

### Decisión ARB
✅ **Aprobado** con presupuesto de **$1,867K** (EV) y revisión trimestral de avance. Payback de 2.1 años dentro del límite de 3 años. Riesgo residual aceptable después de mitigaciones.

---

## 📚 Referencias y Frameworks

- [COBIT 2019](https://www.isaca.org/resources/cobit) - Framework de gobernanza de TI enterprise.
- [TOGAF ADM](https://www.opengroup.org/togaf) - Architecture Development Method (fases de estimación).
- [ITIL 4 - Digital and IT Strategy](https://www.axelos.com/best-practice-solutions/itil-4) - Gestión de valor de servicios.
- [PMI - Practice Standard for Project Estimating](https://www.pmi.org/pmbok-guide-standards/standards/practice-standard-for-project-estimating) - Técnicas de estimación.
- [Gartner - IT Financial Management](https://www.gartner.com/en/information-technology/insights/it-financial-management) - TCO y ROI analysis.

---

**Última actualización:** 2025-01-25  
**Versión:** 1.0  
**Mantenido por:** Enterprise Architecture Office  
**Aprobado por:** CTO & Finance Committee