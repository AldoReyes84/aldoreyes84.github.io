---
layout: default
lang: es
permalink: /etl_ssis_integration_package/es/
title: Paquete ETL SSIS de Integración
---
# Canalización ETL: Integración de datos ficticios de Internet Sales

Este proyecto genera datos sintéticos diarios de Internet Sales usando Python y los carga en una tabla de SQL Server (FactInternetSales2) mediante un paquete SSIS. El objetivo es simular flujos de trabajo ETL realistas para pruebas, transformaciones y automatización.

## Crear una tabla copia para la tabla SQL Server AdventureWorksDW2022.FactInternetSales

<br>

<img width="634" height="91" alt="image" src="https://github.com/user-attachments/assets/31072e54-5a4a-47e7-a3ca-ff9cc68b1a77" />

<br>

## Crear un archivo de reporte diario ficticio de Internet Sales

### Preparar una revisión de los campos a crear

Utilicé el Data Model de Excel conectado a la tabla FactInternetSales de AdventureWorks para preparar los parámetros que usan Python Faker para crear datos aleatorios de InternetSales.

<br>

<img width="622" height="222" alt="image" src="https://github.com/user-attachments/assets/7e9e3e2c-a70b-4aaa-a17e-de7cf8a02555" />


<br>

No estamos intentando recrear una estructura exacta de la tabla SQL porque queremos hacer algunas transformaciones en el Data Flow de SSIS. Los últimos cinco campos se añadirán en el paso de transformación.


-----------------

## Python

El script en Python se conecta a la base de datos AdventureWorksDW2022, obtiene metadatos de FactInternetSales2 y DimProduct, y genera archivos CSV diarios con datos de ventas aleatorios usando pandas, random y Faker.


### Importar librerías

        import pandas as pd                                 # to create the Data Frame
        import random                                       # to create the Random Data
        from datetime import datetime, timedelta            # to manade Date Data
        import pyodbc                                       # to connect and query SQL Server DB Data
        from decimal import Decimal                         # to define decimal numbers for tax rate
        import os                                           # to export the final resulting files to folder


### Conectar a AdventureWorksDW2022

        conn = pyodbc.connect('DRIVER={SQL Server};SERVER="Server_Name";DATABASE=AdventureWorksDW2022;Trusted_Connection=yes;')
        cursor = conn.cursor()

### Obtener la última OrderDate de FactInternetSales2

        cursor.execute("SELECT MAX(OrderDate) FROM FactInternetSales2")
        last_order_date = cursor.fetchone()[0]
        start_date = last_order_date + timedelta(days=1)
        
### Obtener el número de orden más alto con prefijo 'SO'   
           
        cursor.execute("""
        SELECT MAX(CAST(SUBSTRING(SalesOrderNumber, 3, LEN(SalesOrderNumber)) AS INT))
        FROM FactInternetSales2
        WHERE SalesOrderNumber LIKE 'SO%'
        """)
        last_order_number = cursor.fetchone()[0]
        order_counter = (last_order_number or 0) + 1

### Obtener ListPrice y StandardCost desde DimProduct

        cursor.execute("""
        SELECT ProductKey, ListPrice, StandardCost
        FROM DimProduct
        WHERE ProductKey IS NOT NULL
        """)

### Almacenar precios en un diccionario para búsqueda rápida
        product_prices = {}
        for row in cursor.fetchall():
        product_key = row.ProductKey
        list_price = row.ListPrice if row.ListPrice is not None else 0
        standard_cost = row.StandardCost if row.StandardCost is not None else 0
        product_prices[product_key] = {
        'UnitPrice': list_price,
        'ProductStandardCost': standard_cost
        }

        conn.close()

# Definir rango de fechas para generación de órdenes

        current_date = start_date
        end_date = datetime(2014, 2, 3)

Este script leerá la última fecha de orden en FactInternetSales2 y la establecerá como start_date.

end_date se proporciona manualmente para determinar cuántos archivos diarios generará este script.

### Definir valores de lookup

        product_options = [214, 217, 222, 225, 228, 231, 234, 237, 310, 311, 312, 313, 314, 320, 321, 322, 323, 324, 325,             326, 327, 328, 329, 330, 331, 332, 333, 334, 335, 336, 337, 338, 339, 340, 341, 342, 343, 344, 345, 346, 347, 348,            349, 350, 351, 352, 353, 354, 355, 356, 357, 358, 359, 360, 361, 362, 363, 368, 369, 370, 371, 372, 373, 374, 375,            376, 377, 378, 379, 380, 381, 382, 383, 384, 385, 386, 387, 388, 389, 390, 463, 465, 467, 471, 472, 473, 474, 475,            476, 477, 478, 479, 480, 481, 482, 483, 484, 485, 486, 487, 488, 489, 490, 491, 528, 529, 530, 535, 536, 537, 538,            539, 540, 541, 560, 561, 562, 563, 564, 565, 566, 567, 568, 569, 570, 571, 572, 573, 574, 575, 576, 577, 578, 579,            580, 581, 582, 583, 584, 585, 586, 587, 588, 589, 590, 591, 592, 593, 594, 595, 596, 597, 598, 599, 600, 604, 605,            606]
        promotion_options = [1, 2, 13, 14]
        currency_options = [6, 19, 29, 39, 98, 100]

### Inicializar lista para almacenar órdenes

        orders = []

### Bucle por cada día en el rango de fechas

        while current_date <= end_date:

### Generar un número aleatorio de órdenes para el día actual

        num_orders = random.randint(1, 8)

        for _ in range(num_orders):
        product_key = random.choice(product_options)
        price_info = product_prices.get(product_key, {'UnitPrice': 0, 'ProductStandardCost': 0})

        order_id = f"SO{order_counter}"

        tax_rate = Decimal('0.08')

        order_date = current_date.date()
        order_date_key = int(current_date.strftime('%Y%m%d'))  # Formato YYYYMMDD como entero

### Generar DueDate: OrderDate + 4 a 30 días
     
        due_date = order_date + timedelta(days=random.randint(4, 30))
        due_date_key = int(due_date.strftime('%Y%m%d'))

### Generar ShipDate: DueDate + 2 a 15 días
  
        ship_date = due_date + timedelta(days=random.randint(2, 15))
        ship_date_key = int(ship_date.strftime('%Y%m%d'))

### Crear diccionario de campos

        orders.append({
            'ProductKey': product_key,                                # Product identifier
            'OrderDateKey': order_date_key,                           # Order date as integer YYYYMMDD
            'DueDateKey': due_date_key,                               # Due date as integer YYYYMMDD
            'ShipDateKey': ship_date_key,                             # Ship date as integer YYYYMMDD
            'CustomerKey': random.randint(11000, 29483),              # Customer identifier
            'PromotionKey': random.choice(promotion_options),         # Promotion applied
            'CurrencyKey': random.choice(currency_options),           # Currency used
            'SalesTerritory': random.randint(1, 10),                  # Sales territory ID
            'SalesOrderNumber': order_id,                             # Unique order ID
            'SalesOrderLine': random.randint(1, 8),                   # Line number within the order
            'RevisoryNumber': None,                                   # Revisory number null
            'SalesOrderQuantity': 1,                                  # Fixed quantity per order
            'UnitPrice': price_info['UnitPrice'],                     # ListPrice from DimProduct
            'ExtendedAmount': price_info['UnitPrice'],                # Extended amount = UnitPrice
            'UnitPriceDiscount': 0,                                   # Unit price discount = 0
            'ProductStandardCost': price_info['ProductStandardCost'], # StandardCost from DimProduct
            'TotalProductCost': price_info['ProductStandardCost'],    # Total product cost = ProductStandardCost
            'SalesAmount': price_info['UnitPrice'],                   # Sales amount = UnitPrice
            'TaxAmount': price_info['UnitPrice'] * Decimal('0.08'),   # Tax amount = 8% of SalesAmount
            'Freight': None,                                          # Freight requires another DB for practical purposes won't implement it here
            'OrderDate': current_date.date(),                         # Date of the order
            'DueDate': due_date,                                     # Due date
            'ShipDate': ship_date,                                   # Ship date
        })
            
        order_counter += 1  # Increment global order ID

        current_date += timedelta(days=1)  # Move to the next day

### Convertir la lista de órdenes a DataFrame

        df = pd.DataFrame(orders)

### Agrupar por OrderDate

        grouped = df.groupby('OrderDate')

### Mostrar el DataFrame resultante
        
        print(df)

### Definir carpeta de salida y asegurar que exista
        
        output_folder = r'C:\Users\aldoa\Documents\Proyectos\Python Faker\FakeInternetSalesOrders\\'
        os.makedirs(output_folder, exist_ok=True)

### Iterar por cada grupo (día)

        for order_date, group in grouped:
        # Convert the date to YYYYMMDD format
        date_str = order_date.strftime('%Y%m%d')

        # Define the full path for the file
        filename = f"{output_folder}FakeOrders_{date_str}.csv"

### Guardar el grupo como CSV en la carpeta correspondiente
    
        group.to_csv(filename, index=False, encoding='utf-8')

### Resultados

<img width="1432" height="368" alt="image" src="https://github.com/user-attachments/assets/d3ba20e2-567d-484d-9849-4d941a59535b" />

<br>
<br>

<img width="665" height="279" alt="image" src="https://github.com/user-attachments/assets/7f7ce703-800b-47a1-96fc-e319a4e4f373" />

<br>
<br>

-------
## SQL Server Integration Services (SSIS)

Construido en Visual Studio 2022, este paquete SSIS ingiere los archivos CSV generados, transforma los datos y los carga en FactInternetSales2.


### Extract

#### 1. Foreach Loop Container
- Itera a través de todos los archivos en la SourceFolder designada.
- Asigna dinámicamente cada ruta de archivo a la variable FileName.
- Construye rutas completas para source y backup usando FullSourcePath y FullBackupPath.
 

<img width="1159" height="718" alt="image" src="https://github.com/user-attachments/assets/2270bb45-8a73-4cd2-b01f-92337eae2b79" />


<br>
#### 2. Data Flow Task
Procesa cada archivo a través de los siguientes componentes

 Add a Flat File, select one of the FakeOrder

<img width="886" height="746" alt="image" src="https://github.com/user-attachments/assets/6b42d48b-119c-4522-a0b2-c79fc1a061af" />

### Transform

Compare the FactInternetSales2 table and the FakeOrder files headers 

The first fields match ok 
<img width="1296" height="171" alt="image" src="https://github.com/user-attachments/assets/def60e63-f00c-44b7-9fe5-3086c9256f18" />

#### 3. Derived Column

Create the missing fields and set the RevisoryNumber as RevisionNumber and value set as 1 

<img width="1038" height="673" alt="image" src="https://github.com/user-attachments/assets/98bbd715-1f8f-4f37-ab10-f9c6b0eb1cd8" />

Review the source data types 

<img width="338" height="514" alt="image" src="https://github.com/user-attachments/assets/ed9f1535-5621-4be8-94ad-b9c496ed461e" />

#### 4. Data Convertion

Converts data types to match SQL Server schema

<img width="979" height="805" alt="image" src="https://github.com/user-attachments/assets/19e1babd-0080-47f8-b4eb-c2abbb82c8a4" />

#### 5. Conditional Split

Filters out rows with NULLs (precautionary, due to NOT NULL constraints)

<img width="1120" height="392" alt="image" src="https://github.com/user-attachments/assets/c12d1cf9-4a9e-478f-a569-3a5e3045fbb2" />

### Load

#### 6. OLE DB Destination

Inserta las filas válidas en FactInternetSales2
 
<img width="1064" height="554" alt="image" src="https://github.com/user-attachments/assets/e29eae4c-9f83-4bca-b7e6-254dffd7c71f" />

El Mapping detecta todos los campos por defecto pero los campos correctos para ingerir son las copias de Data Convertion

<img width="754" height="636" alt="image" src="https://github.com/user-attachments/assets/20b9e652-77cb-4732-9a45-518c681d29ee" />

#### 7. Flat File Destination 

Captura las filas que fallan durante la inserción (por ejemplo, debido a truncamiento o violaciones de null).

<img width="1018" height="612" alt="image" src="https://github.com/user-attachments/assets/ebe26720-55ce-4ce8-abcc-fc2b57d85393" />

#### 8. File System Task

Mueve los archivos procesados con éxito desde la carpeta source a la carpeta destino.

Crear variables necesarias

<img width="715" height="216" alt="image" src="https://github.com/user-attachments/assets/bc7dd020-98e6-43e9-a95d-a35a28400385" />

Configurar File System Task con las rutas Full Paths que establecimos como variables y la operación para mover el archivo

<img width="1008" height="422" alt="image" src="https://github.com/user-attachments/assets/5de79a67-45de-45fc-b2e3-f1a476e057e2" />



