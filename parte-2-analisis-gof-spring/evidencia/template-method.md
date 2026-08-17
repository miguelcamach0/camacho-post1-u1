# Evidencia — Patrón Template Method en Spring Framework

## Identificación

- **Patrón:** Template Method
- **Categoría GoF:** Comportamiento
- **Clase:** `org.springframework.jdbc.core.JdbcTemplate`
- **Módulo:** `spring-jdbc`
- **Archivo fuente:** `spring-jdbc/src/main/java/org/springframework/jdbc/core/JdbcTemplate.java`
- **Líneas de código:** `404-415` `429-434` `565-578`
- **Métodos analizados:** `execute(StatementCallback<T> action, boolean closeResources)` y `update(String sql)`

## Evidencia 1 — Algoritmo general

El método execute(...)" establece el flujo común de una operación JDBC: obtiene la conexión, crea y configura el "Statement", delega la operación específica mediante el callback, maneja las advertencias y devuelve el resultado.

```java
private <T extends @Nullable Object> T execute(StatementCallback<T> action, boolean closeResources) throws DataAccessException {
		Assert.notNull(action, "Callback object must not be null");

		Connection con = DataSourceUtils.getConnection(obtainDataSource());
		Statement stmt = null;
		try {
			stmt = con.createStatement();
			applyStatementSettings(stmt);
			T result = action.doInStatement(stmt);
			handleWarnings(stmt);
			return result;
		}
```

## Evidencia 2 - Liberación de recursos

El mismo método mantiene centralizada la liberación de los recursos JDBC:

```java
finally {
    if (closeResources) {
        JdbcUtils.closeStatement(stmt);
        DataSourceUtils.releaseConnection(con, getDataSource());
    }
}
```
## Evidencia 3 — Implementación del paso específico

El método update(String sql) proporciona una implementación concreta
del comportamiento delegado mediante UpdateStatementCallback.

```java
class UpdateStatementCallback implements StatementCallback<Integer>, SqlProvider {
    @Override
    public Integer doInStatement(Statement stmt) throws SQLException {
        int rows = stmt.executeUpdate(sql);
        if (logger.isTraceEnabled()) {
            logger.trace("SQL update affected " + rows + " rows");
        }
        return rows;
    }


    @Override
    public String getSql() {
        return sql;
    }
}
return updateCount(execute(new UpdateStatementCallback(), true));
```