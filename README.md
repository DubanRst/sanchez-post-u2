# sanchez-post-u2
# Post-contenido — Unidad 2: Patrones Creacionales

## Descripción

Repositorio del post-contenido de la Unidad 2 de Patrones de Diseño de Software. Un único proyecto Maven (`exportador-reportes/`) que resuelve la exportación de reportes académicos en múltiples formatos (Parte 1) y se extiende con configuración compleja y evaluación de Singleton (Parte 2).

## Cómo ejecutar

```powershell
cd exportador-reportes
mvn compile
mvn exec:java
```

## Decisiones de diseño

### Decisión 1 — Factory Method vs. Abstract Factory (Parte 1)

Patrón elegido: Abstract Factory

Justificación: el problema no crea un único producto que varía por formato, sino dos productos relacionados (ReportBody y ReportHeaderFooter) que deben pertenecer siempre al mismo formato para que el documento sea coherente. Al agregar un formato nuevo (CSV) haría falta una familia completa (cuerpo + encabezado/pie), no una sola clase. El riesgo real del problema es mezclar piezas de familias distintas (cuerpo Excel con encabezado PDF), que es exactamente lo que Abstract Factory previene al agrupar la creación de toda la familia en una sola fábrica. Factory Method se descartó porque resuelve bien la creación de una única jerarquía de producto, pero no garantiza consistencia entre dos productos relacionados.

### Decisión 2 — Mecanismo de extensibilidad de formatos (Parte 1)

Opción elegida: registro dinámico con Map<String, Supplier<ReportFormatFactory>>

Justificación: un switch o cadena de if/else sobre el string de formato obligaría a modificar ese método cada vez que se agregue un formato nuevo (como el CSV planeado), violando OCP. El registro central (ReportFactoryRegistry) resuelve esto exponiendo un método register() que permite agregar formatos nuevos sin tocar el código existente, y un método resolve() que lanza una excepción descriptiva si el formato no está registrado.

### Decisión 3 — Builder vs. constructor telescópico vs. setters (Parte 2)

Patrón elegido: Builder

Justificación: ExportConfig tiene 1 parámetro obligatorio y 8 opcionales. Un constructor con los 9 parámetros obligaría al cliente a recordar el orden exacto, con riesgo de invertir parámetros del mismo tipo (String, boolean) sin que el compilador lo detecte. Constructores sobrecargados crecerían demasiado con 8 parámetros opcionales. Una clase mutable con setters sueltos permitiría dejar el objeto a medio configurar, sin un punto único donde validar la consistencia de la combinación de valores. Builder resuelve las tres limitaciones: expone métodos encadenables solo para lo opcional, valores por defecto razonables, y centraliza la validación de estados inconsistentes en build() (por ejemplo, rechazar compress=true sin outputPath).

### Decisión 4 — ¿ReportFactoryRegistry necesita ser Singleton? (Parte 2)

Conclusión: NO conviene Singleton

Justificación: ReportFactoryRegistry no necesita identidad de objeto, ya que nada lo sustituye por polimorfismo ni lo inyecta por constructor en este proyecto. Su inicialización tampoco es costosa: es un Map con tres entradas armado en un bloque static, sin trabajo relevante que justifique una inicialización perezosa. El propio campo static ya garantiza una única fuente de verdad para toda la JVM sin necesidad de la maquinaria de Singleton (constructor privado con guardas, getInstance(), sincronización). Convertirlo en Singleton clásico agregaría ceremonia sin resolver un problema real. Esta conclusión cambiaría solo si el proyecto evolucionara a una plataforma multi-institución donde cada institución necesitara su propio registro independiente — en ese escenario Singleton dejaría de ser apropiado por la razón contraria: haría falta más de una instancia.

## Herramientas utilizadas

- Java 21, Apache Maven, VS Code, Git, GitHub

## Conclusiones

Este post-contenido mostró que elegir un patrón creacional no es cuestión de intuición o costumbre, sino de analizar la estructura real del problema: la necesidad de mantener coherentes dos productos relacionados llevó a Abstract Factory sobre Factory Method, y el riesgo de un objeto con múltiples parámetros opcionales llevó a Builder sobre alternativas más simples pero limitadas. La evaluación de Singleton para ReportFactoryRegistry dejó como aprendizaje que un patrón no debe aplicarse por costumbre ("los registros centrales siempre son Singleton"), sino solo cuando resuelve un problema real de identidad de objeto, inicialización costosa o necesidad de una única instancia verificable.
