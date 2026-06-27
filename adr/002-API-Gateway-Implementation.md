# ADR 002: Implementación de API Gateway Enterprise

## Estado
Aceptado (2020-08)

## Contexto

### Situación Actual
Durante la migración del sistema POS con Strangler Fig, nos enfrentamos a un problema creciente: teníamos tráfico yendo directamente al monolito legacy (Java 8 / WebLogic) y a los nuevos servicios independientes, sin un punto central de control.

El sistema POS opera en **19,556 sucursales** con transacciones críticas 24/7, especialmente durante fechas pico (Buen Fin, Navidad) donde cada minuto de downtime representa pérdidas millonarias.

Esto generaba:
- **Múltiples puntos de entrada** al sistema, dificultando la seguridad y cumplimiento normativo
- **Lógica de autenticación duplicada** en cada servicio
- **Sin visibilidad centralizada** del tráfico y comportamiento del sistema
- **Dificultad para hacer rollback** rápido de servicios específicos
- **Rate limiting inconsistente** entre monolito y nuevos servicios
- **Versionado de APIs caótico** sin estrategia unificada
- **Imposibilidad de cumplir** con estándares de seguridad PCI-DSS para pagos con tarjeta

### Necesidad de Cambio
Con la migración a Strangler Fig en progreso, necesitábamos:
1. Un punto único de entrada para todo el tráfico (interno y externo)
2. Capacidad de enrutar tráfico dinámicamente entre monolito y servicios
3. Centralizar políticas de seguridad, autenticación y autorización
4. Implementar observabilidad unificada con cumplimiento auditivo
5. Habilitar rollback rápido de servicios individuales
6. Gestionar versionado de APIs de forma consistente
7. Cumplir con estándares PCI-DSS para procesamiento de pagos

### Restricciones
- **Alta disponibilidad:** 99.99% de uptime requerido
- **Soporte enterprise:** Requiere SLA contractual con proveedor
- **Cumplimiento normativo:** PCI-DSS, SOX, auditorías internas
- **Escalabilidad:** Debe soportar picos de 15x el tráfico normal (Buen Fin, Navidad)
- **Integración con stack existente:** Oracle Database 12c, Java 8, WebLogic
- **Presupuesto enterprise:** Aprobado para solución comercial con soporte 24/7

## Opciones Consideradas

### Opción 1: Kong Enterprise (Comercial)
**Descripción:** Implementar Kong Enterprise con soporte contractual 24/7, dashboard administrativo y plugins enterprise.

**Ventajas:**
- ✅ Soporte enterprise 24/7 con SLA contractual
- ✅ Dashboard administrativo para operaciones
- ✅ Plugins enterprise (OAuth2, OIDC, rate limiting avanzado)
- ✅ Cumplimiento PCI-DSS documentado
- ✅ Escalabilidad probada en retail enterprise
- ✅ Independiente de vendor de base de datos
- ✅ Mayor comunidad y ecosistema de plugins
- ✅ **Tiempo de implementación: 6-8 semanas**
- ✅ **Costo: $150K-250K USD anuales**

**Desventajas:**
- ⚠️ Requiere certificación del equipo en Kong Enterprise
- ⚠️ Integración adicional con Oracle Database (no nativa)
- ⚠️ Dependencia de vendor para actualizaciones mayores

**Conclusión:** ✅ **ACEPTADA** como mejor opción por tiempo, presupuesto y flexibilidad

### Opción 2: Oracle API Platform Cloud Service (APICS)
**Descripción:** Implementar Oracle APICS como gateway nativo del ecosistema Oracle.

**Ventajas:**
- ✅ **Integración nativa con Oracle Database** (ya usado corporativamente)
- ✅ Consistencia con stack Oracle existente
- ✅ Soporte Oracle contractual
- ✅ Cumplimiento normativo incluido
- ✅ Dashboard administrativo enterprise
- ✅ Gestión de APIs completa (diseño, publicación, monitoreo)

**Desventajas:**
- ❌ **Costo significativamente mayor: $400K-600K USD anuales** (requiere licencias Oracle adicionales)
- ❌ **Tiempo de implementación: 4-6 meses** (complejidad de integración con ecosistema Oracle completo)
- ❌ Dependencia total de Oracle (vendor lock-in)
- ❌ Curva de aprendizaje pronunciada para equipo no especializado en Oracle
- ❌ Menor flexibilidad para integración con tecnologías no-Oracle
- ❌ Comunidad más pequeña comparada con Kong
- ❌ Requiere infraestructura Oracle Cloud o licencia on-premise costosa

**Conclusión:** ❌ **RECHAZADA** por costo excesivo, tiempo de implementación prolongado y vendor lock-in

**Justificación del rechazo:**
Aunque APICS es la opción "natural" al usar Oracle Database, el análisis de TCO (Total Cost of Ownership) mostró:
- **Costo 3x mayor** que Kong Enterprise
- **Tiempo de implementación 4x mayor** (6 meses vs. 6-8 semanas)
- **Vendor lock-in** limitaría futuras decisiones arquitectónicas
- **ROI negativo** en los primeros 2 años

### Opción 3: Azure API Management
**Descripción:** Implementar Azure API Management como gateway cloud-native.

**Ventajas:**
- ✅ Integración nativa con ecosistema Azure
- ✅ Escalabilidad automática
- ✅ Sin infraestructura que mantener
- ✅ Cumplimiento normativo incluido
- ✅ Costo basado en uso

**Desventajas:**
- ❌ Latencia adicional por estar en cloud (sucursales en México con conectividad variable)
- ❌ Dependencia total de conectividad a internet
- ❌ No apto para tráfico interno entre servicios on-premise
- ❌ Costo impredecible en picos de tráfico

**Conclusión:** ❌ Rechazada por latencia y dependencia de internet para sistema crítico en sucursales

### Opción 4: MuleSoft Anypoint Platform
**Descripción:** Implementar MuleSoft como plataforma de integración completa.

**Ventajas:**
- ✅ Plataforma enterprise probada en retail
- ✅ Soporte Salesforce/enterprise
- ✅ ESB + API Gateway en uno

**Desventajas:**
- ❌ Costo excesivo: $500K-800K USD anuales
- ❌ Over-engineering para necesidades actuales
- ❌ Tiempo de implementación: 4-6 meses
- ❌ Curva de aprendizaje pronunciada

**Conclusión:** ❌ Rechazada por costo y complejidad innecesaria

### Opción 5: Kong OSS (Open Source)
**Descripción:** Implementar Kong Open Source sin soporte enterprise.

**Ventajas:**
- ✅ Sin costo de licencias
- ✅ Comunidad activa

**Desventajas:**
- ❌ **Sin soporte enterprise** (inaceptable para sistema crítico)
- ❌ Sin SLA contractual
- ❌ Sin dashboard administrativo enterprise
- ❌ Sin cumplimiento PCI-DSS documentado
- ❌ Responsabilidad total de operación y seguridad en equipo interno
- ❌ **No cumple con políticas de seguridad corporativas** para sistemas de pagos

**Conclusión:** ❌ **RECHAZADA** por no cumplir requisitos enterprise de seguridad y soporte

### Opción 6: Sin API Gateway (Status Quo)
**Descripción:** Mantener enrutamiento directo sin gateway centralizado.

**Ventajas:**
- ✅ Sin costo adicional
- ✅ Sin componente adicional que mantener

**Desventajas:**
- ❌ Múltiples puntos de entrada (security nightmare)
- ❌ Lógica duplicada en cada servicio
- ❌ Sin visibilidad centralizada
- ❌ Rollback complejo
- ❌ **No cumple con PCI-DSS**
- ❌ **No escala con la migración**

**Conclusión:** ❌ Rechazada por problemas de seguridad, cumplimiento y escalabilidad

## Decisión

**Seleccionamos la Opción 1: Kong Enterprise**

### Justificación para el Business Case

**Comparación de TCO (Total Cost of Ownership) a 3 años:**

| Concepto | Kong Enterprise | Oracle APICS | Diferencia |
|----------|-----------------|--------------|------------|
| Licencias anuales | $200K | $500K | -$300K/año |
| Implementación (una vez) | $100K | $300K | -$200K |
| Capacitación equipo | $50K | $150K | -$100K |
| Mantenimiento anual | $50K | $100K | -$50K/año |
| **TCO 3 años** | **$1.1M** | **$2.55M** | **-$1.45M** |

**ROI del API Gateway Enterprise (Kong):**

| Concepto | Costo/Beneficio |
|----------|-----------------|
| Costo anual licencias Kong Enterprise | $200K USD |
| Costo evitado en incidentes de seguridad | $500K-1M USD/año |
| Costo evitado en downtime durante Buen Fin | $300K-500K USD/evento |
| Ahorro en horas de soporte por centralización | $150K USD/año |
| Cumplimiento PCI-DSS (evita multas) | $1M+ USD potencial |
| **ROI neto año 1** | **$1.5M-2.5M USD** |

**Tiempo de implementación:**
- Kong Enterprise: 6-8 semanas
- Oracle APICS: 4-6 meses
- **Diferencia crítica:** Kong permite cumplir con deadlines de expansión regional (Plazas Veracruz)

**Flexibilidad arquitectónica:**
- Kong: Independiente, puede integrarse con cualquier tecnología
- APICS: Vendor lock-in con ecosistema Oracle
- **Decisión estratégica:** Mantener flexibilidad para futuras decisiones tecnológicas

### Implementación

#### Fase 1: Setup y Configuración Base (Semanas 1-3)
- Desplegar Kong Enterprise en cluster Kubernetes on-premise (3 nodos HA)
- Configurar Oracle Database como backend de configuración (estándar corporativo)
- Implementar monitoreo enterprise (Splunk + dashboards Kong Admin)
- Documentar arquitectura de despliegue con aprobación de seguridad

#### Fase 2: Configuración de Rutas y Servicios (Semanas 4-5)
- Definir rutas para monolito legacy Java (todas las APIs existentes)
- Configurar rutas para nuevos servicios (se agregan gradualmente)
- Implementar estrategia de versionado de APIs (v1, v2, etc.)
- Configurar plugins de logging enterprise para todas las rutas

#### Fase 3: Seguridad y Cumplimiento (Semanas 6-8)
- Implementar plugin de autenticación OAuth2/OIDC integrado con Entra ID
- Configurar rate limiting avanzado por IP, API key y usuario
- Implementar CORS para aplicaciones frontend
- Configurar TLS/SSL con certificados corporativos
- **Validación PCI-DSS** con equipo de seguridad corporativo
- Auditoría de configuración por equipo de compliance

#### Fase 4: Observabilidad y Monitoreo Enterprise (Semanas 9-10)
- Configurar integración con Splunk para logging centralizado
- Implementar métricas enterprise (latencia, error rate, throughput)
- Crear dashboards ejecutivos y técnicos
- Configurar alertas proactivas con integración a ServiceNow
- Implementar distributed tracing con OpenTelemetry

#### Fase 5: Migración Gradual de Tráfico (Semanas 11-14)
- Redirigir 10% del tráfico a través del gateway (validación)
- Monitorear métricas y comparar con tráfico directo
- Aumentar a 50% después de 1 semana sin incidentes
- Migrar 100% del tráfico después de validación completa
- Mantener capacidad de rollback rápido (feature flags)
- **Runbook de operación** documentado y aprobado por operaciones

### Criterios de Éxito
- Overhead de latencia < 30ms (p95)
- Disponibilidad 99.99%
- 100% del tráfico pasando por gateway
- Visibilidad completa de métricas y logs
- Capacidad de rollback en < 2 minutos
- **Aprobación de auditoría PCI-DSS**

## Consecuencias

### Beneficios Obtenidos

#### Técnicas
- ✅ **Punto único de entrada** simplificó seguridad y observabilidad
- ✅ **Overhead de latencia:** 12ms promedio (mejor que objetivo de 30ms)
- ✅ **Rate limiting centralizado** previno ataques y abuso de APIs
- ✅ **Logging unificado en Splunk** permitió análisis de tráfico y cumplimiento auditivo
- ✅ **Rollback rápido:** Capacidad de desactivar servicios en < 2 minutos
- ✅ **Versionado de APIs** consistente y documentado
- ✅ **Alta disponibilidad:** 3 nodos Kong con failover automático, 0 downtime en 18 meses

#### de Negocio
- ✅ **Cumplimiento PCI-DSS** aprobado en auditoría
- ✅ **Mejor seguridad** redujo riesgo de brechas de datos
- ✅ **Visibilidad del tráfico** permitió identificar patrones de uso y optimizar recursos
- ✅ **Tiempo de respuesta a incidentes** reducido de 30 min a 5 min
- ✅ **Habilitó migración gradual** sin interrupciones de servicio
- ✅ **ROI positivo** en primer año ($1.5M+ en beneficios vs. $200K en licencias)

#### Organizacionales
- ✅ **Equipo de operaciones** tiene control centralizado con dashboard enterprise
- ✅ **Desarrolladores** no necesitan preocuparse por cross-cutting concerns
- ✅ **Onboarding de nuevos servicios** estandarizado y documentado
- ✅ **Auditorías simplificadas** gracias a logging centralizado

### Consideraciones y Trade-offs Gestionados

#### 1. Dependencia del Gateway
**Consideración:** Al centralizar todo el tráfico en Kong, se convierte en componente crítico.

**Gestión implementada:**
- ✅ Cluster de 3 nodos Kong con failover automático
- ✅ Health checks cada 5 segundos con recuperación automática
- ✅ Procedimiento de disaster recovery documentado y probado trimestralmente
- ✅ Resultado: **0 downtime en 18 meses de operación**

#### 2. Complejidad de Infraestructura
**Consideración:** Kong Enterprise + Oracle + monitoreo enterprise agrega complejidad.

**Gestión implementada:**
- ✅ Automatización completa con Ansible y Helm charts
- ✅ Equipo certificado en Kong Enterprise (3 ingenieros)
- ✅ Soporte contractual 24/7 con Kong para escalamiento
- ✅ Documentación de operación en runbooks aprobados

#### 3. Curva de Aprendizaje
**Consideración:** Equipo necesita capacitación en Kong Enterprise.

**Gestión implementada:**
- ✅ Certificación oficial Kong Enterprise para 3 ingenieros
- ✅ Sesiones de knowledge transfer con equipo de Kong
- ✅ Documentación interna con casos de uso específicos
- ✅ Resultado: Equipo completamente autónomo en 3 meses

#### 4. Proceso de Cambios de API
**Consideración:** Todos los cambios deben pasar por el gateway.

**Gestión implementada:**
- ✅ Proceso de self-service con aprobación automatizada
- ✅ Pipeline CI/CD para despliegue de configuraciones
- ✅ SLA de 24 horas para cambios estándar
- ✅ Resultado: **No se convirtió en bottleneck**, al contrario, estandarizó procesos

#### 5. Mantenimiento Continuo
**Consideración:** Requiere actualizaciones y parches de seguridad.

**Gestión implementada:**
- ✅ Contrato de soporte incluye actualizaciones
- ✅ Ventanas de mantenimiento mensuales pre-aprobadas
- ✅ Parches de seguridad aplicados en < 48 horas (SLA contractual)
- ✅ Resultado: **Sistema siempre actualizado** sin impacto en operación

#### 6. Testing de Integración
**Consideración:** Testing debe validar integración con gateway.

**Gestión implementada:**
- ✅ Ambiente de staging idéntico a producción
- ✅ Tests automatizados de integración en CI/CD
- ✅ Contract testing con Pact para validar APIs
- ✅ Resultado: **Bugs de integración detectados en staging**, no en producción

## Lecciones Aprendidas

### 1. La opción "natural" del vendor no siempre es la mejor
Oracle APICS era la opción obvia al usar Oracle Database, pero el análisis de TCO mostró que Kong Enterprise era 3x más económico y 4x más rápido de implementar. **Las decisiones arquitectónicas deben basarse en datos, no en inercia.**

### 2. El vendor lock-in tiene un costo oculto
APICS nos habría atado al ecosistema Oracle, limitando futuras decisiones. Kong Enterprise nos dio independencia para integrar con cualquier tecnología, lo cual fue crítico para la migración a microservicios.

### 3. El tiempo de implementación es tan importante como el costo
APICS habría tomado 4-6 meses, perdiendo la ventana de expansión regional. Kong Enterprise en 6-8 semanas nos permitió cumplir con los deadlines del negocio.

### 4. En sistemas enterprise, el soporte contractual no es opcional
Para un sistema que opera en 19,556 sucursales, no puedes depender de soporte comunitario. El SLA contractual con Kong Enterprise nos dio la tranquilidad de tener respaldo 24/7.

### 5. El cumplimiento normativo es un driver arquitectónico
PCI-DSS no fue una ocurrencia tardía, fue un requisito desde el día 1. Esto nos ahorró meses de retrabajo y posibles multas millonarias.

### 6. La inversión en alta disponibilidad se paga sola
El cluster de 3 nodos con failover automático costó 30% más que una instalación básica, pero nos dio 99.99% de disponibilidad. En retail, cada minuto de downtime cuesta decenas de miles de dólares.

### 7. El ROI debe cuantificarse desde el inicio
Presentar el business case con ROI claro ($1.5M+ en año 1) y TCO comparativo vs. APICS fue clave para obtener aprobación ejecutiva. Sin números concretos, el proyecto no habría avanzado.

## Referencias

- Kong Enterprise Documentation: https://docs.konghq.com/enterprise/
- PCI-DSS Compliance Guide: https://www.pcisecuritystandards.org/
- API Gateway Pattern: https://microservices.io/patterns/apigateway.html
- Oracle Database Integration: https://docs.konghq.com/gateway/latest/reference/configuration/#database
- TCO Analysis Framework: https://www.gartner.com/en/information-technology/insights/technology-tco

## Decisiones Relacionadas

- [ADR 001: Strangler Fig Migration](./001-strangler-fig-migration.md)
- [ADR 003: Circuit Breaker Pattern](./003-circuit-breaker-pattern.md)
- [ADR 004: Cross-Team Alignment Framework](./004-cross-team-alignment.md)

---

**Autor:** Mario Guerrero  
**Fecha:** 2020-08  
**Revisado por:** 
- Equipo de Arquitectura
- Equipo de Seguridad (PCI-DSS)
- Equipo de Operaciones
- Equipo de Compliance
- Equipo de Base de Datos (Oracle DBAs)
- Stakeholders de Negocio
- **Aprobado por Architecture Review Board**