# Práctica 3D_Reconstruction

En objetivo principal de esta práctica es la reconstrucción de una escena tridimensional a partir de 2 imágenes por un visor estéreo calibrado.

[imagen de lapractica]

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Desarrollo

En primer lugar tendremos que buscar los puntos hómologos (pares). Para esto utilizamos el método Canny ya que es el que recomiendan para la realización de la práctica.
Canny nos dará como resultado los bordes utilizando la información del gradiente.

Partimos como ya se ha comentado antes de un visor estéreo que se compone de 2 cámaras (izquierda y derecha).

![camiz_camde](https://user-images.githubusercontent.com/10534733/163604644-ac18137e-a737-4530-9740-109a75885c9a.PNG)

Para la obtención de la imagen de cada cámara tendremos que ejecutar los comandos:
````
imagen_left = HAL.getImage('left')
imagen_rigth = HAL.getImage('right')
GUI.showImages(imagen_left,imagen_rigth,True)
````

Tras tener la imagen, aplicamos Canny a la imagen de la cámara izquierda y obtenemos este resultado:

![camara_y_canny](https://user-images.githubusercontent.com/10534733/163605683-225c04df-7ea2-464f-acd2-fa573348b8a6.PNG)

Una vez que ya tenemos los bordes, lo que hacemos a continuación, es obtener las coordenadas de los puntos que conforman los bordes (unos 18k). A la hora de representar estos puntos solo representaresmos unos 10.000 punto de forma aleatoria. Para ello usamos:
````
puntos2d = random.sample(puntos2d, puntos)
````
Donde puntos2d es la cantidad de puntos que forman los bordes y puntos los puntos que queremos que elija de forma aleatoria.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------

Ahora lo que calculamos es el rayo de retroproyección que pasa por el centro óptico de la cámara izquierda que pasa por ese punto y proyecta esta línea sobrela imagen obtenida por la cámara derecha. Para esto ejecutamos este código:
````
def rayo_direcional(pos, puntos2d):
    # El rayo va desde el centro optico a traves de los puntos2d

    y, x = puntos2d[:2]
    punto1 = HAL.getCameraPosition(pos)  
    # trans = HAL.graficToOptical(pos, [x, y, 1])
    punto2 = HAL.backproject(pos, HAL.graficToOptical(pos, [x, y, 1]))[:3] 

    return np.append(punto2 - punto1, [1]), np.append(punto1, [1])  
````
Donde esta función nos devuelve una recta formada por eun vector y un punto. En esta función utilizamos unas funciones que nos facilita el entorno:
  - HAL.getCameraPosition: que devuelve la posición de la cámara.
  - HAL.backproject: reproyecta un punto 2D al sistema de referencia 3D.
  - HAL.graficToOptical

