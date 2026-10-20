# Ecommerce Olist — ventas, logística y clientes

Análisis de **99.441 pedidos reales** de un marketplace brasileño (2016–2018) para responder seis preguntas de negocio sobre ingresos, productos, entregas, satisfacción, recompra y medios de pago.

**Stack:** PostgreSQL 18 · Power BI Desktop

> **Estado:** en desarrollo. Las seis preguntas están definidas; los hallazgos se incorporan a medida que cada una se responde.

---

## Fuente y créditos

| Campo | Detalle |
|---|---|
| Dataset | Brazilian E-Commerce Public Dataset by Olist |
| Enlace | https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce |
| Autor | Francisco Magioli |
| Titular | Olist |
| Licencia | CC BY-NC-SA 4.0 |
| Periodo | septiembre 2016 – octubre 2018 |
| Versión | 2 |

Datos comerciales reales anonimizados por el publicador. Los textos de las reseñas fueron sustituidos por nombres de casas de *Juego de Tronos*. Uso educativo y no comercial; el dataset conserva su licencia original.

---

## Contexto de negocio

Olist conecta pequeños comercios de todo Brasil con canales de venta y se encarga de la logística. El análisis se sostiene en tres preguntas:

1. **¿Dónde está el ingreso?** Categorías, ticket promedio y estacionalidad.
2. **¿Por qué se quejan los clientes?** Tiempo real de entrega, retrasos y su efecto en la calificación.
3. **¿Cuánto vale el cliente que vuelve?** Recompra y segmentación por recencia, frecuencia y valor.

## Preguntas de negocio

| # | Pregunta | Estado |
|---|---|---|
| 01 | ¿Cómo evolucionan los ingresos y el ticket promedio mes a mes, y hay estacionalidad? | Pendiente |
| 02 | ¿Qué categorías concentran el ingreso y cuáles solo volumen? | Pendiente |
| 03 | ¿Cuánto tarda realmente una entrega y qué porcentaje llega después de la fecha estimada? | Pendiente |
| 04 | ¿Cómo cambia la calificación del cliente cuando el pedido llega tarde? | Pendiente |
| 05 | ¿Qué peso tiene la recompra y cómo se segmentan los clientes por recencia, frecuencia y valor? | Pendiente |
| 06 | ¿Qué métodos de pago y esquemas de cuotas concentran el ingreso? | Pendiente |

Cada pregunta se resuelve con un script propio en `sql/` y una visualización en el tablero.

---

## Estructura

```
.
├── data/
│   ├── raw/                CSVs originales descargados de Kaggle
│   └── processed/          Tablas de salida que alimentan el tablero
├── docs/
│   └── data_dictionary.md  Diccionario de tablas, claves y trampas
├── powerbi/                Modelo, reporte e imágenes del tablero
├── sql/                    Un script por pregunta
└── README.md
```

## Reproducción

La descarga requiere cuenta de Kaggle, así que el paso es manual:

1. Abrir https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce e iniciar sesión.
2. Botón **Download** (126 MB, archivo `.zip`).
3. Descomprimir los nueve CSV dentro de `data/raw/`.
4. Crear la base y cargar:

```bash
createdb olist
psql -d olist -f sql/00_setup.sql
```

5. Ejecutar los scripts de `sql/` en orden. Cada uno responde una pregunta y deja su tabla de salida lista para el tablero.

---

## Hallazgos

Se documentan a medida que cada pregunta se responde, con la imagen del tablero y su interpretación de negocio.
