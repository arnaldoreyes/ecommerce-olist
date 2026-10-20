# Diccionario de datos — Olist

Nueve archivos CSV. La granularidad es lo que impide inflar los totales: `orders` es un registro por pedido, `order_items` un registro por ítem y `order_payments` un registro por pago.

> Las columnas se validan contra los CSV al ejecutar `sql/00_setup.sql`.

## Tablas

### `olist_orders_dataset.csv` — 99.441 filas

Un registro por pedido. Es la tabla central del modelo.

| Columna | Tipo | Descripción |
|---|---|---|
| `order_id` | texto | Identificador del pedido |
| `customer_id` | texto | Clave del cliente dentro de `customers` |
| `order_status` | texto | delivered, shipped, canceled, invoiced, processing, unavailable, approved, created |
| `order_purchase_timestamp` | timestamp | Fecha de compra |
| `order_approved_at` | timestamp | Aprobación del pago |
| `order_delivered_carrier_date` | timestamp | Entrega del pedido al transportista |
| `order_delivered_customer_date` | timestamp | Entrega al cliente |
| `order_estimated_delivery_date` | timestamp | Fecha estimada prometida al cliente |

### `olist_order_items_dataset.csv` — 112.650 filas

Un registro por ítem de cada pedido. Aquí vive el precio de venta y el flete.

| Columna | Tipo | Descripción |
|---|---|---|
| `order_id` | texto | Pedido |
| `order_item_id` | entero | Número de secuencia del ítem dentro del pedido |
| `product_id` | texto | Producto |
| `seller_id` | texto | Vendedor que despacha |
| `shipping_limit_date` | timestamp | Fecha límite de despacho del vendedor |
| `price` | numérico | Precio unitario del ítem (BRL) |
| `freight_value` | numérico | Flete del ítem (BRL) |

### `olist_order_payments_dataset.csv` — 103.886 filas

Un registro por pago. Un pedido puede tener varios.

| Columna | Tipo | Descripción |
|---|---|---|
| `order_id` | texto | Pedido |
| `payment_sequential` | entero | Orden del pago dentro del pedido |
| `payment_type` | texto | credit_card, boleto, voucher, debit_card, not_defined |
| `payment_installments` | entero | Número de cuotas |
| `payment_value` | numérico | Valor pagado (BRL) |

### `olist_order_reviews_dataset.csv` — 99.224 filas

Evaluación del cliente. Puede haber más de una por pedido.

| Columna | Tipo | Descripción |
|---|---|---|
| `review_id` | texto | Identificador de la reseña |
| `order_id` | texto | Pedido evaluado |
| `review_score` | entero | 1 a 5 |
| `review_comment_title` | texto | Título del comentario |
| `review_comment_message` | texto | Comentario |
| `review_creation_date` | timestamp | Fecha de la encuesta |
| `review_answer_timestamp` | timestamp | Fecha de respuesta |

### `olist_customers_dataset.csv` — 99.441 filas

Un registro por pedido, no por persona. La persona es `customer_unique_id`.

| Columna | Tipo | Descripción |
|---|---|---|
| `customer_id` | texto | Clave del cliente en el pedido |
| `customer_unique_id` | texto | Identificador real de la persona |
| `customer_zip_code_prefix` | texto | Prefijo postal (5 dígitos) |
| `customer_city` | texto | Ciudad |
| `customer_state` | texto | Estado (UF) |

### `olist_products_dataset.csv` — 32.951 filas

| Columna | Tipo | Descripción |
|---|---|---|
| `product_id` | texto | Producto |
| `product_category_name` | texto | Categoría en portugués |
| `product_name_lenght` | entero | Largo del nombre (typo en el original) |
| `product_description_lenght` | entero | Largo de la descripción (typo en el original) |
| `product_photos_qty` | entero | Número de fotos |
| `product_weight_g` | entero | Peso en gramos |
| `product_length_cm` | entero | Largo en cm |
| `product_height_cm` | entero | Alto en cm |
| `product_width_cm` | entero | Ancho en cm |

### `olist_sellers_dataset.csv` — 3.095 filas

| Columna | Tipo | Descripción |
|---|---|---|
| `seller_id` | texto | Vendedor |
| `seller_zip_code_prefix` | texto | Prefijo postal |
| `seller_city` | texto | Ciudad |
| `seller_state` | texto | Estado (UF) |

### `olist_geolocation_dataset.csv` — 1.000.163 filas

Coordenadas por prefijo postal. Muchas filas por código: hay que deduplicar antes de unir.

| Columna | Tipo | Descripción |
|---|---|---|
| `geolocation_zip_code_prefix` | texto | Prefijo postal |
| `geolocation_lat` | numérico | Latitud |
| `geolocation_lng` | numérico | Longitud |
| `geolocation_city` | texto | Ciudad |
| `geolocation_state` | texto | Estado (UF) |

### `product_category_name_translation.csv` — 71 filas

Traducción de categorías de portugués a inglés.

| Columna | Tipo | Descripción |
|---|---|---|
| `product_category_name` | texto | Categoría en portugués |
| `product_category_name_english` | texto | Categoría en inglés |

---

## Relaciones

```
customers ──1:N──> orders ──1:N──> order_items ──N:1──> products
                      │                    └──────N:1──> sellers
                      ├──1:N──> order_payments
                      └──1:N──> order_reviews

products.product_category_name ──1:1──> product_category_name_translation
customers.customer_zip_code_prefix ──N:1──> geolocation.geolocation_zip_code_prefix
```

| Desde | Hacia | Clave |
|---|---|---|
| `orders` | `customers` | `customer_id` |
| `order_items` | `orders` | `order_id` |
| `order_payments` | `orders` | `order_id` |
| `order_reviews` | `orders` | `order_id` |
| `order_items` | `products` | `product_id` |
| `order_items` | `sellers` | `seller_id` |
| `products` | `product_category_name_translation` | `product_category_name` |

---

## Trampas conocidas

- **`customer_id` no es la persona.** Identifica al cliente dentro de un pedido. La persona es `customer_unique_id`; si se usa `customer_id` para medir recompra, el resultado da 100% de compras únicas.
- **Fan-out de `order_items`.** Unir pedidos con ítems multiplica filas. Sumar `price` después de unir con pagos infla el ingreso.
- **Fan-out de `order_payments`.** Un pedido puede tener varios pagos; sumar `payment_value` sin agrupar por `order_id` duplica el ingreso.
- **`order_reviews` con más de una reseña por pedido.** Agregar antes de unir, o el pedido aparece repetido.
- **`geolocation` con ~1 millón de filas para ~19 mil códigos postales.** Deduplicar por prefijo antes de cualquier join.
- **`order_status` incluye pedidos cancelados y no entregados.** Las métricas de entrega se calculan sobre pedidos con `order_delivered_customer_date` no nulo.
- **2016 no es comparable.** El volumen de 2016 es mínimo frente a 2017-2018; los promedios mensuales de ese año se leen con cuidado.
- **Fechas en texto ISO.** Se castean a `timestamp` al cargar; no se asume que ya vienen como fecha.
