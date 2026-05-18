# F-12: Error Pages Disabled in Production

## What was observed

The `web.xml` for both CPL and 试玩测试/豆豆赚 has all error page definitions commented out:

```xml
<!-- <error-page>
    <exception-type>java.lang.Throwable</exception-type>
    <location>/WEB-INF/pages/error/500.jsp</location>
</error-page>
<error-page>
    <error-code>500</error-code>
    <location>/WEB-INF/pages/error/500.jsp</location>
</error-page>
<error-page>
    <error-code>404</error-code>
    <location>/WEB-INF/pages/error/404.jsp</location>
</error-page>
<error-page>
    <error-code>403</error-code>
    <location>/WEB-INF/pages/error/403.jsp</location>
</error-page> -->
```

With these commented out, unhandled exceptions in the application return a raw Tomcat or Spring error response directly to the client, including the full Java stack trace.

## Why this matters

Stack traces in production responses expose internal package names, class names, method signatures, file paths on the server filesystem, dependency versions, and the exact line of code where an error occurred. This gives an observer a detailed map of the application's internal structure without needing source code access.

Given that the source code is already exposed via the workspace (F-15) and the repository (F-04), this is a lower-priority finding in the current context. However it is worth fixing independently of the broader remediation because it lowers the bar for reconnaissance against this application even after secrets are rotated and the CI system is secured.

## What needs to change

- Uncomment the error page definitions in `web.xml` for all three applications.
- Ensure the error JSP pages themselves do not expose any internal detail in their response body.
- Consider adding a global Spring `@ControllerAdvice` exception handler that logs the full exception server-side while returning a generic error response to the client.
