# Robótica
Prácticas Robótica MUVA



En primer lugar para la realización de la practiva tenemos que obtener la imagen de nuestro campo de visón. Para ello utilizamos la sentencia que nos proporcionan "HAL.getImage()". Con esta sentencia obtenems la imagen que vemos a conticuacón:

 ------incluir imagen vision inicial------ 

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## Filtro de color
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
 
 
 
