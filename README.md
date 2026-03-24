# CatalogoCloud XSD — Refactorización de CatalogoCloud_v2.xsd y CatalogoCloud_v2.xml

**Módulo:** LMSGI
**Autor:** Elena Sáez Lascurain  

---

## Misiones completadas

### Misión 1 — `attributeGroup`: La mochila de atributos
Se han extraído los atributos `id`, `rack` y `estado` del elemento `<servidor>` y se han agrupado en `<xs:attributeGroup name="AtributosServidor">` en la raíz del XSD. Desde `<servidor>` se referencia con `<xs:attributeGroup ref="AtributosServidor"/>`.

### Misión 2 — `group` + `sequence`: El paquete de hardware
Se ha creado `<xs:group name="ComponentesHardware">` con un `<xs:sequence>` que obliga a que los elementos aparezcan en este orden exacto: `cpu` → `ram` → `gpu` → `discos`. El contenido original de `<hardware>` se sustituye por `<xs:group ref="ComponentesHardware"/>`.

### Misión 3 — `minOccurs` / `maxOccurs`: Control de cantidades
- `<gpu>` tiene `minOccurs="0"` porque no todos los servidores disponen de tarjeta gráfica (ej. `srv-backup-01`).
- `<disco>` dentro de `<discos>` tiene `maxOccurs="8"` para limitar el número máximo de discos por servidor.

### Misión 4 — `choice`: Redes excluyentes
Se ha añadido el elemento `<red>` con `minOccurs="0"` dentro de `<software>`. Usa `<xs:choice>` para que el servidor tenga **o bien** `<ip_fija>` **o bien** `<dhcp>`, pero nunca los dos a la vez.

### Misión 5 — `all`: Auditoría flexible
Se ha añadido `<auditoria>` al final de cada `<servidor>` con `minOccurs="0"`. Usa `<xs:all>` para que `fecha_revision`, `tecnico` y `nota` puedan aparecer en cualquier orden, simulando el relleno rápido de un técnico.

### Misión 6 — `unique`: El guardián de la integridad
Se ha añadido `<xs:unique name="UnicoID">` sobre el elemento raíz `<catalogo_cloud>`. El selector apunta a `.//servidor` y el campo a `@id`, garantizando que ningún ID de servidor se repita en todo el documento.

---

## Preguntas de reflexión

### a. ¿Cuántas líneas de código he ahorrado al usar grupos?

Sin el `attributeGroup`, los tres atributos (`id`, `rack`, `estado`) habría que declararlos en cada punto donde se usen. En este esquema aparecen referenciados en dos lugares, lo que supone **6 líneas duplicadas evitadas**.

Sin el `group` de hardware, el bloque `cpu` + `ram` + `gpu` + `discos` con todos sus atributos habría que repetirlo completo cada vez que se quisiera reutilizar. El bloque ocupa aproximadamente **40 líneas**: con una sola reutilización adicional se ahorrarían esas 40 líneas; con dos reutilizaciones, 80.

En total, en este proyecto el ahorro estimado es de **~46 líneas**, y la ventaja real es que cualquier cambio en el hardware se hace en un único sitio.

### b. ¿Qué error me da VS Code si intento poner dos servidores con el mismo ID?

Al duplicar un `id` (por ejemplo, dos servidores con `id="srv-web-01"`), VS Code muestra el siguiente error de validación:
```
Duplicate unique value [srv-web-01] declared for identity constraint "UnicoID" of element "catalogo_cloud".
```

El error señala directamente la línea del segundo `<servidor>` con el ID repetido. Esto confirma que la restricción `<xs:unique>` funciona correctamente: el validador recorre todo el documento buscando el campo `@id` bajo el selector `.//servidor` y lanza el error en cuanto detecta un valor que ya había aparecido antes.
