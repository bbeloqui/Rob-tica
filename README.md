# Robótica
## Práctica Follow Line



En primer lugar para la realización de la practiva tenemos que obtener la imagen de nuestro campo de visón. Para ello utilizamos la sentencia que nos proporcionan "HAL.getImage()". Con esta sentencia obtenems la imagen que vemos a conticuacón:

 ------incluir imagen vision inicial------ 

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Filtro de color
 Una vez que ya tenmos la imagen lo que nos interesa es quedarnos ( en forma binaria ) con la linea roja ya que es lo que tienemos que seguir. Para esto las imagenes que vamos recibiendo las convertimos a HSV, ya que los cambios de iluminación le afectan menos. Para extraer la linea vamos a dar valores máximos y mínimos a cada canal
 - Canal H:  min = 0, max = 255
 - Canal S:  min = 77, max = 255
 - Canal V:  min = 56, max = 255
Luego, tenemos que unir los 3 canales para formar nuestra imagen y una vez que la tenemos realizamos una binarización con la sentencia "threshold". Una vez realizado esto ya tenemos la línea aislada para poder tratarla.
 
````pitón
    img_hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
    line = cv2.inRange(img_hsv, min_hsv, max_hsv) 
    _, line = cv2.threshold(line, 248, 255, cv2.THRESH_BINARY)
````
 ----incluir imagen vision binaria -----

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Una vez que ya tenemos la obtención de la línea unicamente, el siguiente paso es obtener el contorno de la línea para poder "situarla" en la imagen. En mi caso divido mi parte de interes en 2 trozos ("podemos decir que la parte esntre las 2 lineas es lo que viene y de sde la 2 línea hasta el final es lo que tenemos enfrente"). E nla imagen que se muestra se puede ver lo antes explicado sobre las líneas.
Para la obtención de los bordes utilizamos la sentencia "cv2.findContours", pos teriormente si queremos podemos representarlo en la imagen que vamos devolviendo con la sentencia "cv2.drawContours"

````pitón
    contorno_up, _ = cv2.findContours(c_up, cv2.RETR_LIST, cv2.CHAIN_APPROX_SIMPLE)
    contorno_down, _ = cv2.findContours(c_down, cv2.RETR_LIST, cv2.CHAIN_APPROX_SIMPLE)
    cv2.drawContours(img[y0:yf, x0:xf], contorno_up, -1, (0, 255, 0), 1)
    cv2.drawContours(img[y1: y1f, x1:x1f], contorno_down, -1, (0, 255, 0), 1)
````
Una vez que ya tenemos el contorno obtenido mediante la sentencia "cv2.moments(contorno)" obtenemos la coordenada X, Y del centroide. En mi caso lo relizo 2 veces una para cada parte en la que tengo dividido mi parte de interes.
Ya tenemos las coordenadas de nuestros centroides por lo que podemos actualizarlas, ya que las habiamos inicializado a -1 si las coordenadas obtenidas son -1 se queda igual que las inicializadas ( en iteraciones siguientes, se quedara con el valor anterior obtenido) y si no..pues nos quedamos con las actuales (en cada iteración se actualizan las coodenadas de los centroides obtenidas)
Podemos obtener el error del centroide al centro, y así saber cuanto nos separamos de la vertical que seria el lugar perfecto del centroide (lo calculamos para ambos centroides) , X0 y X1 son 0 y mid_h es la mitad del tamaño de columnas de pixeles de la imagen.
````pitón
    err = (x_u + x0) - mid_h
    err_b = (x_d + x1) - mid_h
````

 ----incluir imagen vision normal con liíneas y centroides -----
                     
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------  
### PID 
Calcula un valor de error como la diferencia entre la salida deseada y la salida actual y aplica una correción basada en terminos proporcionales, integrales y derivados.
kp = da una salida que es proporcional al error actual. Si el error es cero kp=0
ki = elimina el error de compensación que acumula el controlador kp. Integra el error durante un periodo de tiempo hasta que el valor del error llega a 0.
kd = proporciona una salida que depende de la tasa de cambio o error con respecto al tiempo.

Traducido a código para la obtencion del PID
````pitón
    p_err = - kp * err
    d_err = - kd * (err - prev_err)
    i_err = - ki * accum_err
````
Donde err es el error del centroide superior, prev_err es el error de la iteración anterior y acumm_err el acumulado. Estos valores se irán actializando en cada itación.
El PID obtenido sera la suma de los 3 errores, que sera nuestra W.
````pitón
    pid = p_err + d_err + i_err
````


            
 
 
 
