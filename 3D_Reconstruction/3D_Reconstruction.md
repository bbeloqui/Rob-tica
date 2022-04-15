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
  - HAL.getCameraPosition ('left/right'): para obtener la posición de la cámara izquierda/derecha desde ROS Driver Camera.
  - HAL.backproject ('left', puntos 2D): para retroproyectar el punto de imagen 2D en el espacio de punto 3D
  - HAL.graficToOptical ('left', puntos 2D): para transformar el Sistema de coordenadas de imagen en el Sistema de cámara

Ya que tenemos el haz de retroproyección, ahora es necesario proyectarlo en la cámara derecha. Se crea una función que crea una mascara con la línea epipolar a partir del rayo de retroproyección.
````
def epipolar_proyeccion (pos, rayo, tamaño_mascara, grosor=9):
    #calcula una proyeccion epipolar de un rayo a una imagen de camara
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
Se toman 2 puntos de la línea de retroproyección y se proyectan sobre la imagen de la derecha, luego se calcula la línea que pasa por ambos puntos de la imagen, obteniendo los puntos extremos. 

Se han utilizado las funciones proporcionadas por el entorno:
    - HAL.project('left', puntos 3D): para retroproyectar un espacio de punto 3D en el punto de imagen 2D.
    - HAL.opticalToGrafic('left', puntos 2D): para transformar el sistema de cámara en el sistema de coordenadas de imagen.

Una vez que se tiene el punto y su proyección epipolar, es necesario encontrar su contraparte, en este caso se opta por aplicar machTemplate, función para la imagen de la franja epipolar, multiplicando la imagen de la derecha por la mascara anteriormente nombrada.

falta imagen con la mascara

Esto se implementa con la función:
````
def homologo (puntos2d, imagen_left, imagen_right, ep_mascara, grosor=9):  
    #machTemplate y homologo
    global left, right
    
    pad = grosor // 2
    x, y = puntos2d[:2]
    template = imagen_left[x - pad:x + 1 + pad, y - pad:y + 1 + pad]
    
    res = cv2.matchTemplate(imagen_right * ep_mascara, template, cv2.TM_CCOEFF_NORMED)
    _, coeff, _, top_left = cv2.minMaxLoc(res)   #para encontar los valores maximos/minimos
     
    top_left = np.array(top_left)
    match_point = top_left[::-1] + pad
    
    return match_point, coeff
````
Dado un punto 2D y la mascara epilpolar es capaz de calcular su homólogo.

imagen resultado de toda la mascara en varios puntos

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------

Hata ahora ya tenemos lo pares de puntos homólogos, y ya se puede calcular ambos rayos de retroproyección y ver dondé se cruzan para calcular el punto 3D.
