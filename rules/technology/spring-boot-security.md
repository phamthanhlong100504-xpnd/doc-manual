---
description: Rules and templates for Spring Security, JWT, and MDC Trace ID implementation.
---

# Spring Security & Trace ID Standards

All microservices in the DTS system MUST implement a standardized security and tracing mechanism to ensure seamless integration with the API Gateway and Identity Service.

## 1. Trace ID (MDC)
- Every incoming HTTP request must be tagged with a unique `X-Request-ID`.
- If the header is missing, generate a new UUID.
- The ID must be stored in SLF4J's MDC (Mapped Diagnostic Context) using the key `traceId`.
- The `application.yaml` MUST include the following logging pattern:
  ```yaml
  logging:
    pattern:
      console: "%d{yyyy-MM-dd HH:mm:ss.SSS} %5p [%X{traceId}] --- [%15.15t] %-40.40logger{39} : %m%n"
  ```

## 2. JWT Authentication
- The JWT secret key MUST be loaded from `jwt.secret` in `application.yaml`, referencing the shared secret in `.agents/config/dts/environment/jwt.md`.
- `userId` MUST be extracted from the standard JWT `sub` claim.
- All granted authorities (permissions) extracted from the token MUST be prefixed with `PERM_`.
- Use `@PreAuthorize("hasAuthority('PERM_<permission_name>')")` on Controllers.

## 3. Implementation (Reusable Code)
When generating a new service, DO NOT WRITE SECURITY CODE FROM SCRATCH. 
You MUST copy the 5 standard template files located in `.agents/rules/templates/java/security/` into the service's `config` (or `security`) package, updating the `package` declaration to match the target service:
1. `JwtProperties.java`
2. `JwtProvider.java`
3. `JwtAuthenticationFilter.java`
4. `SecurityConfig.java`
5. `MdcTraceIdFilter.java`

These files contain the exact, battle-tested implementation that perfectly aligns with `dts-identity`.
