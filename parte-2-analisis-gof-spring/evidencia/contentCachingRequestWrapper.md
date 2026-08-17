# Evidencia — Patrón Decorator en Spring Framework

## Identificación

- **Patrón:** Decorator
- **Categoría GoF:** Estructural
- **Clase:** `org.springframework.web.util.ContentCachingRequestWrapper`
- **Módulo:** `spring-web`
- **Archivo fuente:** `spring-web/src/main/java/org/springframework/web/util/ContentCachingRequestWrapper.java`
- **Clase envuelta:** `jakarta.servlet.http.HttpServletRequest`
- **Clase base:** `jakarta.servlet.http.HttpServletRequestWrapper`
- **Líneas de código:** `57-84`
- **Métodos analizados:** `ContentCachingRequestWrapper(...)`, `getInputStream()` y `getContentAsByteArray()`

## Fragmento de código fuente

```java
public class ContentCachingRequestWrapper extends HttpServletRequestWrapper {

    private final FastByteArrayOutputStream cachedContent;

    public ContentCachingRequestWrapper(
            HttpServletRequest request, int cacheLimit) {

        super(request);

        int contentLength = request.getContentLength();

        this.cachedContent = (contentLength > 0
                ? new FastByteArrayOutputStream(
                    (cacheLimit > 0
                    ? Math.min(contentLength, cacheLimit)
                    : contentLength))
                : new FastByteArrayOutputStream());

        this.contentCacheLimit = (cacheLimit > 0 ? cacheLimit : null);
    }
}