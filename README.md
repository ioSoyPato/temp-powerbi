# AHHHHHH 

### Importar todas las tablas
1. Ve a **Inicio > Obtener datos > Texto/CSV**.
2. Importa los archivos:
   - `DimCity.csv`
   - `DimCustomer.csv`
   - `DimStockItem.csv`
   - `DimDate.csv`
   - `JuneFactSale.csv`

### Transformar los datos en Power Query
1. Limpia duplicados:
   - Selecciona columnas clave y ve a **Inicio > Quitar duplicados**.
2. Elimina valores nulos:
   - Filtra columnas para eliminar filas en blanco.
3. Estandariza formatos:
   - Configura tipos de datos:
     - Fechas: **Fecha**
     - Números: **Decimal Number** o **Entero**
     - Texto: **Texto**

### Combinar datos
1. Combina `JuneFactSale` con los datos históricos:
   - Selecciona la tabla histórica (`FactSales`).
   - Ve a **Inicio > Combinar consultas > Combinar consultas como nuevas**.
   - Selecciona `JuneFactSale` como tabla secundaria y conecta las columnas clave (`Date`, `StockItemKey`).
   - Elimina duplicados en la tabla combinada.

---

## 2. Crear visualizaciones requeridas

### a) Visualizaciones diversas
1. **Gráfico de barras**:
   - Muestra ventas por mes:
     - Eje X: `Date` agrupado por mes.
     - Eje Y: Suma de `SalesAmount`.
2. **Gráfico de líneas**:
   - Tendencia de ventas totales:
     - Eje X: `Date`.
     - Eje Y: Suma de `SalesAmount`.
3. **Mapa**:
   - Visualiza ventas por ciudad:
     - Ubicación: `City` (de `DimCity`).
     - Tamaño/Color: `SalesAmount`.
4. **Tabla resumen**:
   - Incluye:
     - Ventas totales (`SalesAmount`).
     - Cantidad vendida (`CantidadVendida`).
     - Profit (`Profit`) por estado/provincia.
5. **Gráfico circular**:
   - Distribución de ventas por categoría de producto:
     - Leyenda: `Category` (de `DimStockItem`).
     - Valores: Suma de `SalesAmount`.
6. **Gráfico de dispersión**:
   - Relaciona `Profit` con `CantidadVendida`.

### b) Tarjetas de datos (Data Cards)
1. Indicadores clave:
   - Ventas totales: `SUM('FactSales'[SalesAmount])`.
   - Cantidad vendida total: `SUM('FactSales'[CantidadVendida])`.
   - Profit total: `SUM('FactSales'[Profit])`.
   - Ciudad con más ventas:
     ```DAX
     CiudadMayorVentas = 
     MAXX(
         SUMMARIZE(
             'FactSales',
             'DimCity'[City],
             "Ventas", SUM('FactSales'[SalesAmount])
         ),
         [Ventas]
     )
     ```

### c) Visuales de filtro
1. Agrega segmentaciones para:
   - Fecha (`Date`).
   - Ciudad (`City`).
   - Categoría de producto (`Category`).
   - Estado/provincia (`State Province`).

---

## 3. Responder las preguntas de la práctica

### Pregunta 1: Mes, Año y Profit para Nevada
1. Crea una tabla/matriz:
   - Columnas: `Date` (formato año-mes) y `Profit`.
   - Filtro: `State Province = Nevada`.

### Pregunta 2: Cambio mes a mes en cantidad vendida
1. Medida DAX para cambio absoluto:
   ```DAX
   CambioAbsoluto = 
   SUM('FactSales'[CantidadVendida]) - 
   CALCULATE(SUM('FactSales'[CantidadVendida]), PREVIOUSMONTH('FactSales'[Date]))

2. Medida DAX para cambio relativo:
``` DAX
CambioRelativo = 
DIVIDE(
    [CambioAbsoluto], 
    CALCULATE(SUM('FactSales'[CantidadVendida]), PREVIOUSMONTH('FactSales'[Date]))
)
```

3. Muestra mayo y junio de 2016 en una tabla con estas medidas.

### Pregunta 3: Ventas totales de productos de refrigeración
1. Filtra DimStockItem para Category = "Chiller Stock".
2. Medida DAX para ventas totales:
```
VentasChiller = 
CALCULATE(
    SUM('FactSales'[SalesAmount]),
    'DimStockItem'[Category] = "Chiller Stock"
)
```

### Preguntas adicionales
Pregunta 4: Estado con mayor profit:
- Gráfico de barras:
    - Eje X: State Province.
    - Eje Y: Suma de Profit.


Pregunta 5: Top 5 productos más vendidos:
- Gráfico de barras:
    - Eje X: ProductName (de DimStockItem).
    - Eje Y: Suma de CantidadVendida.
    - Filtra los 5 más altos.
