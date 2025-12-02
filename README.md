# DETECCIÓN DE JUGADORES DE FÚTBOL CON YOLOv8

## Introducción

El objetivo de este proyecto fue desarrollar un sistema automático para detectar jugadores, árbitros, arqueros y la pelota en imágenes y videos de partidos de fútbol.
La detección robusta de jugadores es un problema desafiante debido a variaciones de: <br> <br>
•	iluminación <br>
•	distancia a la cámara <br>
•	colores de camisetas <br>
•	resoluciones distintas <br>
•	movimientos rápidos <br> <br>
Para este trabajo se utilizaron modelos YOLOv8, un modelo preentrenado específico para fútbol disponible en HuggingFace y dos datasets:
1.	Football1 – Extraido de Roboflow
2.	Nacional - dataset propio anotado con Roboflow
El proyecto incluyó, fine tunear el modelo encontrado, evaluación cruzada, análisis de métricas, análisis de errores y  extracción de frames de video.

## Datasets

### Football1 (Roboflow)
Dataset generalista de partidos internacionales, con vistas tribuna y televisivas.
Clases: player, goalkeeper, referee, ball.
Distribución: <br>
•	Train: 1053 <br>
•	Valid: 259 <br>
•	Test: 126 <br>

Este dataset fue descargado desde la pagina de Roboflow en formato YOLOv8. Fue utilizado para finetunear el modelo de huggingface.
Pagina del Dataset: https://universe.roboflow.com/football-innq8/football-i1lvk

<img width="1298" height="424" alt="image" src="https://github.com/user-attachments/assets/6cf57b97-1c9f-41a6-8c90-9b3ca068290b" />


### Nacional (dataset propio)

Dataset creado a partir de videos del Club Nacional de Football, donde se seleccionaron a mano 350 imagenes. Para etiquetar cada imagen se utilizo una herramienta de Roboflow.
Incluye las distintas camisetas (local, visitante, alternativas).
Tamaño total: 350 imágenes.
Distribución: <br>
•	Train: 200 <br>
•	Valid: 20 <br>
•	Test: 130 <br>

Este dataset es más difícil debido a la similitud de colores, ángulos poco comunes y mayor ruido visual.
Para realizar este dataset, se tomaron 350 imágenes de distintos partidos de la temporada 2025, contemplando que en todos los partidos se tomaran la misma cantidad de imágenes. Tambien se tuvo en cuenta que Nacional estuviera con las 4 equipaciones disponibles (Blanca, roja, azul y celeste). Una vez realizadas las imágenes se etiquetaron manualmente una por una para tener el dataset completo.
Este dataset cuenta con 2 versiones. La versión 1 tiene todas las imágenes en test, de forma que se utilizara para evaluar los distintos modelos que se planteen. Sin embargo, la versión 2 tiene la distribución antes mencionada, este dataset se utilizara para finetunear los modelos.
Pagina del Dataset: https://app.roboflow.com/football-2shry/futbol-uruguayo-ylfyu/4

<img width="1298" height="457" alt="image" src="https://github.com/user-attachments/assets/4d8fa35b-82f5-4b03-955a-e36e3f4b168d" />


## Modelos evaluados
Para el proyecto se tomaron como referencia los siguientes modelos. El modelo YOLOv8 se utilizo solo para comparar genéricamente al principio de las pruebas, luego se dejo decontemplar ya que al no estar preentrenado todas las categorías son igual de probables y no era muy representativas sus respuestas.

### YOLOv8n (baseline)

Es la versión más liviana de la familia YOLOv8. Se diseñó para lograr un equilibrio entre velocidad, uso de memoria y precisión. Este modelo tiene una arquitectura compacta con aproximadamente 11 millones de parámetros.

### Modelo HuggingFace — “yolo-v8-football-players-detection”
Modelo preentrenado en un dataset de fútbol internacional.
Generaliza bien dentro de su dominio original.

### Modelo Futbol_Finetune
HuggingFace fine-tuneado sobre Football1.
Produce excelente desempeño dentro de ese dataset.

### Modelo Nacional_Finetune
Re-entrenamiento del modelo HugginFace de 10 épocas usando solo el dataset de Nacional v2.
Mejora el desempeño al adaptarse al dominio específico.

## Análisis global de los modelos y datasets

Los resultados se van a comparar en los siguientes modelos

•	hf_football → modelo preentrenado de HuggingFace <br>
•	finetune_football → modelo hf_football fine-tuneado sobre Football1 <br>
•	hf_Nacional → modelo fine-tuneado sobre el dataset Nacional <br>

Evaluados en dos datasets: <br>
•	football1 <br>
•	Nacional <br>

Los valores reportan: <br>
•	Precisión <br>
•	Recall <br>
•	mAP50 <br>

A continuación se presentan los resultados de cada modelo 

### Modelo: hf_football


| Dataset    | Precision | Recall |  mAP50 |
|------------|----------:|-------:|-------:|
| football1  |    0.0777 |  0.316 | 0.0647 |
| Nacional   |    0.2670 |  0.210 | 0.1840 |

El modelo de hugging face, en el dataset football1 rinde mal, ya que tiene una precisión y un recall muy bajos. Esto indica que el dataset original de HF no coincide con el estilo visual de football1.
Sin embargo, con el dataset de Nacional, sorprendentemente mejora un poco (más precisión), pero sigue siendo un modelo débil.
Esto da a entender que el modelo de HuggingFace no está entrenado en imágenes similares ni a football1 ni a Nacional, por lo cual no generaliza bien.

## Modelo: finetune_football (fine tuned en Football1)


| Dataset    | Precision | Recall |  mAP50 |
|------------|----------:|-------:|-------:|
| football1  |     0.860 |  0.815 |  0.840 |
| Nacional   |     0.097 |  0.262 |  0.064 |

Este modelo funciona extremadamente bien para su dataset de entrenamiento. Sin embargo, para el dataset Nacional cae fuertemente. Esto significa que el modelo aprendió muy bien football1, pero no generaliza a imágenes del GPC, camisetas de Nacional, otras cámaras, otras iluminaciones.

### Modelo: hf_Nacional (finetune en Nacional)


| Dataset    | Precision | Recall |  mAP50 |
|------------|----------:|-------:|-------:|
| football1  |    0.0787 |  0.710 |  0.137 |
| Nacional   |    0.8495 |  0.714 | 0.7799 |

Este es el mejor modelo para las imagenes del dataset Nacional, ya que detecta muy bien a los jugadores reales, comete pocos falsos positivos, el mAP50 es relativamente bueno. Para el otro dataset, tiene un recall sorprendentemente alto pero baja precisión. Esto significa que encuentra muchos jugadores, pero confunde cosas y no se encuentra calibrado para el estilo de este dataset.
Es el mejor modelo para videos reales de Nacional, y su generalización inversa es mejor que la del finetune_football.

<img width="1107" height="830" alt="image" src="https://github.com/user-attachments/assets/6a4e702a-6bdc-4773-b9bb-15194026c7a1" />

Los errores comunes de este modelo en el dataset de Nacional fueron 

•	player ↔ referee <br>
•	goalkeeper ↔ player <br>
•	ball ↔ background <br>

Estos errores visuales son coherentes con la naturaleza del dataset.
<p float="left">
  <img width="35%" height="1133" alt="image" src="https://github.com/user-attachments/assets/29e7e325-be86-484f-88aa-f7ea39412e9c" />
  <img width="35%" height="1125" alt="image" src="https://github.com/user-attachments/assets/1741d77f-3870-45ef-9137-e18b68fc7dec" />
  <img width="35%" height="1125" alt="image" src="https://github.com/user-attachments/assets/b7fb3d9f-6867-4294-8ce8-ed92a3f1c28a" />
</p>


## Procesamiento de Video 

para realizar esta parte se implementó: <br>
•	extracción de fragmentos de video por tiempo <br>
•	conversión a frames configurando FPS deseado <br>
•	inferencia YOLO sobre cada frame <br>
•	reconstrucción del video detectado <br>
Este pipeline permite comparar modelos visualmente y analizar jugadas específicas.

<p float="left">
  <img src="Videos/gif/video_3m30s.gif" width="45%" />
  <img src="Videos/gif/video_detectado_3m30s.gif" width="45%" />
</p>

<p float="left">
  <img src="Videos/gif/video_5m10s.gif" width="45%" />
  <img src="Videos/gif/video_detectado_5m10s.gif" width="45%" />
</p>

<p float="left">
  <img src="Videos/gif/video_8m00s.gif" width="45%" />
  <img src="Videos/gif/video_detectado_8m00s.gif" width="45%" />
</p>

## Conclusiones

•	Los modelos finetuneados son muy sensibles a los dataset que se utilizaron para reentrenar <br>
•	Ninguno de los dos modelos generaliza. <br>
•	El modelo Hf_Nacional generaliza bien en el futbol uruguayo, esto se vio reflejado en los videos. <br>
•	Más datos en el dataset de nacional mejorarían aun mas el rendimiento del modelo <br>


## Trabajo futuro
•	Implementar e integrar tracking, en videos o en tiempo real. <br>
•	Aumentar el dataset de Nacional con mas imágenes <br>
•	Integrar reconocimiento de camisetas o equipos <br>
•	Detectar otras cosas como líneas, arcos, zonas, goles, etc. <br>









