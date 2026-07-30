---
trigger: always_on
---

# Exception Template

## Purpose

Generate custom exceptions and global exception handlers.

---

## Template

```java
public class XxxException extends RuntimeException {

    public XxxException(String message) {
        super(message);
    }

}
```

```java
// ErrorResponse.java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class ErrorResponse {
    private int status;
    private String error;
    private String message;
    private String traceId;
}
```

```java
// GlobalExceptionHandler.java
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(BusinessValidationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleBusinessValidationException(BusinessValidationException ex) {
        log.warn("Business validation failed: {}", ex.getMessage());
        return buildErrorResponse(HttpStatus.BAD_REQUEST, "Bad Request", ex.getMessage());
    }

    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleResourceNotFoundException(ResourceNotFoundException ex) {
        log.warn("Resource not found: {}", ex.getMessage());
        return buildErrorResponse(HttpStatus.NOT_FOUND, "Not Found", ex.getMessage());
    }

    @ExceptionHandler(AccessDeniedException.class)
    @ResponseStatus(HttpStatus.FORBIDDEN)
    public ErrorResponse handleAccessDeniedException(AccessDeniedException ex) {
        log.warn("Access denied: {}", ex.getMessage());
        return buildErrorResponse(HttpStatus.FORBIDDEN, "Forbidden", ex.getMessage());
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGenericException(Exception ex) {
        log.error("Unexpected server error occurred", ex);
        return buildErrorResponse(HttpStatus.INTERNAL_SERVER_ERROR, "Internal Server Error", "An unexpected error occurred. Please contact support.");
    }

    private ErrorResponse buildErrorResponse(HttpStatus status, String error, String message) {
        return ErrorResponse.builder()
                .status(status.value())
                .error(error)
                .message(message)
                .traceId(MDC.get("traceId"))
                .build();
    }
}
```

---

# Required Members

- Exception Class
- Error Message
- Error Code (Optional)
- Global Exception Handler

---

# Standard Responsibilities

- Represent business errors
- Represent system errors
- Provide meaningful error messages
- Centralize exception handling

---

# Rules

## MUST

### EXCEPTION-001

Extend `RuntimeException` for business exceptions.

### EXCEPTION-002

Use meaningful exception names.

### EXCEPTION-003

Provide clear error messages.

### EXCEPTION-004

Handle exceptions using `@RestControllerAdvice`.

### EXCEPTION-005

Map exceptions to appropriate HTTP status codes.

### EXCEPTION-006

Keep exception classes lightweight.

---

## MUST NOT

### EXCEPTION-007

Do not catch generic `Exception` unless necessary.

### EXCEPTION-008

Do not swallow exceptions silently.

### EXCEPTION-009

Do not expose stack traces to clients.

### EXCEPTION-010

Do not place business logic inside exception classes.

---

# Checklist

- [ ] Custom Exception
- [ ] Clear Message
- [ ] Proper HTTP Status
- [ ] Global Handler
- [ ] No Business Logic