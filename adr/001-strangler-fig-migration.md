# ADR 001: Migración Gradual con Patrón Strangler Fig

## Estado
Aceptado (2020-07)

## Contexto

### Situación Inicial
El sistema POS operaba en 19,556 sucursales con una arquitectura monolítica legacy en **Java 8** con base de datos **Oracle 12c**. El sistema presentaba:

- **Alta latencia** en servicios críticos, especialmente durante fechas pico (Buen Fin, Navidad)
- **Acoplamiento fuerte** entre módulos (facturación, inventario, pagos, promociones)
- **Tiempos de compilación** superiores a 12 minutos para el monolito completo
- **Cobertura de pruebas** inferior al 15%
- **Dificultad para escalar** componentes específicos sin afectar todo el sistema
- **Procedimientos almacenados Oracle** con lógica de negocio embebida (más de 500 stored procedures)
- **Transacciones distribuidas** complejas usando XA transactions de Oracle

### Necesidad de Cambio
El negocio requería:
1. Expandir el sistema a nuevas regiones (Plazas Chihuahua, Plazas Veracruz)
2. Implementar nuevas estrategias de venta cíclica a 40 días
3. Reducir incidentes durante fechas de alto volumen
4. Mejorar tiempo de respuesta en punto de venta
5. Cumplir con estándares PCI-DSS para procesamiento de pagos con tarjeta
6. Preparar el sistema para integración con e-commerce y omnicanalidad

### Restricciones
- **No se podía detener la operación:** El sistema debía mantenerse en producción 24/7
- **Presupuesto enterprise:** Aprobado para solución comercial con soporte
- **Tiempo ajustado:** Plazos comerciales para expansión regional (6 meses)
- **Riesgo alto:** Cualquier error afectaba a 19,556 sucursales
- **Cumplimiento normativo:** PCI-DSS, SOX, auditorías internas
- **Stack existente:** Oracle Database, WebLogic, Java 8

## Opciones Consideradas

### Opción 1: Reescritura Completa (Big Bang)
**Descripción:** Reconstruir todo el sistema desde cero con arquitectura moderna de microservicios.

**Ventajas:**
- Arquitectura limpia desde el inicio
- Oportunidad para eliminar toda la deuda técnica
- Tecnología moderna (microservicios, cloud-native)

**Desventajas:**
- Tiempo estimado: 18-24 meses
- Costo: $2M-3M USD (5-10x mayor que migración gradual)
- Riesgo extremo: período prolongado sin sistema funcional
- No viable para restricciones de tiempo del negocio
- Pérdida de lógica de negocio embebida en 500+ stored procedures de Oracle

**Conclusión:** ❌ Rechazada por riesgo, costo y tiempo inaceptables

### Opción 2: Migración Gradual con Strangler Fig
**Descripción:** Reemplazar gradualmente componentes del monolito Java con servicios independientes, usando un proxy para enrutar tráfico.

**Ventajas:**
- Migración incremental, reduciendo riesgo en cada paso
- Sistema permanece en producción durante toda la migración
- Permite validar arquitectura con componentes reales
- Costo y tiempo distribuidos en el tiempo
- Posibilidad de rollback en cada fase
- Preserva lógica de negocio existente en Oracle

**Desventajas:**
- Requiere disciplina para mantener ambos sistemas durante transición
- Complejidad temporal en enrutamiento de tráfico
- Necesita inversión en herramientas de observabilidad
- Requiere estrategia para migrar stored procedures gradualmente

**Conclusión:** ✅ Aceptada como mejor balance entre riesgo, costo y tiempo

### Opción 3: Mantener Monolito y Optimizar
**Descripción:** Quedarse con la arquitectura actual Java/Oracle y optimizar rendimiento.

**Ventajas:**
- Sin riesgo de migración
- Costo inmediato bajo

**Desventajas:**
- No resuelve problemas de escalabilidad
- Deuda técnica continúa creciendo
- No habilita expansión regional
- Limita innovación futura
- No cumple con requisitos de omnicanalidad

**Conclusión:** ❌ Rechazada por no abordar necesidades del negocio

## Decisión

**Seleccionamos la Opción 2: Migración Gradual con Patrón Strangler Fig**

### Implementación

#### Fase 1: Identificación de Bounded Contexts (Semanas 1-3)
Realizamos sesiones de Event Storming para identificar contextos delimitados:
- Facturación
- Inventario
- Pagos (crítico para PCI-DSS)
- Promociones
- Reportes

Para cada contexto, documentamos:
- Stored procedures de Oracle asociados
- Lógica de negocio embebida
- Dependencias con otros contextos
- Volumen de transacciones diario

#### Fase 2: Implementación de API Gateway (Semanas 4-8)
Desplegamos Kong Enterprise como punto único de entrada para:
- Enrutamiento de tráfico (monolito Java vs. nuevos servicios)
- Autenticación centralizada (OAuth2/OIDC con Entra ID)
- Rate limiting
- Logging y observabilidad (integración con Splunk)
- Cumplimiento PCI-DSS

**Ver ADR 002 para detalles de implementación**

#### Fase 3: Extracción Gradual de Servicios (Semanas 9-20)
Priorizamos servicios por:
1. **Mayor latencia** (impacto directo en experiencia de usuario)
2. **Menor acoplamiento** (más fáciles de extraer)
3. **Mayor valor de negocio** (ROI inmediato)
4. **Requisitos de cumplimiento** (PCI-DSS para pagos)

Orden de extracción:
1. **Servicio de Pagos** (alta latencia, crítico para PCI-DSS)
   - Migración de lógica de pagos desde stored procedures Oracle
   - Implementación de encriptación end-to-end
   - Integración con procesadores de pagos certificados PCI
   
2. **Servicio de Inventario** (latencia media, bajo acoplamiento)
   - Migración de consultas complejas Oracle a servicios
   - Implementación de caché distribuido con Redis
   - Sincronización eventual con monolito durante transición

3. **Servicio de Promociones** (latencia media, acoplamiento bajo)
   - Extracción de reglas de negocio desde Oracle
   - Implementación de motor de reglas en Java

#### Fase 4: Implementación de Resiliencia (Semanas 15-18)
Añadimos **Resilience4j** (estándar para Java) en todas las llamadas entre servicios para:
- Circuit Breakers para prevenir fallos en cascada
- Retry policies con backoff exponencial
- Rate limiters
- Bulkheads para aislamiento de recursos
- Timeouts configurables

**Ver ADR 003 para detalles de implementación**

#### Fase 5: Monitoreo y Observabilidad Enterprise (Semanas 19-22)
Implementamos:
- Logging estructurado en puntos críticos del flujo de negocio
- Integración con Splunk para centralización de logs
- Métricas de latencia, error rate y throughput con Prometheus + Grafana
- Dashboards ejecutivos y técnicos
- Alertas proactivas con integración a ServiceNow
- Distributed tracing con Jaeger

#### Fase 6: Migración de Stored Procedures (Semanas 20-24)
Estrategia para migrar lógica de Oracle:
1. Identificar stored procedures críticas (alto uso, lógica compleja)
2. Reimplementar lógica en Java dentro de microservicios
3. Validar resultados comparando con Oracle (dual-write temporal)
4. Desactivar stored procedures en Oracle gradualmente
5. Mantener backup de lógica Oracle documentada

### Criterios de Éxito
- Latencia reducida en al menos 30%
- Change orders post-liberación reducidos en al menos 40%
- 0 incidentes críticos durante fechas pico
- Migración completada sin interrupciones de servicio
- Cumplimiento PCI-DSS aprobado en auditoría

## Consecuencias

### Beneficios Obtenidos

#### Técnicas
- ✅ **Reducción de 35% en latencia** de servicios críticos
- ✅ **Reducción de 45% en change orders** post-liberación
- ✅ **Mejor observabilidad** del sistema mediante logging estratégico en Splunk
- ✅ **Escalabilidad independiente** de componentes
- ✅ **Tiempos de compilación** reducidos de 12 a 3 minutos por servicio
- ✅ **Cobertura de pruebas** aumentó de 15% a 65%
- ✅ **Cumplimiento PCI-DSS** aprobado en auditoría

#### de Negocio
- ✅ **0 incidentes críticos** durante Buen Fin y Navidad
- ✅ **Migración exitosa** a Plazas Chihuahua y Veracruz
- ✅ **Ahorro estimado:** $400K-900K USD en costos evitados
- ✅ **Habilitación de nuevas estrategias** de venta cíclica
- ✅ **ROI positivo** en primer año ($1.5M+ en beneficios)

#### Organizacionales
- ✅ **Mejor coordinación** entre QA, Infraestructura y Soporte
- ✅ **Transferencia de conocimiento** mediante pair programming
- ✅ **Modelo de migración** replicable para futuras expansiones
- ✅ **Equipo capacitado** en arquitecturas distribuidas

### Consideraciones y Trade-offs Gestionados

#### 1. Complejidad Temporal durante Transición
**Consideración:** Mantener monolito Java y nuevos servicios simultáneamente.

**Gestión implementada:**
- ✅ API Gateway como punto único de control
- ✅ Feature flags para activar/desactivar servicios gradualmente
- ✅ Documentación clara de qué componentes están en monolito vs. servicios
- ✅ Resultado: Transición sin incidentes en 6 meses

#### 2. Migración de Stored Procedures de Oracle
**Consideración:** Más de 500 stored procedures con lógica de negocio embebida.

**Gestión implementada:**
- ✅ Priorización de stored procedures críticas
- ✅ Dual-write temporal para validación
- ✅ Reimplementación en Java con pruebas exhaustivas
- ✅ Documentación de lógica Oracle como referencia
- ✅ Resultado: 80% de stored procedures migradas en 6 meses

#### 3. Curva de Aprendizaje en Patrones Distribuidos
**Consideración:** Equipo Java necesitaba aprender microservicios, circuit breakers, etc.

**Gestión implementada:**
- ✅ Pair programming intensivo durante primeros 3 meses
- ✅ Sesiones de capacitación semanal (1 hora)
- ✅ Certificación en Resilience4j y patrones de microservicios
- ✅ Documentación de patrones y mejores prácticas
- ✅ Resultado: Equipo completamente autónomo en 4 meses

#### 4. Transacciones Distribuidas
**Consideración:** Oracle XA transactions reemplazadas por sagas pattern.

**Gestión implementada:**
- ✅ Implementación de Saga Pattern para transacciones distribuidas
- ✅ Event sourcing para trazabilidad
- ✅ Compensación automática en caso de fallos
- ✅ Resultado: Consistencia eventual con trazabilidad completa

## Lecciones Aprendidas

### 1. La paciencia es una virtud arquitectónica
Strangler Fig requiere tiempo y disciplina. No es para proyectos que necesitan resultados inmediatos, pero para sistemas críticos en producción, es la mejor opción.

### 2. La migración de stored procedures es más compleja de lo esperado
La lógica de negocio embebida en Oracle tomó más tiempo del previsto. Debimos haber empezado esta fase 2 meses antes.

### 3. La observabilidad no es opcional
Sin logging estratégico y métricas claras (Splunk, Prometheus, Grafana), no podríamos haber validado que la migración estaba funcionando. Invertimos 3 semanas en instrumentación que nos ahorraron meses de debugging.

### 4. El API Gateway es el corazón de la migración
Sin Kong Enterprise, la migración habría sido caótica. Nos permitió controlar tráfico, hacer rollback rápido y tener un punto único de observabilidad.

### 5. La comunicación cross-team es tan importante como la arquitectura
Las reuniones de 15 minutos tuvieron más impacto que cualquier decisión técnica. Debimos implementarlas desde el día 1.

### 6. PCI-DSS es un driver arquitectónico, no un afterthought
Integrar cumplimiento desde el inicio nos ahorró meses de retrabajo. El servicio de pagos fue el primero en migrarse precisamente por esto.

## Referencias

- Martin Fowler: [Strangler Fig Pattern](https://martinfowler.com/bliki/StranglerFigApplication.html)
- Sam Newman: [Building Microservices](https://samnewman.io/books/building_microservices/)
- Microsoft: [Architectural Patterns](https://docs.microsoft.com/en-us/azure/architecture/patterns/)
- Oracle: [Migrating from Monolith to Microservices](https://www.oracle.com/technical-resources/articles/java/microservices-monolith.html)

## Decisiones Relacionadas

- [ADR 002: API Gateway Implementation](./002-api-gateway-implementation.md)
- [ADR 003: Circuit Breaker Pattern](./003-circuit-breaker-pattern.md)
- [ADR 004: Cross-Team Alignment Framework](./004-cross-team-alignment.md)

---

**Autor:** Mario Guerrero  
**Fecha:** 2020-07  
**Revisado por:** 
- Equipo de Arquitectura
- Equipo de Base de Datos (Oracle DBAs)
- Equipo de Seguridad (PCI-DSS)
- **Aprobado por Architecture Review Board**