```python
# %%
import seaborn as sns
import matplotlib.pyplot as plt
from googletrans import Translator
import pandas as pd
from matplotlib_venn import venn2
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np
from matplotlib import gridspec
import ptitprince as pt
from matplotlib.patches import Patch


```

# Pie Chart

La siguiente visualización es un Pie chart que muestra la distribucion economica de los gastos en los presupuestos de la Generalitat de Cataluña en el ejercicio del 2020. Datos extraidos de https://analisi.transparenciacatalunya.cat/Economia/Pressupostos-aprovats-de-la-Generalitat-de-Catalun/yd9k-7jhw


```python

# Inicializar el traductor
translator = Translator()

# Cargamos el dataset de iris
df_presuposts=pd.read_csv("Pressupostos_aprovats_de_la_Generalitat_de_Catalunya_20231105.csv")

# Filtrar los datos para el ejercicio de 2020 y la columna 'Ingrés / Despesa' para 'D' (Despesas/Gastos)
df_2020 = df_presuposts[(df_presuposts['Exercici'] == 2020) & (df_presuposts['Ingrés / Despesa'] == 'D')]

# Agrupar por 'Nom Agrupació' y sumar los 'Import Consolidat Sector Públic'
gastos_por_grupo = df_2020.groupby('Nom Agrupació')['Import Consolidat Sector Públic'].sum()

# Definir un umbral para el tamaño mínimo de las categorías para mostrar individualmente
umbral = gastos_por_grupo.sum() * 0.025  # Por ejemplo, el 5% del total

# Crear una serie donde todo lo que está por debajo del umbral se suma en la categoría 'Otros'
otros = gastos_por_grupo[gastos_por_grupo < umbral].sum()
gastos_agrupados = gastos_por_grupo[gastos_por_grupo >= umbral]
gastos_agrupados['Otros'] = otros  # Agregar la categoría 'Otros'

# Suponer que tenemos una lista de strings en catalán
nombres_en_catalan = gastos_agrupados.index.tolist()

# Traducir cada nombre usando Google Translate
nombres_traducidos = [translator.translate(nombre, src='ca', dest='es').text for nombre in nombres_en_catalan]

# Crear el pie chart con las categorías agrupadas
plt.figure(figsize=(10, 8), facecolor='white')
plt.pie(gastos_agrupados, labels=nombres_traducidos, autopct='%1.1f%%', startangle=140)
plt.axis('equal')  # Para que el pie chart sea circular
plt.title('Gastos por Grupo de Entidades en los presupuestos de Cataluña del 2020 (con categorías agrupadas)')
plt.show()
```


![png](output_2_0.png)


# Venn Diagrams

La siguiente visualización muestra las especies de amfibios que habitan en Brasil y Bolivia, este grafico permite observar las especies que son compartidas entre los dos paises y las que son diferentes. Datos extraidos de https://datacatalog.worldbank.org/search/dataset/0063384/Global-Species-Database


```python

# Cargamos el dataset de crecimiento de plantas
df_animales=pd.read_csv("Species_Database_wb_datanam.csv")
# Agrupamos los datos por la columna 'binomial' y luego extraemos una lista de países para cada grupo
species_countries = df_animales.groupby('binomial')['wb_datanam'].apply(list).to_dict()

# Filtramos las especies que se encuentran en Brasil o Bolivia
brazil_species = set(df_animales[df_animales['wb_datanam'] == 'Brazil']['binomial'])
bolivia_species = set(df_animales[df_animales['wb_datanam'] == 'Bolivia']['binomial'])

plt.figure(figsize=(10, 10), facecolor='white')
venn = venn2([brazil_species, bolivia_species], ('Brasil', 'Bolivia'))
plt.title("Especies de Anfibios que habitan en Brasil y Bolivia")


# Asigna colores específicos a cada sección del diagrama de Venn
venn.get_patch_by_id('10').set_color('red')  # Especies exclusivas de Brasil
venn.get_patch_by_id('01').set_color('green')   # Especies exclusivas de Bolivia
venn.get_patch_by_id('11').set_color('orange')  # Especies compartidas

# Añade una leyenda personalizada
legend_elements = [Patch(facecolor='red', label='Especies exclusivas de Brasil'),
                   Patch(facecolor='orange', label='Especies compartidas'),
                   Patch(facecolor='green', label='Especies exclusivas de Bolivia')]
plt.legend(handles=legend_elements, title="Leyenda")

# Mostrar el gráfico
plt.show()
```


![png](output_4_0.png)


# Rain cloud Plots

La siguiente visualización muestra la diferencia de salarios en India respecto a los Data Scientist y los Data Analysts. Para realizar la visualización se han excluido outliers en los salarios a traves del IQR. Datos extraidos de https://www.kaggle.com/datasets/iamsouravbanerjee/analytics-industry-salaries-2022-india/


```python


df_salary=pd.read_csv("salary\Partially Cleaned Salary Dataset.csv")
# Filtramos solo los datos de 'Data Scientist' y 'Data Analyst'
relevant_jobs = df_salary[df_salary['Job Title'].isin(['Data Scientist', 'Data Analyst'])]

# Conversión de salarios de rupias a dólares
conversion_rate = 0.012
relevant_jobs['Salary'] = relevant_jobs['Salary'] * conversion_rate


# Calculamos el IQR
Q1 = relevant_jobs['Salary'].quantile(0.25)
Q3 = relevant_jobs['Salary'].quantile(0.75)
IQR = Q3 - Q1

# Definimos los límites para los outliers
lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

# Filtramos los outliers
filtered_jobs = relevant_jobs[(relevant_jobs['Salary'] >= lower_bound) & (relevant_jobs['Salary'] <= upper_bound)]


# Creamos los graficos para realizar el rain plot
f, ax = plt.subplots(figsize=(14, 10), facecolor='white')
ort="h"; pal = "Set2"

ax=pt.half_violinplot( x = filtered_jobs["Salary"], y = filtered_jobs["Job Title"], data = filtered_jobs[["Job Title","Salary"]], palette = pal, bw = .2, cut = 0.,
                      scale = "area", width = .6, inner = None, orient = ort)
ax=sns.stripplot( x = filtered_jobs["Salary"], y = filtered_jobs["Job Title"], data = filtered_jobs[["Job Title","Salary"]], palette = pal, edgecolor = "white",
                 size = 3, jitter = 1, zorder = 0, orient = ort, ax = ax )
ax=sns.boxplot( x = filtered_jobs["Salary"], y = filtered_jobs["Job Title"], data = filtered_jobs[["Job Title","Salary"]], color = "black", width = .15, zorder = 10,\
            showcaps = True, boxprops = {'facecolor':'none', "zorder":10},\
            showfliers=True, whiskerprops = {'linewidth':2, "zorder":10},\
               saturation = 1, orient = ort)

# Añadimos los titulos del grafico
ax.set_xlabel('Salario (en USD)')
ax.set_ylabel('Posición laboral')
plt.title("Comparación de Salarios entre Científicos de Datos y Analistas de Datos (Sin Valores Atípicos) en Dólares")
plt.show()

```

    <ipython-input-14-fdb9662adfad>:7: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      relevant_jobs['Salary'] = relevant_jobs['Salary'] * conversion_rate  # Convierte los salarios a dólares
    


![png](output_6_1.png)



```python

```
