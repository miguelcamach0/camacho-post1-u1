# Evidencia — Patrón Singleton en Spring Framework

## Identificación

- **Patrón:** Singleton
- **Categoría GoF:** Creacional
- **Clase:** `org.springframework.beans.factory.support.DefaultSingletonBeanRegistry`
- **Módulo:** `spring-beans`
- **Archivo fuente:** `spring-beans/src/main/java/org/springframework/beans/factory/support/DefaultSingletonBeanRegistry.java`
- **Líneas de código:** `159-173`
- **Método analizado:** `addSingleton(String beanName, Object singletonObject)`

## Fragmento de código fuente

```java
protected void addSingleton(String beanName, Object singletonObject) {
    Object oldObject = this.singletonObjects.putIfAbsent(beanName, singletonObject);
    if (oldObject != null) {
        throw new IllegalStateException("Could not register object [" + singletonObject +
                "] under bean name '" + beanName + "': there is already object [" + oldObject + "] bound");
    }
    this.singletonFactories.remove(beanName);
    this.earlySingletonObjects.remove(beanName);
    this.registeredSingletons.add(beanName);

    Consumer<Object> callback = this.singletonCallbacks.get(beanName);
		if (callback != null) {
			callback.accept(singletonObject);
		}
}