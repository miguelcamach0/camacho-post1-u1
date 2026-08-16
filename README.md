# camacho-post1-u1
Post-contenido — Refactorización SOLID y análisis de patrones GoF en Spring

# Sección Análisis de Violaciones SOLID — Parte 1

| Principio |           Método/Sección afectada          |              Descripción de la violación                     |
|-----------|--------------------------------------------|--------------------------------------------------------------|
| SRP       | Clase OrderProcessor y métodos:            | La clase OrderProccesor actúa como un *God Object* al        |
|           | calculateTotal+applyDiscount+saveOrder+    | acumular múltiples responsabilidades: cálculo total, aplicar |
|           | sendEmail()+printReport                    | descuentos, persistencia de órdenes, notificación por correo |
|           |                                            | y generación de reportes. Por lo tanto, existen motivos inde-|
|           |                                            | pendientes que podrían requerir modificaciones en la clase.  |
|-----------|--------------------------------------------|--------------------------------------------------------------|
| OCP       | applyDiscount (if/else sobre customerType) | El método evalúa los tipos de cliente mediante estructuras   | 
|           |                                            | condicionales `if (customerType.equals(...))`. Si se llega a |
|           |                                            | incorporar un nuevo tipo de descuento (Ej: "ESTUDIANTE"), es |
|           |                                            | necesario modificar el método existente y agregar nuevas con-|
|           |                                            | diciones, en lugar de extender su comportamiento mediante    |
|           |                                            | polimorfismo.                                                |
|-----------|--------------------------------------------|--------------------------------------------------------------|
| DIP       | Atributo "orders" y métodos saveOrder+     | El módulo de alto nivel OrderProcessor se acopla directamente|
|           |   sendEmail                                | a los detalles de bajo nivel e infraestructura al gestionar  |
|           |                                            | el almacenamiento en memoria mediante ArrayList y realizar la|
|           |                                            | notificación directamente mediante "System.out.println()", en|
|           |                                            | lugar de depender de abstracciones que permitan desacoplar   |
|           |                                            | estas responsabilidades de sus implementaciones concretas.   |
|-----------|--------------------------------------------|--------------------------------------------------------------|
|ISP        | Clase OrderProcessor                       | No se trata de una violación evidenciada en el código, pero  |
|           |                                            | al carecer de interfaces segregadas y monolíticas, cualquier |
|           |                                            | cliente que requiera únicamente el cálculo de montos o       |
|           |                                            | descuentos queda obligado a depender e interactuar con       |
|           |                                            | métodos irrelevantes para su propósito, como la persistencia |
|           |                                            | de la orden o la emisión de reportes.                        |
|-----------|--------------------------------------------|--------------------------------------------------------------|