# Generador de Datos Sintéticos para Microsoft Fabric

Dos notebooks para Microsoft Fabric que trabajan en secuencia:

1. **`GeneradorDatosSinteticosPD.ipynb`** — Genera datasets sintéticos desde configuraciones YAML y los escribe como Parquet en el Lakehouse.
2. **`CargadorTablas.ipynb`** — Lee esos Parquets y los registra como Delta tables en el schema que configures.

---

## ¿Para quién es esto?

**Creadores de demos y contenido** — Si desarrollas reportes de Power BI o soluciones en Fabric y necesitas datos de ejemplo que se vean reales (nombres, fechas, montos, categorías con distribuciones realistas), este generador te permite crear un dataset completo en minutos sin depender de datos de producción.

**Profesionales que trabajan con datos sensibles** — Cuando el dataset real contiene información personal o confidencial, los datos sintéticos te permiten compartir, publicar o entregar tu solución sin exponer nada. Los datos generados son estadísticamente plausibles pero completamente ficticios.

**Testers y QA** — Simula volúmenes específicos (100 filas para desarrollo, 1 millón para pruebas de carga), casos especiales (nulos, valores extremos, distribuciones sesgadas) o escenarios de negocio concretos — todo desde un archivo de configuración, sin tocar el código.

---

> ⚠️ **Plataforma requerida**: Microsoft Fabric con un Lakehouse adjunto. No es compatible con Databricks ni Spark local en esta versión.

---

## Paso a paso

### 1. Descargar los notebooks

En esta página de GitHub, descarga los dos archivos `.ipynb`:

- Haz clic en `GeneradorDatosSinteticosPD.ipynb` → botón **Download raw file**
- Repite para `CargadorTablas.ipynb`

---

### 2. Importar en tu workspace de Fabric

1. Abre tu workspace en [Microsoft Fabric](https://app.fabric.microsoft.com)
2. Haz clic en **+ New item** → **Import notebook**
3. Selecciona los dos archivos `.ipynb` descargados
4. Asegúrate de que el notebook tenga un **Lakehouse adjunto** (panel izquierdo del notebook)

---

### 3. Instalar las dependencias

Abre `GeneradorDatosSinteticosPD.ipynb` y ejecuta la primera celda de instalación:

```python
%pip install faker faker-commerce pyyaml tqdm
```

> Reinicia el kernel después de la instalación si Fabric lo solicita.

---

### 4. Crear tu YAML de configuración

Parte de uno de los ejemplos en `Ejemplos/comunidad/` y adáptalo a tu caso.
La estructura mínima es:

```yaml
dataset:
  name: mi_tabla
  description: "Descripción del dataset"
  rows: 1000
  schema: MiSchema          # Schema de destino en el Lakehouse
  output: "Files/Data/MiCarpeta"
  columns:
    - name: ID
      type: int
      sequence: {start: 1, step: 1}
    - name: Nombre
      type: string
      faker: name
```

Consulta la sección **Generadores disponibles** para ver todas las opciones.

---

### 5. Validar la sintaxis del YAML

Antes de subir el archivo, pega su contenido en **[yamllint.com](https://www.yamllint.com/)** y verifica que no haya errores de indentación o formato. Los errores de YAML son silenciosos y difíciles de depurar en el notebook.

---

### 6. Subir el YAML al Lakehouse

1. En el panel del Lakehouse, navega a la carpeta donde quieres guardar tus configuraciones (ej: `Files/config/`)
2. Haz clic en **...** → **Upload files**
3. Selecciona tu archivo `.yml`

---

### 7. Configurar y ejecutar el generador

En `GeneradorDatosSinteticosPD.ipynb`, edita la **Celda 2**:

```python
FAKER_LOCALE      = "es_ES"              # Idioma: es_ES, en_US, fr_FR, etc.
RANDOM_SEED       = 42                   # None = aleatorio cada ejecución
BASE_OUTPUT_PATH  = "Files"

YAML_FILES = [
    "Files/config/mi_tabla.yml",         # Ruta a tu YAML en el Lakehouse
]
```

Ejecuta todas las celdas con **Run all**. Los Parquets se escriben en la carpeta `output` definida en tu YAML.

---

### 8. Cargar como Delta tables (opcional)

Si quieres que los datos queden disponibles como tablas en el Lakehouse, abre `CargadorTablas.ipynb` y edita la **Celda 1**:

```python
DESTINATION_SCHEMA = "dbo"              # Schema destino por defecto
                                        # El campo 'schema' del YAML lo sobreescribe por dataset
WRITE_MODE         = "overwrite"        # "overwrite" reemplaza | "append" agrega filas
YAML_FILES         = [
    "Files/config/mi_tabla.yml",        # Misma lista que en el generador
]
```

Ejecuta todas las celdas. Cada Parquet queda registrado como `{schema}.{nombre}` en el Lakehouse.

---

## Generadores disponibles

| Generador | Descripción | Ejemplo en YAML |
|-----------|-------------|-----------------|
| `sequence` | Valores secuenciales | `sequence: {start: 1, step: 1}` |
| `template` | Plantilla con secuencia formateada | `template: "SKU-{sequence:04d}"` |
| `faker` | Datos falsos realistas (nombres, emails, etc.) | `faker: name` / `faker: email` |
| `values` | Elige aleatoriamente de una lista fija | `values: ["A", "B", "C"]` |
| `range` | Valor en rango (int, decimal o date) | `range: [1, 100]` / `range: ["2020-01-01", "2024-12-31"]` |
| `weighted_values` | Lista con probabilidades por valor | `weighted_values: [{value: "X", weight: 0.7}, ...]` |
| `conditional` | Valor que depende de otra columna | `conditional: {based_on: Dept, mappings: {...}}` |
| `calculated` | Fórmula Python con columnas ya generadas | `calculated: {based_on: [A, B], formula: "A * B"}` |
| `fk` | Foreign Key desde un Parquet existente | `fk: {source: "Files/Data/stores.parquet", column: StoreKey}` |
| `nullable` | Porcentaje de NULLs en la columna | `nullable: 0.30` (30% NULL) |

**Tipos de columna soportados**: `int`, `string`, `decimal`, `bool`, `date`, `datetime`

---

## Ejemplos incluidos

Ubicados en `Ejemplos/comunidad/` — todos independientes entre sí, listos para usar:

| Archivo | Dominio | Filas | Generadores destacados |
|---------|---------|------:|------------------------|
| `employees.yml` | Recursos Humanos | 500 | sequence, faker, conditional, weighted_values, bool, nullable |
| `products_catalog.yml` | E-commerce | 300 | template, calculated (margen bruto) |
| `support_tickets.yml` | SaaS / Soporte | 1 000 | weighted_values, conditional, nullable (tickets abiertos) |
| `iot_readings.yml` | IoT / Industria | 10 000 | calculated (Heat Index), weighted_values (alertas) |

---

## Genera YAMLs con IA

Los ejemplos de la comunidad son un excelente punto de partida para pedirle a Claude Code (u otro LLM) que genere un YAML adaptado a tu industria o caso de uso.

**Cómo hacerlo:**

1. Descarga este `README.md` y uno o más YAMLs de `Ejemplos/comunidad/` como referencia
2. Abre Claude Code (o tu LLM preferido) y usa este prompt:

```
Tengo un generador de datos sintéticos que usa archivos YAML de configuración.
Aquí está la documentación completa del generador: [pega el contenido de README.md]
Aquí tienes un ejemplo de YAML: [pega el contenido de employees.yml o cualquier otro ejemplo]

Genera un YAML para [industria o dominio, ej: "una clínica médica" / "una cadena de restaurantes" / "una fintech de préstamos"].
El dataset debe representar [descripción breve, ej: "historial de consultas médicas con 2000 filas"].
Incluye columnas realistas para el dominio y distribuciones que tengan sentido de negocio (ej: más registros en categorías frecuentes, algunos nulos donde corresponda).
```

El LLM tendrá toda la referencia — generadores disponibles, sintaxis exacta y ejemplos reales — para producir un YAML listo para usar o muy cerca de eso.

---

## Adaptar los paths a tu Lakehouse

Los YAMLs usan paths relativos al root del Lakehouse (`Files/`). Ajusta `output` según tu estructura:

```yaml
dataset:
  output: "Files/Data/MiCarpeta"   # ← adaptar a tu carpeta
```

Para FK lookups, ajusta también `fk.source`:

```yaml
fk:
  source: "Files/Data/MiCarpeta/mi_tabla.parquet"  # ← adaptar
  column: MiKey
```

---

## Datasets históricos (SCD Type 2)

Para generar cambios históricos sobre un dataset existente, usa `source_parquet` en lugar de `rows`:

```yaml
dataset:
  name: product_changes
  source_parquet: "Files/Data/Products/products_base.parquet"
  change_generation:
    change_ratio: 0.15              # 15% de registros tendrán cambios
    num_changes_per_product: [1, 3] # Entre 1 y 3 cambios por registro
  columns:
    - name: ProductKey
      type: int
      from_source: "ProductKey"     # Copia del parquet fuente
    - name: ListPrice
      type: decimal
      from_source: "ListPrice"
      variation: [-0.20, 0.20]      # ±20% del valor original
```

---

## Validación avanzada de YAMLs

El archivo `Ejemplos/schema_dataset.yaml` define la estructura válida de un YAML de configuración. Para validar localmente con [yamale](https://github.com/23andMe/Yamale):

```bash
pip install yamale
yamale -s Ejemplos/schema_dataset.yaml Ejemplos/comunidad/employees.yml
```

---

## Licencia

MIT — ver [LICENSE](LICENSE)
