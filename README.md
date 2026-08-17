# camacho-post1-u1
Post-contenido — Refactorización SOLID y análisis de patrones GoF en Spring

## Descripción
Repositorio del post-contenido de la Unidad 1 de Patrones de Diseño de Software — Sexto Semestre. Contiene dos partes: refactorización SOLID de un God Object (parte-1-refactorizacion-solid/) y análisis de patrones GoF en Spring Framework (parte-2-analisis-gof-spring/).

## Parte 1 — Refactorización SOLID
Proyecto Maven que refactoriza OrderProcessor aplicando SRP, OCP y DIP. Ver parte-1-refactorizacion-solid/.

# Sección Análisis de Violaciones SOLID — Parte 1

| Principio | Método/Sección afectada | Descripción de la violación |
|---|---|---|
| **SRP** | Clase OrderProcessor: calculateTotal(), applyDiscount(), saveOrder(), sendEmail() y printReport() | La clase OrderProcessor actúa como un *God Object* al acumular múltiples responsabilidades: cálculo del total, aplicación de descuentos, persistencia de órdenes, envío de notificaciones y generación de reportes. Por lo tanto, existen múltiples motivos independientes que podrían requerir modificaciones en la clase, lo que evidencia una violación del Principio de Responsabilidad Única. |
| **OCP** | Método applyDiscount() y condicionales sobre customerType | El método evalúa los tipos de cliente mediante estructuras condicionales como if (customerType.equals(...)). Si se incorpora un nuevo tipo de descuento, por ejemplo "ESTUDIANTE", es necesario modificar el método existente y agregar nuevas condiciones. Esto impide extender el comportamiento mediante polimorfismo sin modificar el código existente, evidenciando una violación del Principio Abierto/Cerrado. |
| **DIP** | Atributo orders y métodos saveOrder() y sendEmail() | OrderProcessor se acopla directamente a detalles concretos de infraestructura: utiliza ArrayList para almacenar las órdenes y System.out.println() para realizar la notificación. No existen abstracciones que permitan desacoplar estas responsabilidades de sus implementaciones concretas. Por ello, la clase de alto nivel depende directamente de detalles de bajo nivel, evidenciando una violación del Principio de Inversión de Dependencias. |


## Parte 2 — Análisis de Patrones GoF en Spring

| # | Patrón | Categoría | Clase en Spring |
|---|--------|-----------|-----------------|
| 1 | Singleton | Creacional | `org.springframework.beans.factory.support.DefaultSingletonBeanRegistry` |
| 2 | Decorator | Estructural | `org.springframework.web.util.ContentCachingRequestWrapper` |
| 3 | Template Method | Comportamiento | `org.springframework.jdbc.core.JdbcTemplate` |

Ver parte-2-analisis-gof-spring/documento-analisis.md.

## Herramientas utilizadas
- Java 17, Apache Maven, VS Code, Git, GitHub
- Código fuente de Spring Framework (investigación)

## Conclusiones
A través de la actividad pude comprender que una clase con múltiples responsabilidades puede generar un diseño difícil de mantener, por lo que la aplicación de los principios SOLID, especialmente SRP, OCP y DIP, permite obtener clases más cohesivas, extensibles y con menor acoplamiento. Asimismo, aprendí que los patrones de diseño GoF no necesariamente se aplican de manera idéntica a su definición original sino que pueden adaptarse a las necesidades específicas de un framework o problema de software. Finalmente, la actividad me permitió reconocer que SOLID y los patrones GoF son herramientas complementarias que contribuyen a construir sistemas más organizados, flexibles y fáciles de evolucionar.