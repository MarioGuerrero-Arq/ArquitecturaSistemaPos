# ADR 003: Implementación de Circuit Breaker Pattern con Resilience4j

## Estado
Aceptado (2020-09)

## Contexto

### Situación Actual
Con la migración a microservicios en progreso y el API Gateway (Kong Enterprise) implementado, identificamos un problema crítico: cuando un servicio externo (pagos, inventario, promociones) tenía latencia alta o caía, el sistema completo se degradaba.

El sistema POS opera en **19,556 sucursales** con transacciones críticas 24/7. Cada minuto de downtime en fechas pico representa pérdidas de $50K-100K USD.

Ejemplos de incidentes previos:
- **Incidente 1:** Servicio de pagos con latencia de 10s → 19,556 sucursales esperando → timeout en cascada → sistema completo caído por 45 minutos → pérdida estimada $2M USD
- **Incidente 2:** Servicio de inventario caído → todas las transacciones fallando → pérdida estimada de $500K en ventas durante horas pico
- **Incidente 3:** Servicio de promociones lento → respuestas en 8s → experiencia de usuario degradada → quejas de tiendas

El problema raíz: **llamadas síncronas HTTP entre servicios Java sin timeout adecuado ni mecanismo de fallback**.

### Necesidad de Cambio
Necesitábamos:
1. **Prevenir fallos en cascada:** Si un servicio cae, no debe arrastrar al sistema completo
2. **Degradación elegante:** El sistema debe seguir funcionando (aunque con funcionalidad reducida) cuando un servicio falla
3. **Recuperación automática:** Cuando el servicio se recupera, el sistema debe volver a usarlo automáticamente
4. **Observabilidad:** Saber qué servicios están fallando y con qué frecuencia
5. **Timeouts configurables:** Diferentes servicios tienen diferentes SLAs
6. **Integración con stack Java:** Debe funcionar con Spring Boot, WebLogic, etc.

### Restricciones
- **No cambiar contratos de API:** Los servicios existentes no podían modificarse
- **Implementación incremental:** Debíamos poder agregar circuit breakers servicio por servicio
- **Overhead mínimo:** El patrón no podía agregar latencia significativa (< 10ms)
- **Configuración dinámica:** Deberíamos poder ajustar umbrales sin redeployar
- **Compatibilidad con Java 8+:** Stack corporativo estándar
- **Integración con monitoreo enterprise:** Prometheus, Grafana, Splunk
- **Cumplimiento PCI-DSS:** Especialmente crítico para servicio de pagos

## Opciones Consideradas

### Opción 1: Resilience4j
**Descripción:** Usar Resilience4j, la biblioteca estándar de resiliencia para Java, sucesora espiritual de Hystrix.

**Ventajas:**
- ✅ **Estándar de facto para Java** (reemplazo moderno de Hystrix)
- ✅ Integración nativa con Spring Boot (Spring Cloud Circuit Breaker)
- ✅ Soporte para Circuit Breaker, Retry, Rate Limiter, Time Limiter, Bulkhead
- ✅ Comunidad activa y documentación excelente
- ✅ Configuración vía application.yml (dinámica con Spring Cloud Config)
- ✅ Métricas integradas con Prometheus y Micrometer
- ✅ Soporte para anotaciones (@CircuitBreaker, @Retry, etc.)
- ✅ Tiempo de implementación: 2-3 semanas
- ✅ **Sin costo de licencias** (open source con soporte comercial disponible)
- ✅ Forward-compatible con Java 11+ y Java 17

**Desventajas:**
- ⚠️ Requiere implementación en cada servicio (no centralizado)
- ⚠️ Curva de aprendizaje para patrones avanzados

**Conclusión:** ✅ **ACEPTADA** como estándar corporativo para resiliencia en Java

### Opción 2: Hystrix (Netflix)
**Descripción:** Usar Hystrix, el patrón original de Netflix para resiliencia.

**Ventajas:**
- ✅ Patrón probado a escala masiva (Netflix)
- ✅ Dashboard de monitoreo en tiempo real (Hystrix Dashboard)
- ✅ Soporte para fallbacks y bulkheads

**Desventajas:**
- ❌ **En modo mantenimiento desde 2018** (no se desarrolla activamente)
- ❌ No soporta Java 11+ de forma nativa
- ❌ Requiere Hystrix Dashboard adicional (infraestructura)
- ❌ Integración limitada con Spring Boot 2.x+
- ❌ Comunidad migrando a Resilience4j
- ❌ Riesgo de seguridad sin parches futuros

**Conclusión:** ❌ Rechazada por estar en mantenimiento y no ser forward-compatible

### Opción 3: Sentinel (Alibaba)
**Descripción:** Usar Sentinel, la solución de resiliencia de Alibaba.

**Ventajas:**
- ✅ Soporte para múltiples protocolos (HTTP, gRPC, Dubbo)
- ✅ Dashboard de monitoreo
- ✅ Soporte para flow control, circuit breaking, system adaptive protection

**Desventajas:**
- ❌ Comunidad más pequeña comparada con Resilience4j
- ❌ Documentación principalmente en chino
- ❌ Menos integración con ecosistema Spring
- ❌ Adopción limitada fuera de Asia
- ❌ Soporte enterprise limitado en México

**Conclusión:** ❌ Rechazada por adopción limitada y barreras de documentación

### Opción 4: Implementación Custom
**Descripción:** Construir nuestro propio circuit breaker desde cero.

**Ventajas:**
- ✅ Control total sobre comportamiento
- ✅ Integración perfecta con nuestras necesidades específicas

**Desventajas:**
- ❌ Tiempo de desarrollo: 6-8 semanas
- ❌ Reinventar la rueda (Resilience4j ya resuelve esto)
- ❌ Alto riesgo de bugs en componente crítico
- ❌ Mantenimiento continuo requerido
- ❌ No hay justificación de ROI

**Conclusión:** ❌ Rechazada por tiempo, riesgo y falta de ROI

### Opción 5: Sin Circuit Breaker (Status Quo)
**Descripción:** Mantener llamadas síncronas sin protección.

**Ventajas:**
- ✅ Sin overhead adicional
- ✅ Sin cambios en código

**Desventajas:**
- ❌ Fallos en cascada continúan
- ❌ Sistema completo cae cuando un servicio falla
- ❌ Sin degradación elegante
- ❌ Incidentes continúan con pérdidas millonarias
- ❌ **Inaceptable para sistema crítico con 19,556 sucursales**

**Conclusión:** ❌ Rechazada por problemas críticos de disponibilidad y pérdidas financieras

## Decisión

**Seleccionamos la Opción 1: Resilience4j**

### Justificación para el Business Case

**ROI de Resilience4j:**

| Concepto | Costo/Beneficio |
|----------|-----------------|
| Costo de implementación (2-3 semanas equipo) | $50K USD |
| Costo evitado en incidentes de cascada | $500K-1M USD/año |
| Costo evitado en downtime durante Buen Fin | $2M USD/evento |
| Mejora en disponibilidad (98.5% → 99.7%) | $300K USD/año |
| **ROI neto año 1** | **$2.5M-3M USD** |

**Criterios de selección:**

| Criterio | Peso | Resilience4j | Hystrix | Sentinel | Custom |
|----------|------|--------------|---------|----------|--------|
| Soporte Java moderno | 25% | 10 | 4 | 7 | 8 |
| Integración Spring Boot | 20% | 10 | 5 | 6 | 3 |
| Comunidad y documentación | 15% | 10 | 6 | 4 | 2 |
| Tiempo de implementación | 15% | 9 | 7 | 6 | 3 |
| Métricas y observabilidad | 15% | 10 | 8 | 8 | 5 |
| Costo | 10% | 10 | 10 | 10 | 2 |
| **Puntuación ponderada** | **100%** | **9.8** | **6.2** | **6.3** | **4.3** |

## Implementación

### Fase 1: Análisis de Servicios Críticos (Semana 1)
Identificamos servicios externos que requerían circuit breaker:
- Servicio de Pagos (crítico, SLA 99.99%, PCI-DSS)
- Servicio de Inventario (alto volumen, SLA 99.9%)
- Servicio de Promociones (medio volumen, SLA 99.5%)
- Servicio de Reportes (bajo volumen, SLA 99%)

Para cada servicio, definimos:
- Timeout máximo aceptable
- Umbral de fallos para abrir circuito
- Tiempo de espera antes de intentar semi-abrir
- Estrategia de fallback
- Métricas de monitoreo

### Fase 2: Configuración de Resilience4j (Semana 2)
Implementamos configuración centralizada en `application.yml`:

```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        registerHealthIndicator: true
        slidingWindowSize: 10
        minimumNumberOfCalls: 5
        permittedNumberOfCallsInHalfOpenState: 3
        automaticTransitionFromOpenToHalfOpenEnabled: true
        waitDurationInOpenState: 30s
        failureRateThreshold: 50
        eventConsumerBufferSize: 10
      inventoryService:
        registerHealthIndicator: true
        slidingWindowSize: 20
        minimumNumberOfCalls: 10
        permittedNumberOfCallsInHalfOpenState: 5
        automaticTransitionFromOpenToHalfOpenEnabled: true
        waitDurationInOpenState: 60s
        failureRateThreshold: 40
        eventConsumerBufferSize: 10
  retry:
    instances:
      paymentService:
        maxRetryAttempts: 3
        waitDuration: 1s
        enableExponentialBackoff: true
        exponentialBackoffMultiplier: 2
  timelimiter:
    instances:
      paymentService:
        timeoutDuration: 5s
      inventoryService:
        timeoutDuration: 3s
  bulkhead:
    instances:
      paymentService:
        maxConcurrentCalls: 50
        maxWaitDuration: 100ms
```

Creamos configuración centralizada con Spring Cloud Config para gestión dinámica.

### Fase 3: Implementación en Servicios (Semanas 3-4)
Para cada servicio crítico, implementamos usando anotaciones de Spring:

```java
@Service
public class PaymentServiceClient {
    
    private final PaymentApiClient paymentApiClient;
    
    public PaymentServiceClient(PaymentApiClient paymentApiClient) {
        this.paymentApiClient = paymentApiClient;
    }
    
    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    @Retry(name = "paymentService")
    @TimeLimiter(name = "paymentService")
    @Bulkhead(name = "paymentService", type = Bulkhead.Type.SEMAPHORE)
    public CompletableFuture<PaymentResponse> processPayment(PaymentRequest request) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                return paymentApiClient.process(request);
            } catch (Exception ex) {
                log.error("Payment service call failed", ex);
                throw ex;
            }
        });
    }
    
    public CompletableFuture<PaymentResponse> paymentFallback(
            PaymentRequest request, Exception ex) {
        log.warn("Using fallback for payment service: {}", ex.getMessage());
        return CompletableFuture.completedFuture(
            PaymentResponse.builder()
                .status("QUEUED")
                .message("Payment will be processed later")
                .build()
        );
    }
}
```

Agregamos fallbacks para degradación elegante en el servicio de inventario:

```java
@Service
public class InventoryServiceClient {
    
    private final InventoryApiClient inventoryApiClient;
    
    public InventoryServiceClient(InventoryApiClient inventoryApiClient) {
        this.inventoryApiClient = inventoryApiClient;
    }
    
    @CircuitBreaker(name = "inventoryService", fallbackMethod = "inventoryFallback")
    @Retry(name = "inventoryService")
    @TimeLimiter(name = "inventoryService")
    public CompletableFuture<InventoryResponse> checkInventory(ProductRequest request) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                return inventoryApiClient.check(request);
            } catch (Exception ex) {
                log.error("Inventory service call failed", ex);
                throw ex;
            }
        });
    }
    
    public CompletableFuture<InventoryResponse> inventoryFallback(
            ProductRequest request, Exception ex) {
        log.warn("Using fallback for inventory service");
        return CompletableFuture.completedFuture(
            InventoryResponse.builder()
                .available(false)
                .message("Inventory check temporarily unavailable")
                .estimatedRestockTime(LocalDateTime.now().plusHours(2))
                .build()
        );
    }
}
```

### Fase 4: Testing y Validación (Semana 5)
- **Unit tests:** Validar comportamiento de circuit breaker (closed, open, half-open)
- **Integration tests:** Simular fallos de servicios externos con WireMock
- **Load tests:** Validar comportamiento bajo carga con JMeter
- **Chaos engineering:** Introducir fallos aleatorios con Chaos Monkey
- **Pruebas de fallback:** Validar que degradación elegante funciona correctamente
- **Pruebas PCI-DSS:** Validar que fallbacks de pagos cumplen con requirements

Ejemplo de test unitario:

```java
@SpringBootTest
class PaymentServiceClientTest {
    
    @MockBean
    private PaymentApiClient paymentApiClient;
    
    @Autowired
    private PaymentServiceClient paymentServiceClient;
    
    @Test
    void shouldOpenCircuitAfterMultipleFailures() {
        // Given
        when(paymentApiClient.process(any()))
            .thenThrow(new RuntimeException("Service unavailable"));
        
        // When & Then
        // First 5 calls should fail normally
        for (int i = 0; i < 5; i++) {
            assertThatThrownBy(() -> 
                paymentServiceClient.processPayment(createRequest()).get())
                .isInstanceOf(RuntimeException.class);
        }
        
        // 6th call should trigger circuit breaker
        CompletableFuture<PaymentResponse> response = 
            paymentServiceClient.processPayment(createRequest());
        
        // Then fallback should be used
        assertThat(response.get().getStatus()).isEqualTo("QUEUED");
    }
    
    @Test
    void shouldUseFallbackWhenCircuitIsOpen() {
        // Given
        CircuitBreakerRegistry registry = CircuitBreakerRegistry.ofDefaults();
        CircuitBreaker circuitBreaker = registry.circuitBreaker("paymentService");
        circuitBreaker.transitionToOpenState();
        
        // When
        CompletableFuture<PaymentResponse> response = 
            paymentServiceClient.processPayment(createRequest());
        
        // Then
        assertThat(response.get().getStatus()).isEqualTo("QUEUED");
        assertThat(response.get().getMessage())
            .isEqualTo("Payment will be processed later");
    }
}
```

### Fase 5: Monitoreo y Alertas Enterprise (Semana 6)
- Configurar métricas de Resilience4j con Micrometer
- Integrar con Prometheus para recolección de métricas
- Crear dashboards en Grafana con estado de circuit breakers
- Configurar alertas en ServiceNow cuando circuit breaker se abre frecuentemente
- Implementar logging estructurado de eventos de circuit breaker
- Integrar con Splunk para análisis de patrones de fallos

Configuración de métricas con Micrometer:

```java
@Configuration
public class MetricsConfig {
    
    @Bean
    public MeterRegistryCustomizer<MeterRegistry> metricsCommonTags() {
        return registry -> registry.config()
            .commonTags("application", "pos-service", 
                       "environment", "production");
    }
    
    @Bean
    public TaggedCircuitBreakerMeterRegistrar circuitBreakerMeterRegistrar() {
        return new TaggedCircuitBreakerMeterRegistrar(
            CircuitBreakerRegistry.ofDefaults());
    }
}
```

Dashboard de Grafana con paneles para:
- Estado de circuit breakers (closed, open, half-open)
- Tasa de fallos por servicio
- Tiempo de respuesta con y sin fallback
- Número de llamadas con fallback activado
- Alertas cuando circuit breaker se abre más de 3 veces en 1 hora

## Criterios de Éxito
- Reducción de 60% en incidentes de cascada
- Tiempo de recuperación automática < 60 segundos
- 0 caídas completas del sistema por fallos de servicios externos
- Visibilidad completa del estado de circuit breakers
- Overhead de latencia < 10ms
- Cumplimiento PCI-DSS mantenido en todos los fallbacks

## Consecuencias

### Beneficios Obtenidos

#### Técnicas
- ✅ **Prevención de fallos en cascada:** Servicios degradados no arrastran al sistema completo
- ✅ **Degradación elegante:** Sistema sigue funcionando con funcionalidad reducida
- ✅ **Recuperación automática:** Circuit breakers se cierran automáticamente cuando servicio se recupera
- ✅ **Timeouts configurables:** Cada servicio tiene SLA apropiado
- ✅ **Observabilidad:** Dashboard en tiempo real del estado de servicios (Grafana)
- ✅ **Overhead de latencia:** 5ms promedio (mejor que objetivo de 10ms)
- ✅ **Integración nativa** con stack Java/Spring Boot
- ✅ **Forward-compatible** con Java 11+ y Java 17

#### de Negocio
- ✅ **Reducción de 65% en incidentes** relacionados con servicios externos
- ✅ **Disponibilidad mejorada:** De 98.5% a 99.7%
- ✅ **Experiencia de usuario:** Clientes ven mensajes de "servicio temporalmente no disponible" en lugar de errores genéricos
- ✅ **Ahorro estimado:** $2.5M-3M USD en pérdidas por downtime evitado (año 1)
- ✅ **ROI positivo** inmediato ($2.5M beneficios vs. $50K implementación)
- ✅ **Cumplimiento PCI-DSS** mantenido en fallbacks de pagos

#### Organizacionales
- ✅ **Equipo de soporte** tiene visibilidad de qué servicio está fallando
- ✅ **Desarrolladores** implementan resiliencia de forma estandarizada
- ✅ **Cultura de resiliencia** se fortaleció en toda la organización
- ✅ **Conocimiento transferido** mediante pair programming y documentación

### Consideraciones y Trade-offs Gestionados

#### 1. Complejidad Adicional en Código de Servicios
**Consideración:** Cada servicio debe implementar circuit breakers, retry, fallbacks.

**Gestión implementada:**
- ✅ Biblioteca interna con configuraciones pre-definidas
- ✅ Templates de código para casos comunes
- ✅ Code reviews obligatorios para implementaciones de resiliencia
- ✅ Documentación con ejemplos específicos del dominio
- ✅ **Resultado:** Implementación estándar en 2-3 días por servicio

#### 2. Tuning de Umbrales
**Consideración:** Umbrales incorrectos pueden causar problemas (muy sensibles o muy laxos).

**Gestión implementada:**
- ✅ Empezar con valores conservadores basados en SLAs
- ✅ Revisión mensual de configuración basada en métricas reales
- ✅ A/B testing de diferentes configuraciones
- ✅ Alertas automáticas cuando umbrales necesitan ajuste
- ✅ **Resultado:** Umbrales optimizados en 3 meses

#### 3. Mantenimiento de Fallbacks
**Consideración:** Los fallbacks deben evolucionar con el negocio.

**Gestión implementada:**
- ✅ Documentar cada fallback con caso de uso y limitaciones
- ✅ Revisión trimestral de fallbacks con equipo de producto
- ✅ Tests automatizados para validar comportamiento de fallbacks
- ✅ Feature flags para activar/desactivar fallbacks
- ✅ **Resultado:** Fallbacks alineados con necesidades de negocio

#### 4. Testing de Escenarios de Fallo
**Consideración:** Testing debe simular fallos realistas.

**Gestión implementada:**
- ✅ WireMock para simular respuestas de servicios externos
- ✅ Chaos Monkey para introducir fallos aleatorios
- ✅ Ambiente de staging idéntico a producción
- ✅ Tests automatizados en CI/CD pipeline
- ✅ **Resultado:** Bugs de resiliencia detectados en staging, no en producción

#### 5. Cumplimiento PCI-DSS en Fallbacks
**Consideración:** Fallbacks de pagos deben cumplir con PCI-DSS.

**Gestión implementada:**
- ✅ Fallbacks no almacenan datos de tarjeta
- ✅ Encriptación end-to-end mantenida en fallbacks
- ✅ Auditoría de fallbacks por equipo de seguridad
- ✅ Documentación de cumplimiento para auditores
- ✅ **Resultado:** Cumplimiento PCI-DSS mantenido en todos los escenarios

## Lecciones Aprendidas

### 1. Circuit breakers no son opcionales en arquitecturas distribuidas
En sistemas distribuidos, los fallos son inevitables. Circuit breakers convierten fallos catastróficos en degradaciones manejables. Para un sistema con 19,556 sucursales, esto es crítico.

### 2. Los fallbacks son tan importantes como los circuit breakers
Un circuit breaker sin fallback solo previene cascadas, pero no mejora experiencia de usuario. Fallbacks bien diseñados permiten degradación elegante y mantienen al usuario informado.

### 3. La observabilidad es crítica
Sin métricas y alertas (Prometheus, Grafana, Splunk), no sabíamos cuándo los circuit breakers se activaban. Implementamos dashboard que muestra estado en tiempo real de todos los circuit breakers.

### 4. Tuning basado en datos, no en suposiciones
Empezamos con umbrales conservadores y ajustamos basado en métricas reales. Los umbrales incorrectos pueden ser peores que no tener circuit breakers.

### 5. Resilience4j es la elección correcta para Java
Hystrix estaba en mantenimiento, Sentinel tenía barreras de documentación, y construir custom no tenía ROI. Resilience4j es el estándar moderno para Java.

### 6. La inversión en resiliencia tiene ROI inmediato
$50K de implementación vs. $2.5M+ en beneficios evitados. En sistemas críticos, la resiliencia no es un gasto, es una inversión con retorno inmediato.

### 7. PCI-DSS debe considerarse en fallbacks
Los fallbacks no son "segunda clase". Deben cumplir con los mismos requisitos de seguridad que el flujo normal, especialmente en procesamiento de pagos.

## Referencias

- Resilience4j Documentation: https://resilience4j.readme.io/docs
- Spring Cloud Circuit Breaker: https://spring.io/projects/spring-cloud-circuitbreaker
- Circuit Breaker Pattern: https://martinfowler.com/bliki/CircuitBreaker.html
- Microsoft: Circuit Breaker Pattern: https://docs.microsoft.com/en-us/azure/architecture/patterns/circuit-breaker
- Bulkhead Pattern: https://docs.microsoft.com/en-us/azure/architecture/patterns/bulkhead
- PCI-DSS Requirements: https://www.pcisecuritystandards.org/

## Decisiones Relacionadas

- [ADR 001: Strangler Fig Migration](./001-strangler-fig-migration.md)
- [ADR 002: API Gateway Implementation](./002-api-gateway-implementation.md)
- [ADR 004: Cross-Team Alignment Framework](./004-cross-team-alignment.md)

---

**Autor:** Mario Guerrero  
**Fecha:** 2020-09  
**Revisado por:** 
- Equipo de Arquitectura
- Equipo de SRE (Site Reliability Engineering)
- Equipo de Operaciones
- Equipo de Seguridad (PCI-DSS)
- **Aprobado por Architecture Review Board**