# Práctica 3D_Reconstruction

El objetivo principal de esta práctica es la reconstrucción de una escena tridimensional a partir de 2 imágenes por un visor estéreo calibrado.

![inicio](https://user-images.githubusercontent.com/10534733/163675761-85a4c77c-17cf-4717-94ad-be1ff0fadc5f.PNG)

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Desarrollo

En primer lugar tendremos que buscar los puntos hómologos (pares). Para esto utilizamos el método Canny ya que es el que recomienda para la realización de la práctica.
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

Una vez que ya tenemos los bordes, lo que hacemos a continuación es obtener las coordenadas de los puntos que conforman los bordes (unos 18k). A la hora de representar estos puntos solo representaresmos unos 10.000 puntos de forma aleatoria. Para ello usamos:
````
puntos2d = random.sample(puntos2d, puntos)
````
Donde puntos2d es la cantidad de puntos que forman los bordes y puntos los puntos que queremos que elija de forma aleatoria.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------

Ahora lo que calculamos es el rayo de retroproyección que pasa por el centro óptico de la cámara izquierda que pasa por ese punto y proyecta esta línea sobre la imagen obtenida por la cámara derecha. Para ésto ejecutamos este código:
````
def rayo_direcional(pos, puntos2d):
    y, x = puntos2d[:2]
    punto1 = HAL.getCameraPosition(pos)  
    # trans = HAL.graficToOptical(pos, [x, y, 1])
    punto2 = HAL.backproject(pos, HAL.graficToOptical(pos, [x, y, 1]))[:3] 

    return np.append(punto2 - punto1, [1]), np.append(punto1, [1])  
````
Donde esta función nos devuelve una recta formada por un vector y un punto. En esta función utilizamos unas funciones que nos facilitan el entorno:
  - HAL.getCameraPosition ('left/right'): para obtener la posición de la cámara izquierda/derecha desde ROS Driver Camera.
  - HAL.backproject ('left', puntos 2D): para retroproyectar el punto de imagen 2D en el espacio de punto 3D.
  - HAL.graficToOptical ('left', puntos 2D): para transformar el sistema de coordenadas de imagen en el sistema de cámara.

Ya que tenemos el haz de retroproyección, ahora es necesario proyectarlo en la cámara derecha. Se crea una función que crea una máscara con la línea epipolar a partir del rayo de retroproyección.
````
def epipolar_proyeccion (pos, rayo, tamaño_mascara, grosor=9):
    vd0 = rayo[0] + rayo[1]
    proyeccion_vd0 = HAL.project(pos, vd0)
    
    vd1 = (10*rayo[0]) + rayo[1]
    proyeccion_vd1 = HAL.project(pos, vd1)
    #sistema de camara a sistema coor de la imagen
    p0 = HAL.opticalToGrafic(pos, proyeccion_vd0)
    p1 = HAL.opticalToGrafic(pos, proyeccion_vd1)
    
    vector = p1-p0
    
    rectay = lambda x, v:(v[1] * (x - p0[0]) / v[0]) + p0[1]
    
    p0 = np.array([0, rectay(0, vector)]).astype(np.int)
    p1 = np.array([tamaño_mascara[1], rectay(tamaño_mascara[1], vector)]).astype(np.int)
    
    mascara = np.zeros(tamaño_mascara)
    cv2.line(mascara, tuple(p0), tuple(p1), (1,1,1), grosor)
    
    return mascara.astype(bool)
````

![izq_y_mascara](https://user-images.githubusercontent.com/10534733/163862873-87403b95-8b2c-4d59-a542-6dc2d5b5b05c.PNG)

Se toman 2 puntos de la línea de retroproyección y se proyectan sobre la imagen de la derecha, luego se calcula la línea que pasa por ambos puntos de la imagen, obteniendo los puntos extremos. 

Se han utilizado las funciones proporcionadas por el entorno:
  - HAL.project('left', puntos 3D): para retroproyectar un espacio de punto 3D en el punto de imagen 2D.
  - HAL.opticalToGrafic('left', puntos 2D): para transformar el sistema de cámara en el sistema de coordenadas de imagen.

Una vez que se tiene el punto y su proyección epipolar, es necesario encontrar su contraparte, en este caso se opta por aplicar machTemplate, función para la imagen de la franja epipolar, multiplicando la imagen de la derecha por la máscara anteriormente nombrada.


Esto se implementa con la función:
````
def homologo (puntos2d, imagen_left, imagen_right, ep_mascara, grosor=9):  
    global left, right
    
    pad = grosor // 2
    x, y = puntos2d[:2]
    template = imagen_left[x - pad:x + 1 + pad, y - pad:y + 1 + pad]
    
    res = cv2.matchTemplate(imagen_right * ep_mascara, template, cv2.TM_CCOEFF_NORMED)
    _, coeff, _, top_left = cv2.minMaxLoc(res)   
     
    top_left = np.array(top_left)
    match_point = top_left[::-1] + pad
    
    return match_point, coeff
````
Dado un punto 2D y la máscara epilpolar es capaz de calcular su homólogo.

Hasta ahora ya tenemos los pares de puntos homólogos, y ya se pueden calcular ambos rayos de retroproyección y ver dondé se cruzan para calcular el punto 3D.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Resultados

Los resultados de la práctica se muestran en el siguiente video y algunas imágenes. Los resultados que se ven no muestran todos los puntos posibles, ya que por tiempo no consigo mostrar todos ya que requiere mucho tiempo y el entorno cierra la comunicacion con el Docker. Se representan aproximadamente 10.000 putnos.

![3d4](https://user-images.githubusercontent.com/10534733/163683511-333d9cc9-64af-41b4-8991-7a5e60d75e5d.PNG)
![3d1](https://user-images.githubusercontent.com/10534733/163683513-45b30f06-9a09-4c47-9c8c-7be1fee99893.PNG)
![3d2](https://user-images.githubusercontent.com/10534733/163683514-ae084c48-19a5-4849-83e9-e614c86d2b75.PNG)
![3d3](https://user-images.githubusercontent.com/10534733/163683515-605a2803-518e-46f8-a700-cd3ad84b1d24.PNG)

https://user-images.githubusercontent.com/10534733/163683648-0e16a888-ceb3-4bdd-8ac2-16eb4dd6f98b.mp4



