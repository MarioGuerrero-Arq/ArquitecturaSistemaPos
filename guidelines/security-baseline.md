# Security Baseline & Zero-Trust Architecture

## 📋 Propósito

Este documento establece la línea base de seguridad (Security Baseline) bajo un modelo **Zero-Trust** para nuestras arquitecturas enterprise. Este baseline es de cumplimiento obligatorio para todos los microservicios, APIs y componentes de infraestructura, asegurando el cumplimiento de normativas críticas como **PCI-DSS**, SOX y políticas internas de seguridad corporativa.

**Objetivos:**
- Implementar el principio de "Nunca confiar, siempre verificar" (Zero-Trust).
- Garantizar el cumplimiento de PCI-DSS para procesamiento de pagos.
- Proteger datos sensibles (PII, datos de tarjetas) en tránsito y en reposo.
- Estandarizar la gestión de identidades, secretos y cifrado.

---

## 🛡️ Principios Zero-Trust

### 1. Verify Explicitly
Siempre autenticar y autorizar basándose en todos los puntos de datos disponibles (identidad del usuario, estado del dispositivo, ubicación, anomalías de red).

### 2. Use Least Privilege Access
Limitar el acceso de usuarios y servicios estrictamente a lo necesario para su función (Just-In-Time y Just-Enough-Access).

### 3. Assume Breach
Diseñar la arquitectura asumiendo que la red interna ya está comprometida. Segmentar redes, encriptar todo y monitorear continuamente.

---

## 🔐 Identidad y Gestión de Accesos (IAM)

### Autenticación de Usuarios (Human Identity)
**Estándar:** OAuth 2.0 + OpenID Connect (OIDC) mediante Entra ID (Azure AD) o proveedor corporativo equivalente.

**Reglas:**
- ✅ **MFA Obligatorio:** Multi-Factor Authentication requerido para todos los accesos a paneles administrativos y sistemas críticos.
- ✅ **SSO (Single Sign-On):** Todas las aplicaciones internas deben integrar SSO.
- ❌ **Prohibido:** Autenticación básica (Basic Auth), credenciales hardcodeadas, o sesiones persistentes sin expiración.

### Autenticación de Servicios (Machine Identity)
**Estándar:** Managed Identities (cuando aplica en Cloud) o Certificados X.509 / Client Credentials Flow (OAuth2) para comunicaciones service-to-service.

**Reglas:**
- ✅ Usar **Managed Identities** para acceso a bases de datos (Oracle), Key Vaults y colas (Service Bus/RabbitMQ).
- ✅ Rotación automática de secretos cada 90 días mediante Azure Key Vault / HashiCorp Vault.
- ❌ **Prohibido:** Cadenas de conexión con passwords en `application.properties` o variables de entorno no encriptadas.

### Autorización (RBAC y ABAC)
- **RBAC (Role-Based Access Control):** Para permisos a nivel de aplicación (ej. `ROLE_ADMIN`, `ROLE_CASHIER`).
- **ABAC (Attribute-Based Access Control):** Para decisiones contextuales (ej. "El cajero solo puede procesar pagos si la tienda es la asignada y el monto es < $5,000").
- Los tokens JWT deben contener los scopes/roles mínimos necesarios (`scope: "read:inventory"`).

---

## 🌐 Seguridad de APIs (API Gateway)

Todo el tráfico externo e interno entre dominios de negocio debe pasar por el **API Gateway (Kong Enterprise)**.

### Plugins de Seguridad Obligatorios en Kong
1. **Authentication:** `jwt` o `oauth2` para validar tokens en el edge.
2. **Rate Limiting:** Para prevenir abusos y ataques DDoS (configurado por consumer/IP).
3. **IP Restriction:** Whitelisting de IPs para APIs administrativas o de terceros.
4. **CORS:** Configuración estricta (no usar `*` en producción).
5. **Bot Detection / WAF:** Integración con Web Application Firewall para bloquear patrones de inyección.

### Protección de Endpoints
- **Validación de Input:** El API Gateway debe rechazar payloads que excedan límites de tamaño (ej. max 1MB) o que contengan patrones XSS/SQLi básicos.
- **Headers de Seguridad:** Kong debe inyectar automáticamente:
  ```http
  Strict-Transport-Security: max-age=31536000; includeSubDomains
  X-Content-Type-Options: nosniff
  X-Frame-Options: DENY
  Content-Security-Policy: default-src 'self'
  ```

---

## 🔒 Protección de Datos y PCI-DSS

### Clasificación de Datos
| Nivel | Ejemplo | Requerimiento de Cifrado |
|-------|---------|--------------------------|
| **Confidencial (PCI)** | PAN (Número de tarjeta), CVV, PIN | Cifrado en tránsito (TLS 1.2+) y reposo (AES-256). Tokenización obligatoria. |
| **Secreto** | Passwords, API Keys, Tokens | Cifrado en reposo. Hashing (bcrypt/Argon2) para passwords. |
| **PII (Personal)** | Nombre, Email, Teléfono, RFC | Cifrado en reposo. Mascarado en logs. |
| **Interno** | IDs internos, configs no sensibles | Cifrado en tránsito. |

### Reglas de Oro para PCI-DSS
1. **NUNCA almacenar CVV/CVC** después de la autorización.
2. **NUNCA loggear datos de tarjeta** (PAN debe estar enmascarado, ej. `**** **** **** 1234`).
3. **Tokenización:** Usar un proveedor certificado (ej. Stripe, Adyen, o Tokenizador interno) para reemplazar el PAN con un token.
4. **Segmentación de Red:** El entorno donde se procesan datos de tarjeta (CDE - Cardholder Data Environment) debe estar aislado en subredes específicas con firewalls estrictos.

### Cifrado en Tránsito y Reposo
- **Tránsito:** TLS 1.2 o superior. Cipher suites fuertes (AES-GCM, ECDHE).
- **Reposo:** 
  - Oracle Database: TDE (Transparent Data Encryption) habilitado para tablespaces sensibles.
  - Object Storage / Discos: Cifrado automático con claves gestionadas (KMS).

---

## 🗝️ Gestión de Secretos

**Estándar:** Azure Key Vault / HashiCorp Vault.

**Flujo de obtención de secretos:**
1. El microservicio arranca sin secretos en su configuración.
2. Al iniciar, usa su Managed Identity / Certificado para autenticarse contra el Key Vault.
3. Descarga los secretos en memoria (o los inyecta vía sidecar/CSI driver).
4. **Rotación:** Los secretos en el Vault se rotan automáticamente. El microservicio debe soportar recarga de configuración (refresh tokens/keys) sin reinicio.

❌ **Prohibido:**
- Subir secretos a repositorios de código (Git).
- Usar variables de entorno en claro en manifiestos de Kubernetes.
- Enviar secretos por Slack, Teams o correo electrónico.

---

## 🏗️ Seguridad en Infraestructura y Kubernetes

### Hardening de Kubernetes
- **RBAC de K8s:** Principio de menor privilegio para los ServiceAccounts.
- **Network Policies:** Obligatorio definir políticas de red. Un pod solo puede comunicarse con los pods que necesita (default deny all).
- **Pod Security Standards:** Ejecutar pods como `non-root`. Read-only root filesystem.
- **Secrets en K8s:** No usar `Secrets` nativos de K8s en plano. Usar integración con External Secrets Operator (conectado a Key Vault).

### Escaneo de Vulnerabilidades
- **Imágenes de Docker:** Escaneo obligatorio en el pipeline CI/CD (ej. Trivy, Snyk) antes de subir al registro. Bloquear despliegue si hay vulnerabilidades CRITICAL o HIGH.
- **Dependencias:** Escaneo de dependencias (OWASP Dependency-Check, Snyk Open Source) en cada PR.

---

## 📊 Observabilidad y Monitoreo de Seguridad

### Logging de Seguridad
Todo evento de seguridad debe ser loggeado en formato JSON estructurado y enviado a **Splunk**:
- Intentos de autenticación fallidos.
- Cambios de permisos o roles.
- Acceso a datos clasificados como Confidencial/PCI.
- Errores de autorización (401, 403).

### Métricas y Alertas (P1 - Críticas)
- **Pico de 401/403:** Posible ataque de fuerza bruta o token comprometido.
- **Acceso desde IP/Geolocalización anómala.**
- **Modificación de reglas de firewall o Network Policies.**
- **Intentos de inyección (detectados por WAF).**

### Auditoría
- Habilitar **Cloud Audit Logs** / **Oracle Audit Vault** para rastrear quién hizo qué y cuándo a nivel de base de datos e infraestructura.
- Los logs de auditoría deben ser inmutables (write-once) y retenidos por al menos 1 año (o según política legal).

---

## ✅ Checklist de Seguridad para Go-Live

Antes de que cualquier servicio pase a producción, el equipo de seguridad debe validar:

### Identidad y Acceso
- [ ] Autenticación vía OIDC/OAuth2 integrada.
- [ ] MFA habilitado para accesos administrativos.
- [ ] Roles y permisos revisados (Least Privilege).

### Datos y Cifrado
- [ ] TLS 1.2+ en todos los endpoints.
- [ ] Base de datos con TDE activado (si aplica).
- [ ] Cero datos PCI/PII en logs (validado con sampleo).
- [ ] Secretos gestionados vía Key Vault (no en código/config).

### API y Red
- [ ] Tráfico pasando por API Gateway (Kong).
- [ ] Rate Limiting y WAF configurados.
- [ ] Network Policies de Kubernetes definidas (Default Deny).
- [ ] CORS configurado restrictivamente.

### CI/CD y Despliegue
- [ ] Escaneo de vulnerabilidades de imagen y dependencias aprobado.
- [ ] El contenedor corre como `non-root`.
- [ ] Health checks configurados sin exponer datos sensibles.

### Monitoreo
- [ ] Logs de seguridad enviados a Splunk.
- [ ] Alertas de seguridad configuradas en ServiceNow/PagerDuty.
- [ ] Auditoría de base de datos activada.

---

## 📚 Referencias y Normativas

- [PCI-DSS v4.0 Requirements](https://www.pcisecuritystandards.org/document_library)
- [NIST Zero Trust Architecture (SP 800-207)](https://csrc.nist.gov/publications/detail/sp/800-207/final)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [Kubernetes Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Kubernetes_Security_Cheat_Sheet.html)

---

**Última actualización:** 2025-01-15  
**Versión:** 1.0  
**Mantenido por:** Information Security Guild  
**Aprobado por:** CISO & Architecture Review Board