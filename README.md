# sanchez-post-u2
Post-contenido — Exportación de reportes académicos con patrones creacionales justificados

## Decisiones de diseño

### Decisión 1 — Factory Method vs. Abstract Factory (Parte 1)

Patrón elegido: Abstract Factory

Justificación: el problema no crea un único producto que varía por formato, sino dos productos relacionados (ReportBody y ReportHeaderFooter) que deben pertenecer siempre al mismo formato para que el documento sea coherente. Al agregar un formato nuevo (CSV) haría falta una familia completa (cuerpo + encabezado/pie), no una sola clase. El riesgo real del problema es mezclar piezas de familias distintas (cuerpo Excel con encabezado PDF), que es exactamente lo que Abstract Factory previene al agrupar la creación de toda la familia en una sola fábrica. Factory Method se descartó porque resuelve bien la creación de una única jerarquía de producto, pero no garantiza consistencia entre dos productos relacionados.

### Decisión 2 — Mecanismo de extensibilidad de formatos (Parte 1)

Opción elegida: registro dinámico con Map<String, Supplier<ReportFormatFactory>>

Justificación: un switch o cadena de if/else sobre el string de formato obligaría a modificar ese método cada vez que se agregue un formato nuevo (como el CSV planeado), violando OCP. El registro central (ReportFactoryRegistry) resuelve esto exponiendo un método register() que permite agregar formatos nuevos sin tocar el código existente, y un método resolve() que lanza una excepción descriptiva si el formato no está registrado.