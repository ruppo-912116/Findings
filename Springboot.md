## 🔄 Lifecycle of @PreAuthorize and StreamingResponseBody
✅ What happens first:
Spring's security filters (like FilterSecurityInterceptor) and @PreAuthorize evaluate before your controller method is invoked.

If the user lacks permission, the method is never entered.

⚠️ However, in your case:
Your controller returns a StreamingResponseBody, which defers actual logic execution (like reading from S3 and writing to the response) until after the method has already returned a ResponseEntity.

That means:

Security is checked → ✅ access is granted

Spring MVC calls your controller → ✅ you return a StreamingResponseBody

Spring commits headers → ❗ (this is key — headers are flushed)

Spring starts executing the lambda in StreamingResponseBody

Something goes wrong inside that lambda (e.g. exception)

Spring Security tries to handle the exception, but the response is already committed → 💥 you get:
