# Visualización de Datos

* **Asignatura:** Visualización de Datos
* **Titulación:** Grado en Ciencia e Ingeniería de Datos · Universidad Autónoma de Madrid
* **Integrantes:** Jorge Rodríguez, Marc Martínez y Juan Fernández

---

## 1. Descripción

La práctica trabaja con datos del universo de Juego de Tronos y desarrolla visualizaciones de dos tipos:

* **Grafos con NetworkX:** relaciones entre personajes, medidas de centralidad y comunidades.
* **Mapas con Folium:** distribución geográfica de las apariciones y recorrido de los lugares visitados en un episodio.
* **Grafos interactivos con PyVis (opcional):** exploración de la red mediante simulaciones físicas e interacción.

El desarrollo principal está en el notebook Jupyter de la práctica.

---

## 2. Estructura de archivos y datos

El notebook espera encontrar los datos en la misma carpeta desde la que se ejecuta:

```text
practica/
├── practica2.ipynb
├── game_of_thrones/
│   └── data/
│       ├── characters.json
│       ├── episodes.json
│       └── ...
├── GOT-Map/
│   ├── land.geojson
│   └── places.geojson
├── places.geojson.json
└── graficos/
    ├── parte1_networkx/
    ├── parte2_folium/
    └── parte3_pyvis_opcional/
```

### Datos de Juego de Tronos

La carpeta `game_of_thrones/data/` contiene los ficheros JSON del conjunto de datos `game-of-thrones-data`. Entre ellos se utilizan `characters.json` y `episodes.json`.

### Datos geográficos

* `GOT-Map/land.geojson`: geometría de las superficies terrestres.
* `GOT-Map/places.geojson`: ciudades y castillos.
* `places.geojson.json`: versión ampliada de los lugares que incorpora también regiones y mares, por ejemplo *The North*, *The Crownlands* o *The Narrow Sea*. Esta versión permite asociar las localizaciones de las escenas con coordenadas.

> **Nota:** Si las carpetas de datos no están presentes pero se encuentran los ZIP originales en la carpeta del notebook, la primera celda de código intenta descomprimirlos automáticamente.

---

## 3. Requisitos e instalación

La práctica se ha verificado con **Python 3.12**.

### Crear un entorno virtual

**En Linux o macOS:**
```bash
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
```

**En Windows (PowerShell):**
```powershell
py -3.12 -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
```

### Instalar dependencias

```bash
python -m pip install ipykernel numpy pandas scipy networkx matplotlib folium branca pyvis
```

`pyvis` solo es necesario para la parte opcional. `ipykernel` permite seleccionar el entorno como kernel de Jupyter/VS Code y `scipy` se utiliza en los layouts y en PageRank de NetworkX.

### Abrir y ejecutar el notebook

1. Abre la carpeta de la práctica en VS Code.
2. Abre el notebook `.ipynb`.
3. Selecciona el intérprete/kernel del entorno `.venv`.
4. Reinicia el kernel si acabas de instalar las dependencias.
5. Ejecuta todas las celdas en orden.

En Jupyter también se puede iniciar el servidor desde el entorno virtual e abrir el notebook desde el navegador.

---

## 4. Contenido de la práctica

### Parte 1. Visualización de grafos con NetworkX

#### 1.1 Construcción del grafo de relaciones

Se explora `characters.json` y se construye una red de personajes. El nombre se utiliza como identificador: los 389 registros del fichero se reducen a 370 nombres únicos.

Se seleccionan seis tipos de relación:

| Relación | Dirección | Interpretación |
| :--- | :--- | :--- |
| **Parentesco** | Dirigida | Padre/madre $\rightarrow$ hijo/a |
| **Hermanos** | No dirigida | Relación simétrica |
| **Matrimonio** | No dirigida | Relación simétrica |
| **Asesinato** | Dirigida | Asesino $\rightarrow$ víctima |
| **Servicio** | Dirigida | Sirviente $\rightarrow$ señor |
| **Alianza** | No dirigida | Relación simétrica |

El grafo se implementa como `networkx.MultiDiGraph`, ya que puede haber varias relaciones distintas entre dos personajes. Las relaciones simétricas se almacenan una sola vez por pareja.

Se comparan `spring_layout` y `kamada_kawai_layout`. Para la componente gigante se utiliza Kamada-Kawai, que separa mejor las familias y sitúa entre ellas a los personajes que actúan como puente.

En la visualización:
* El tamaño de los nodos representa el grado.
* El color de los nodos representa la casa.
* El color de las aristas representa el tipo de relación.
* Las relaciones dirigidas se muestran con línea continua y flecha; las no dirigidas, con línea discontinua.
* La vista principal etiqueta los personajes más relevantes para evitar saturación.

#### 1.2 Análisis de centralidad y relevancia de personajes

Se calculan cinco medidas:

* **Grado (`degree`):** número de conexiones directas.
* **Intermediación (`betweenness`):** frecuencia con la que un nodo aparece en caminos mínimos entre otros nodos.
* **Cercanía (`closeness`):** proximidad a los demás nodos alcanzables.
* **Vector propio (`eigenvector`):** importancia en función de la importancia de los vecinos.
* **PageRank:** importancia que se transmite siguiendo la dirección de las relaciones.

Las cuatro primeras se calculan sobre una proyección simple no dirigida de los 370 personajes. PageRank utiliza las direcciones originales y representa en ambos sentidos las relaciones simétricas.

Se generan comparaciones de los diez personajes principales para cada métrica.

#### 1.3 Subredes y comunidades

Se estudian subredes y agrupaciones de personajes para observar cómo se organiza la red de relaciones. Las visualizaciones permiten examinar las conexiones entre personajes y casas y reconocer agrupaciones dentro del grafo.

---

### Parte 2. Visualización de mapas con Folium

#### 2.1 Mapa base

Se cargan las geometrías GeoJSON del territorio y los lugares con Folium. El mapa permite representar las localizaciones del universo de Juego de Tronos y trabajar con las coordenadas asociadas a las escenas.

#### 2.2 Importancia de cada ubicación

Se agregan las apariciones de las localizaciones a partir de los episodios y escenas. La visualización distingue regiones/mares de ciudades/castillos mediante capas activables.

El tamaño y el color de los círculos codifican el número de episodios en los que aparece cada lugar. Los *tooltips* muestran información breve y los *popups* ofrecen una ficha más completa, con datos como episodios, escenas, minutos en pantalla, temporadas, primera aparición, sublugares y personajes frecuentes.

#### 2.3 Ruta de un episodio

La ruta se construye a partir de las escenas de un episodio, utilizando `location` y `subLocation`. Las escenas consecutivas que ocurren en el mismo lugar se agrupan en una parada.

La visualización incorpora:
* Marcadores numerados según el orden de las paradas.
* Un marcador único cuando un lugar aparece varias veces, mostrando los números de sus paradas.
* Trayectos coloreados cronológicamente con la escala `viridis`.
* Flechas para indicar el sentido del recorrido.
* Arcos separados para distinguir viajes de ida y vuelta y repeticiones.
* *Tooltips* y *popups* con información de lugares y trayectos.

El notebook genera rutas para los episodios seleccionados mediante las variables `TEMPORADA` y `EPISODIO`. Incluye, entre otros ejemplos, **T2E05 — The Ghost of Harrenhal**.

---

### Parte 3. Grafos interactivos con PyVis (opcional)

Esta parte es opcional y no se evalúa. Exporta el grafo de relaciones a HTML mediante PyVis.

Se generan dos configuraciones físicas:
* `Barnes-Hut`
* `ForceAtlas2Based`

Los nodos se colorean según la casa y su tamaño depende de la intermediación. Las aristas distinguen los tipos de relación, muestran flechas para las dirigidas y líneas discontinuas para las no dirigidas. Los *tooltips* incluyen información de personajes y relaciones. La interfaz permite explorar la red, seleccionar nodos y ajustar los parámetros físicos.

---

## 5. Resultados generados

Al ejecutar el notebook se crea la carpeta `graficos/` con esta estructura:

```text
graficos/
├── parte1_networkx/
│   └── figuras y visualizaciones exportadas en PDF
├── parte2_folium/
│   ├── mapas interactivos en HTML
│   └── versiones estáticas en PDF
└── parte3_pyvis_opcional/
    ├── 3_grafo_barnes_hut.html
    └── 3_grafo_force_atlas_2based.html
```

Los mapas de Folium se guardan como HTML interactivo y también como PDF estático. Los grafos de PyVis se exportan como HTML.

---

## 6. Notas de ejecución

* Ejecuta las celdas en orden desde el principio para que se creen las rutas y variables necesarias.
* Comprueba que los JSON y GeoJSON están en las ubicaciones indicadas en la sección de estructura.
* Los mapas y grafos HTML interactivos utilizan bibliotecas JavaScript remotas, por lo que necesitan conexión a Internet al abrirse.
* Si no se instala PyVis, el notebook omite la parte opcional.
* Los PDF se generan con Matplotlib.

---

## 7. Tecnologías utilizadas

* Python 3.12
* Jupyter Notebook / VS Code
* NumPy y pandas
* SciPy
* NetworkX
* Matplotlib
* Folium y Branca
* PyVis (opcional)
* Datos JSON y geometrías GeoJSON