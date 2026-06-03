# EDA - Análisis Exploratorio del Dataset Titanic con Pandas

##  Descripción del Proyecto
[cite_start]Este proyecto implementa el flujo completo de análisis de datos (cargar -> explorar -> limpiar -> transformar -> analizar) basado en el Capítulo 2 de Géron [cite: 11] [cite_start]utilizando exclusivamente la librería `pandas`. [cite_start]El objetivo fundamental es determinar los factores determinantes asociados con la probabilidad de supervivencia durante el trágico hundimiento del RMS Titanic el 15 de abril de 1912[cite: 9, 13].

---

##  Estructura del Repositorio

```text
titanic-eda/
├── data/
│   └── titanic_clean.csv      # Dataset sanitizado y enriquecido (Paso 7)
├── notebook/
│   └── Desarrollo.ipynb       # Notebook Jupyter principal con el EDA completo
├── .gitignore                 # Exclusión de entornos virtuales y temporales
├── README.md                  # Documentación técnica (este archivo)
└── requirements.txt           # Dependencias del proyecto (Pandas, Jupyter)

```

##  🛠️ Configuración del Entorno y Herramientas

```text

Para garantizar la reproducibilidad del análisis sin interferir con las librerías globales del sistema, se utilizó un entorno virtual de Python (venv) y se gestionaron las dependencias mediante pip.

PowerShell
# 1. Activación del entorno (PowerShell)
.\venv\Scripts\Activate.ps1

# 2. Instalación de dependencias del archivo requirements.txt
pip install -r requirements.txt

```

**Nota técnica:** Debido al uso de Python 3.13, se configuraron restricciones de versiones superiores para pandas y numpy para asegurar compatibilidad nativa con wheels precompilados en Windows, evitando la necesidad de instalar herramientas de compilación pesadas de C++ (Visual Studio).

# 📊 Metodología Aplicada (Flujo de Datos)

## 1. Exploración Inicial y Tratamiento de Nulos

Al auditar los **891 registros originales**, se identificaron valores faltantes (*NaN*) y se aplicaron criterios heurísticos de ingeniería de datos según los umbrales de pérdida:

### Embarked (0.22% de nulos)
Al ser una variable categórica nominal (texto sin orden jerárquico), se imputó utilizando la **Moda estadística** (`'S'`). No es posible promediar texto; se utiliza el valor más frecuente.

### Age (19.87% de nulos)
Al ser una variable numérica con distribución sesgada, se imputó utilizando la **Mediana** (**28 años**). La mediana es robusta frente a valores atípicos (*outliers*), a diferencia de la media aritmética, que se ve afectada por los extremos.

### Cabin (77.10% de nulos)
Superó el umbral de tolerancia del **50%**. Imputar más de la mitad de una columna introduciría patrones artificiales dañinos, por lo que se procedió a su eliminación estructural mediante `.drop()`.

---

## 2. Ingeniería de Características (*Feature Engineering*)

Se crearon nuevas variables de negocio para extraer patrones ocultos en el dataset:

### FamilySize
Variable cuantitativa discreta:

\[
FamilySize = SibSp + Parch + 1
\]

Permite consolidar el tamaño del núcleo familiar.

### IsAlone
Variable bandera (*flag*) binaria (`1` o `0`) para identificar pasajeros sin responsabilidades familiares inmediatas.

### AgeGroup
Discretización (*binning*) de la edad continua en rangos ordinales específicos:

- Niño
- Adolescente
- Adulto
- Mayor

### Title
Extracción del título social contenido en el nombre mediante expresiones regulares:

```python
r' ([A-Za-z]+)\.'
```

Permite aislar el estatus socioeconómico de la época (*Mr., Mrs., Miss., Master.*, etc.).

---

# 📈 Hallazgos y Análisis Multivariado

El análisis basado en agrupaciones complejas (`.groupby()`) arrojó las siguientes tasas de supervivencia:

## Análisis Univariado / Bilineal

### Impacto de Género

| Género | Tasa de Supervivencia |
|---------|----------------------:|
| Mujeres | 74.20% |
| Hombres | 18.89% |

Las mujeres obtuvieron una tasa de supervivencia significativamente superior a la de los hombres.

### Impacto de Clase Social

| Clase | Tasa de Supervivencia |
|--------|----------------------:|
| Primera Clase | 62.96% |
| Segunda Clase | 47.28% |
| Tercera Clase | 24.24% |

Se observa una fuerte correlación positiva entre el nivel socioeconómico y la probabilidad de supervivencia.

### Impacto Etario

| Grupo de Edad | Tasa de Supervivencia |
|--------------|----------------------:|
| Niño | 57.97% |
| Mayor | 22.73% |

Los niños lideraron la tasa de supervivencia, mientras que los pasajeros de mayor edad presentaron la menor probabilidad de sobrevivir.

---

# 🔍 Análisis Multivariado (Jerarquía de Rescate)

Al cruzar simultáneamente las variables **Sex**, **Pclass** y **AgeGroup** mediante un índice jerárquico multinivel (*MultiIndex*), se determinó que la premisa histórica de **"mujeres y niños primero"** estuvo fuertemente condicionada por el nivel socioeconómico.

## Principales Hallazgos

- Las **mujeres y niñas de primera y segunda clase** fueron evacuadas casi en su totalidad, registrando tasas de supervivencia entre **96.81% y 100.00%**.

- El **género** actuó como el principal filtro de supervivencia, mientras que la **clase social** operó como un segundo filtro determinante.

- Un **niño varón de segunda clase** presentó una probabilidad de supervivencia del **100.00%**, mientras que un **niño de tercera clase** descendió a apenas **36.00%**.

- La **edad por sí sola no garantizaba la supervivencia**. Por ejemplo:
  - Niño de tercera clase → **36.00%**
  - Mujer adulta de primera clase → **97.53%**

## Conclusión

El análisis exploratorio demuestra que los tres factores con mayor impacto en la supervivencia fueron el género, la clase social y la edad, operando en ese orden jerárquico estricto. Las hipótesis planteadas en el Paso 1 fueron validadas: pertenecer a la primera clase cuadruplicó la probabilidad de rescate frente a la tercera clase en varones, y el segmento de niños mantuvo prioridad sobre los adultos. No obstante, el enfoque multivariado demostró ser superior, revelando que la edad no garantizaba la vida de forma aislada, ya que un niño de tercera clase tenía menor probabilidad de sobrevivir (36.00%) que una mujer adulta de primera clase (97.53%). Finalmente, la decisión de limpiar el dataset imputando la edad con la mediana preservó la estructura central sin distorsionar las tasas, mientras que omitir la variable "Cabin" evitó sesgar el modelo con un 77.10% de registros artificiales.