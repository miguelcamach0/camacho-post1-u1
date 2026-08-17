# Análisis de Patrones GoF en Spring Framework

## Portada

* **Nombre del Estudiante:** Miguel Camacho
* **Código:** 02240131046
* **Curso:** Patrones de Diseño de Software B-2026
* **Unidad:** Unidad 1 — Refactorización SOLID y Patrones GoF
* **Fecha:** 16 de agosto de 2026

---

# Introducción
Los patrones de diseño GoF constituyen soluciones reutilizables para problemas recurrentes en el diseño de sistemas orientados a objetos. Su aplicación permite organizar responsabilidades, reducir el acoplamiento y facilitar la extensión y mantenimiento del software. En este contexto, el presente análisis toma como caso de estudio el ecosistema de Spring, particularmente el código fuente de Spring Framework, con el propósito de identificar la aplicación de tres patrones pertenecientes a categorías GoF diferentes: Singleton, de tipo creacional; Decorator, de tipo estructural; y Template Method, de tipo comportamental. La selección de estos patrones permite observar cómo diferentes estrategias de diseño pueden utilizarse para resolver problemas relacionados con la gestión de instancias, la extensión de funcionalidades y la reutilización de flujos de ejecución (Gamma et al., 1995).

Spring Framework constituye la base sobre la cual se desarrolla gran parte del ecosistema Spring y proporciona mecanismos para la construcción de aplicaciones empresariales mediante funcionalidades como la gestión de objetos, la inyección de dependencias, el acceso a datos y el procesamiento de solicitudes web. Spring Boot, por su parte, se construye sobre Spring Framework y facilita la configuración y puesta en marcha de aplicaciones basadas en este ecosistema. Por esta razón, el análisis se concentra en componentes concretos del código fuente de Spring Framework, verificando mediante evidencia directa cómo `DefaultSingletonBeanRegistry`, `ContentCachingRequestWrapper` y `JdbcTemplate` emplean estructuras asociadas con los patrones seleccionados. Asimismo, se analiza la relación de estos patrones con principios SOLID y las consecuencias que tendría prescindir de estas soluciones de diseño.

---

# Análisis de Patron 1: Singleton

**Patrón y categoría GoF.** 
El patrón identificado es Singleton, perteneciente a la categoría de patrones creacionales de GoF. Su propósito general es garantizar que exista una única instancia de un objeto dentro de un ámbito determinado y proporcionar un mecanismo controlado para acceder a ella (Gamma et al., 1994). En Spring Framework, esta idea se implementa mediante el singleton scope, aunque con una diferencia importante respecto al Singleton clásico de GoF: Spring garantiza una instancia por bean y por contenedor IoC, no una instancia global de una clase para todo el sistema (Spring Framework, s. f.).

**Ubicación en Spring Framework.**
La implementación analizada se encuentra en la clase *org.springframework.beans.factory.support.DefaultSingletonBeanRegistry*, perteneciente al módulo "spring-beans". Esta clase implementa la interfaz *org.springframework.beans.factory.config.SingletonBeanRegistry* y funciona como un registro genérico de instancias compartidas. 

**Problema que resuelve.** 
En un contenedor IoC como Spring, diferentes componentes pueden requerir el mismo bean como dependencia. Crear una nueva instancia cada vez que otro componente lo solicita produciría duplicación innecesaria de objetos y dificultaría la administración de su ciclo de vida. El registro de singletons proporciona un punto centralizado para almacenar y recuperar las instancias compartidas mediante su nombre.

Además, esta solución permite que Spring administre el ciclo de vida de dichas instancias. "DefaultSingletonBeanRegistry" mantiene información sobre los singletons registrados y también permite gestionar beans que deben destruirse durante el cierre del registro, incluyendo sus dependencias para establecer un orden apropiado de destrucción.

Al analizar, se muestra que, sin este mecanismo, el contenedor tendría que resolver manualmente la reutilización de instancias, aumentando el código necesario para administrar dependencias y dificultando el control centralizado de su ciclo de vida.

**Evidencia de código.**
La implementación puede observarse en el método "addSingleton", localizado en "DefaultSingletonBeanRegistry". A continuación el fragmento que constituye la evidencia principal:

```java
protected void addSingleton(String beanName, Object singletonObject) {
    //Registra el bean solo si el nombre no existe en el mapa principal (evitar sobrescribir), garantizando la unicidad del objeto.
    Object oldObject = this.singletonObjects.putIfAbsent(beanName, singletonObject);

    //Si ya existía una instancia previa bajo el mismo nombre, lanza un error de estado.
    if (oldObject != null) {
        throw new IllegalStateException("Could not register object [" + singletonObject +
                "] under bean name '" + beanName + "': there is already object [" + oldObject + "] bound");
    }

    // Si no, limpia referencias temporales usadas durante la resolución de dependencias circulares.
    this.singletonFactories.remove(beanName);
    this.earlySingletonObjects.remove(beanName);

    //Registra el nombre del bean en el conjunto oficial de singletons completamente creados.
    this.registeredSingletons.add(beanName);
}
```

**Principio SOLID reforzado.**
La relación más directa se establece con el Principio de Responsabilidad Única (SRP). DefaultSingletonBeanRegistry concentra la responsabilidad de administrar el registro y ciclo de vida de las instancias compartidas, sin asumir la lógica de negocio de los objetos registrados ni un mecanismo específico para crearlos.

También puede establecerse una relación con el Principio de Inversión de Dependencias (DIP) aunque debe presentarse como una relación estructural y no como una consecuencia directa del patrón Singleton. Spring define la interfaz SingletonBeanRegistry como una abstracción para exponer la administración de instancias compartidas, mientras que DefaultSingletonBeanRegistry proporciona una implementación concreta. Esto permite separar el contrato de gestión de los singletons de su implementación.

---

# Análisis de Patron 2: Decorator

**Patrón y categoría GoF.** 
El patrón identificado es Decorator, perteneciente a la categoría de patrones estructurales de GoF. Su propósito consiste en agregar responsabilidades o comportamiento a un objeto de manera flexible, sin modificar directamente su implementación original. Para ello, el objeto decorador mantiene una interdaz compatible con el componente que envuelve, permitiendo que los clientes continúen utilizándolo de manera transparente (Gamma et al., 1994).

A diferencia de una solución basada exclusivamente en herencia, el patrón Decorator permite combinar responsabilidades mediante composición y delegación.

**Ubicación en Spring Framework.** 
La aplicación analizada se encuentra en la clase: *org.springframework.web.util.ContentCachingRequestWrapper* perteneciente al módulo "spring-web" de Spring Framework. 

Esta clase extiende: *jakarta.servlet.http.HttpServletRequestWrapper*, La clase "HttpServletRequestWrapper" pertenece a la API Servlet y proporciona la infraestructura necesaria para envolver una solicitud HTTP. Sobre esta estructura *ContentCachingRequestWrapper* incorpora funcionalidad adicional propia de Spring relacionada con el almacenamiento del contenido de la petición.

La clase recibe un objeto "HttpServletRequest" y conserva la
compatibilidad con el contrato de la solicitud original, mientras
agrega capacidades relacionadas con la caché del contenido que se
va leyendo.

**Problema que resuelve.** 
En el procesamiento de solicitudes HTTP, el contenido recibido mediante el flujo de entrada puede ser consumido durante la ejecución de la aplicación. En determinadas situaciones, componentes del framework, como mecanismos de registro o auditoría, necesitan conservar el contenido que ya fue leído para poder consultarlo posteriormente.

"ContentCachingRequestWrapper" resuelve esta necesidad envolviendo la solicitud HTTP original y agregando un mecanismo de almacenamiento del contenido leído. De esta manera, la clase no necesita modificar la implementación original de "HttpServletRequest", sino que añade la capacidad de conservar los datos mediante una capa adicional.

Esta solución proporciona mayor flexibilidad que modificar directamente la implementación de la solicitud, ya que la capacidad de almacenamiento puede incorporarse únicamente cuando resulta necesaria. Sin este mecanismo, funcionalidades como el registro del contenido de una petición tendrían que incorporarse directamente en las implementaciones de las solicitudes o mediante diferentes clases especializadas.

**Evidencia de código.** 
La aplicación del patrón puede observarse en la clase "ContentCachingRequestWrapper", particularmente en la forma en que recibe y envuelve una solicitud HTTP existente y agregar estructuras internas para almacenar su contenido. He aquí un fragmento representativo del código fuente de Spring Framework:

```java
//Hereda del Decorator base de Servlet para envolver y extender HttpServletRequest.
public class ContentCachingRequestWrapper
        extends HttpServletRequestWrapper {

    //Memoria intermedia interna en memoria para almacenar el cuerpo de la petición retenido.
    private final FastByteArrayOutputStream cachedContent;

    //Constructor que recibe la petición original a envolver y el límite de caché.
    public ContentCachingRequestWrapper(
            HttpServletRequest request, int cacheLimit) {

        //Pasa la petición original a la clase base (Mantiene su contrato de la interfaz)
        super(request);

        //Obtiene el tamaño del cuerpo de la petición HTTP en bytes
        int contentLength = request.getContentLength();

        
        //Inicializa la memoria intermedia optimizando la moeria según la longitud declarada y el límite.
        this.cachedContent =
                (contentLength > 0
                ? new FastByteArrayOutputStream(
                    (cacheLimit > 0
                    ? Math.min(contentLength, cacheLimit)
                    : contentLength))
                : new FastByteArrayOutputStream());
    }
}

```

**Principio SOLID reforzado.**
La relación más directa entre el patrón Decorator y los principios SOLID en este contexto corresponde al Principio de Abierto/Cerrado (OCP). Rajput (2017) explica la utilización del patrón Decorator dentro del desarrollo con Spring y muestra cómo este patrón permite incorporar responsabilidades adicionales sobre un componente mediante una estructura decoradora. Esta característica se relaciona con OCP, debido a que el comportamiento puede extenderse sin modificar directamente la implementación del componente original.

En "ContentCachingRequestWrapper", esta relación se observa porque Spring incorpora la capacidad de almacenar el contenido leído de un
"HttpServletRequest" mediante un objeto envolvente. De esta manera, la
funcionalidad original de la solicitud no necesita modificarse para
incorporar esta capacidad adicional.

De manera complementaria, puede establecerse una relación complementaria con el Principio de Responsabilidad Única (SRP). La responsabilidad adicional relacionada con el almacenamiento del contenido se mantiene en ContentCachingRequestWrapper, evitando que la implementación original de la solicitud tenga que incorporar directamente esta funcionalidad.

---

# Análisis de Patron 3: Template Method

**Patrón y categoría GoF.**
El patrón identificado es Template Method, perteneciente a la categoría de patrones de comportamiento. Su propósito consiste en definir la estructura general de un algoritmo, manteniendo estables sus pasos principales y permitiendo que determinadas operaciones sean proporcionadas o redefinidas de manera específica. Gamma et al. (1994) incluyen Template Method dentro de los patrones comportamentales y lo presentan como una técnica para reutilizar la estructura de un algoritmo mientras se permiten variaciones en algunos de sus pasos.

En Spring Framework, este concepto se presenta mediante una adaptación basada en "callbacks". En lugar de utilizar exclusivamente herencia para redefinir métodos, como ocurre en la formulación clásica del patrón, "JdbcTemplate" mantiene el flujo común de acceso a JDBC y delega la operación específica a objetos que implementan interfaces callback. Rajput (2017) relaciona el patrón Template Method con la reutilización de código y la reducción de código repetitivo en aplicaciones basadas en Spring.

**Ubicación en Spring Framework.**
La implementación analizada se encuentra en la clase: *org.springframework.jdbc.core.JdbcTemplate*, esta clase pertenece al módulo "spring-jdbc" de Spring Framework y se encuentra en: *spring-jdbc/src/main/java/org/springframework/jdbc/core/JdbcTemplate.java*

"JdbcTemplate" proporciona una abstracción sobre el flujo habitual de JDBC. La documentación de la propia clase indica que centraliza el flujo principal de JDBC, mientras permite que el código de aplicación proporcione el SQL y las operaciones específicas mediante diferentes interfaces callback. Entre estas interfaces se encuentran "StatementCallback", "PreparedStatementCallback", "ResultSetExtractor" y "RowMapper".

**Problema que resuelve.**
El principal problema que resuelve "JdbcTemplate" es la repetición de código asociado con las operaciones tradicionales de JDBC. Una operación JDBC requiere normalmente obtener una conexión, crear y configurar un "Statement", ejecutar una operación, manejar advertencias y excepciones y finalmente liberar los recursos utilizados.

La centralización de este flujo permite que las operaciones concretas no tengan que repetir la misma infraestructura. "JdbcTemplate" mantiene estos pasos comunes en sus métodos de ejecución y permite que la operación específica sea proporcionada mediante callbacks. De esta manera, el código de aplicación puede concentrarse en la operación que necesita realizar, mientras el framework administra los aspectos repetitivos de infraestructura.

**Evidencia de código.**
La aplicación del patrón puede observarse en el método privado "execute(StatementCallback<T> action, boolean closeResources)" de "JdbcTemplate". Este método establece el flujo común de una operación JDBC:

```java

//MÉTODO PLANTILLA: Define el algoritmo fijo de gestión de recursos y ejecución JDBC
private <T extends @Nullable Object> T execute(StatementCallback<T> action, boolean closeResources) throws DataAccessException {

    //Valida precondiciones: garantizar que se recibió una estrategia/callback válida
    Assert.notNull(action, "Callback object must not be null");

    //Obtiene conexión limpia a la base de datos desde el DataSource centralizado
    Connection con = DataSourceUtils.getConnection(obtainDataSource());
    Statement stmt = null;
    try {
        //Crea el Statement y aplicar configuraciones globales (ej. timeouts, maxRows)
        stmt = con.createStatement();
        applyStatementSettings(stmt);
        
        //Delega la lógica SQL específica al callback inyectado
        T result = action.doInStatement(stmt);
        
        //Procesa advertencias SQL de forma estandarizada
        handleWarnings(stmt);
        return result;
    }

    ...

    finally {
        //Garantiza la liberación segura de recursos en cualquier escenario
        if (closeResources) {
            JdbcUtils.closeStatement(stmt);
            DataSourceUtils.releaseConnection(con, getDataSource());
        }
    }
}
```
Esta estructura permite que el flujo común permanezca bajo responsabilidad de JdbcTemplate, independientemente de la operación específica que se ejecute.

La relación entre el algoritmo general y una operación concreta puede observarse en el método update(String sql). Este método crea un UpdateStatementCallback y posteriormente lo proporciona al método execute():

```java
//CALLBACK / ESTRATEGIA CONCRETA: Implementa el paso variable para operaciones UPDATE/INSERT/DELETE
class UpdateStatementCallback implements StatementCallback<Integer>, SqlProvider {
    @Override
    public Integer doInStatement(Statement stmt) throws SQLException {
        //Ejecuta la sentencia SQL específica de actualización y retorna el número de filas afectadas
        int rows = stmt.executeUpdate(sql);
        if (logger.isTraceEnabled()) {
            logger.trace("SQL update affected " + rows + " rows");
        }
        return rows;
    }

    @Override
    public String getSql() {
        // Proporciona el SQL original para fines de logging y traducción de excepciones
        return sql;
    }
}

//Conecta la estructura fija del algoritmo con el comportamiento variable
return updateCount(execute(new UpdateStatementCallback(), true));
```

En este caso, execute() conserva el flujo general, mientras UpdateStatementCallback proporciona la operación específica mediante doInStatement(). La llamada execute(new UpdateStatementCallback(), true) conecta ambas partes: el algoritmo común recibe el comportamiento concreto que debe ejecutar.

**Principio SOLID reforzado.**
La implementación presenta una relación principalmente con el Principio de Abierto/Cerrado (OCP). El flujo general definido por JdbcTemplate permanece estable, mientras diferentes operaciones pueden incorporarse mediante callbacks como StatementCallback sin modificar la lógica central de adquisición y liberación de recursos. Esta separación permite extender el comportamiento disponible manteniendo estable la infraestructura común.

También existe una relación complementaria con el Principio de Responsabilidad Única (SRP). JdbcTemplate concentra la gestión del flujo común de JDBC, mientras que los callbacks proporcionan las operaciones específicas que deben ejecutarse. De esta manera, la gestión de recursos y errores se mantiene separada de la lógica particular de cada operación.

Sin embargo, estos principios no deben considerarse una consecuencia automática del patrón Template Method. La relación con OCP y SRP se deriva de la forma concreta en que Spring estructura JdbcTemplate y sus callbacks.

---

## Conclusiones

El análisis realizado me permitió comprobar que los patrones GoF no constituyen únicamente soluciones teóricas, sino herramientas de diseño que pueden integrarse de manera sistemática en frameworks de gran escala como Spring Framework. El patrón Singleton permite centralizar la gestión de instancias compartidas mediante `DefaultSingletonBeanRegistry`; Decorator permite extender el comportamiento de objetos existentes mediante componentes como `ContentCachingRequestWrapper`; y Template Method permite mantener flujos de ejecución comunes mientras delega operaciones específicas, como ocurre con `JdbcTemplate` y sus callbacks. En conjunto, los tres casos evidencian una relación directa entre patrones de diseño y principios solid, permitiendo reducir acoplamiento y duplicaciones. Asimismo, gracias al análisis pude aprender que un framework puede adaptar un patrón clásico a sus propias necesidades estructurales, como ocurre con el uso de composición y callbacks en la implementación de Template Method. Desde la perspectiva del diseño propio, estos casos muestran la importancia de identificar primero las responsabilidades, los puntos de variación, las necesidades reales y las dependencias de un sistema antes de seleccionar un patrón, evitando aplicarlos de manera mecánica y utilizándolos como herramientas para construir código más cohesivo, extensible y mantenible.

## Referencias
Gamma, E., Helm, R., Johnson, R., & Vlissides, J. (1994). *Design patterns: Elements of reusable object-oriented software*. Addison-Wesley Professional.

Rajput, D. (2017). *Spring 5 design patterns: Master efficient application development with patterns such as proxy, singleton, the template method, and more*. Packt Publishing.

Spring Framework. (s. f.). *DefaultSingletonBeanRegistry*. Spring Framework API Documentation. https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/support/DefaultSingletonBeanRegistry.html

Spring Framework. (s. f.). *ContentCachingRequestWrapper*. Spring Framework API Documentation. https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/util/ContentCachingRequestWrapper.html

Spring Framework. (s. f.). *JdbcTemplate*. Spring Framework API Documentation. https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html


