# CARRITO-ESQUIVANDO-PELOTITAS-CON-ARDUINO-Y-OPENCV

### Universidad Nacional de San Antonio Abad del Cusco (UNSAAC)
### Departamento Académico de Informática
### Robótica y Procesamiento de Señales

#### Integrantes

- BOLAÑOS CCOPA, RONY				      -   154624
- CALLASACA ACUÑA, FERNANDO        -  140989
- LIMPE QUISPE, JERRY ANDERSON     -  140985
- MENDOZA CJUIRO, NILO FIDEL       -  144987
- SERRANO AMAU, FIORELA ELOISA     -  133968

#### Descripción Proyecto

Este trabajo está elaborado por alumnos de la Escuela Profesional de Ingeniería Informática y de Sistemas de la Universidad Nacional de San Antonio Abad del Cusco, los aportes vertidos aquí, están cimentados en conocimientos previos que se impartieron en la escuela profesional, además de investigación continua y asesoría de amigos, docentes, entre otros. El presente trabajo es parte de un proyecto mucho más grande, cuya finalidad está orientado a los estudiantes y es tener una idea latente de las capacidades de la robótica, conocer las aplicaciones de esta y, de ser posible, aportar con la mejora en el campo de la robótica.
 
## Código comentado

```javascript
#******************************************************************
#*********************** CARRITO POR BLUETOOTH ********************
#******************************************************************
# Asignatura: Robotica y Procesamiento de Señal
# Semestre:   2020-I
#******************************************************************
#******************************************************************

#============================ LIBRERIAS ============================
#Importamos la librerias
import cv2
import numpy as np
import urllib
import urllib.request
import math
import serial
import time
#Importamos la libreria serial para conectar Arduino con Python


#============================ PROGRAMA ============================
# Iniciamos la comunicacion con el arduino con el puerto COM4
# Y una comunicacion serial de 9600
ser =serial.Serial('COM4',9600)
#definimos un procedimiento para dibujar contornos y hallar coordenadas
def dibujar(mask,color,w,f):
  contornos,_ = cv2.findContours(mask, cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_SIMPLE)
  #Se recorre cada uno de los contornos encontrados
  IN=[]
  for c in contornos:
    #Determinamos el area en pixiles del contorno
    area = cv2.contourArea(c)
    #Si el area es mayor al valor se dibuja
    A=[]
    if area > 1000:
      r=math.sqrt(area/math.pi)
      p=r*2
      distancia = (w*f)/p
      #buscamos areas centrales
      M = cv2.moments(c)
      #un if por si el denominador es 0 y pueda ser indeterminado
      if (M["m00"]==0): M["m00"]=1
      #Encontramos coordenadas X y Y
      x = int(M["m10"]/M["m00"])
      y = int(M['m01']/M['m00'])
      A.append(x)
      A.append(y)
      A.append(distancia)
      IN.append(A)
      #mejora la visualizacion del contorno
      nuevoContorno = cv2.convexHull(c)
      #dibujamos un circulo de 7 pixeles en el centro del objeto encontrado XY 
      cv2.circle(image,(x,y),7,(0,255,0),-1)
      #para visualizar el texto
      mensaje="D = "+str(round(distancia,2))+" cm"
      cv2.putText(image,mensaje,(x,y),font,1,(0,0,255),2,cv2.LINE_AA)
      cv2.putText(image,'{},{}'.format(x,y),(x+10,y), font, 0.50,(0,255,0),1,cv2.LINE_AA)
      #dibujamos los contornos
      cv2.drawContours(image, [nuevoContorno], 0, color, 3)
  return IN
      

##Determinar la distancia Menor de un arreglo
def DistanciaMArreglo(A):
  
  if (len(A)==0):
    return 2000;
  else :
    menor=A[0][2]
    for k in range(1, len(A)):
      if(menor>A[k][2]):
        menor=A[k][2]
  return menor
##Modulo para determinar la distancia menor
def DistanciaMTotal(A,B,C):
  menor1=DistanciaMArreglo(A)
  menor2=DistanciaMArreglo(B)
  menor3=DistanciaMArreglo(C)
 
  menor=menor1
  if(menor>menor2):
    menor=menor2
    if(menor>menor3):
      menor=menor3
  if(menor>menor3):
    menor=menor3
  return menor
  
def EscribirMatriz(A):
  for k in range(len(A)):
    print(A[k][0],A[k][1],A[k][2])

#Creamos una variable de conexion IP con la camara
url = 'http://192.168.1.3:8080/shot.jpg'

#Definimos el rango de valores del azul
azulBajo = np.array([100,100,20],np.uint8)
azulAlto = np.array([125,255,255],np.uint8)



#Definimos el rango de valores del rojo
redBajo1 = np.array([0,100,20],np.uint8)
redAlto1 = np.array([5,255,255],np.uint8)

redBajo2 = np.array([175,100,20],np.uint8)
redAlto2 = np.array([179,255,255],np.uint8)

#Definimos el rango de valores del Violeta
violetaBajo = np.array([140, 50, 20], np.uint8)
violetaAlto = np.array([170, 255, 255], np.uint8)
#Definimos el rango de valores del Verde
verdeBajo = np.array([36, 50, 20], np.uint8)
verdeAlto = np.array([75, 255, 255], np.uint8)


font = cv2.FONT_HERSHEY_SIMPLEX
disP=30
#Iniciamos el programa en un bucle
while True:
  ser.write(str('5').encode())
  
  #Iniciamos la transmision de la camara
  imgResp = urllib.request.urlopen(url)
  imgNp = np.array(bytearray(imgResp.read()),dtype = np.uint8)
  #Almacenamos la transmision en una variable image
  image = cv2.imdecode(imgNp,-1)

  #Transformamos la imagen de BGR a HSV
  frameHSV = cv2.cvtColor(image,cv2.COLOR_BGR2HSV)
  #Obtenemos una imagen binaria del azul
  maskAzul = cv2.inRange(frameHSV,azulBajo,azulAlto)
 
  #Obtenemos una imagen binaria del rojo
  maskRed1 = cv2.inRange(frameHSV,redBajo1,redAlto1)
  maskRed2 = cv2.inRange(frameHSV,redBajo2,redAlto2)
  maskRed = cv2.add(maskRed1,maskRed2)
 
  #Obtenemos una imagen binaria del violeta
  maskVerde = cv2.inRange(frameHSV,verdeBajo,verdeAlto)
 
  #Dibujamos los contornos encontrados segun su color
  time.sleep(2)
  A=dibujar(maskAzul,(255,0,0),6,450)
  R=dibujar(maskRed,(0,0,255),6,653)
  V=dibujar(maskVerde,(0,255,0),6,653)
  cv2.imshow('frame',image)
  print("de A")
  EscribirMatriz(A)
  print("de R")
  EscribirMatriz(R)
  print("de V")
  EscribirMatriz(V)
  menor=DistanciaMTotal(A,R,V)
  if(menor==2000):
     for k in range (int(15)):
      ser.write(str('1').encode())
      ser.write(str('5').encode())
    
  if(menor>disP and menor<1000):
    dM=menor-disP
    print(dM)
    print("*****Avanzando*****")
    for k in range (int(dM)):
      ser.write(str('1').encode())
      ser.write(str('5').encode())
  
  #Iniciamos la transmision de la camara
  imgResp = urllib.request.urlopen(url)
  imgNp = np.array(bytearray(imgResp.read()),dtype = np.uint8)
  #Almacenamos la transmision en una variable image
  image = cv2.imdecode(imgNp,-1)

  #Transformamos la imagen de BGR a HSV
  frameHSV = cv2.cvtColor(image,cv2.COLOR_BGR2HSV)
  #Obtenemos una imagen binaria del azul
  maskAzul = cv2.inRange(frameHSV,azulBajo,azulAlto)
 
  #Obtenemos una imagen binaria del rojo
  maskRed1 = cv2.inRange(frameHSV,redBajo1,redAlto1)
  maskRed2 = cv2.inRange(frameHSV,redBajo2,redAlto2)
  maskRed = cv2.add(maskRed1,maskRed2)
 
  #Obtenemos una imagen binaria del violeta
  maskVerde = cv2.inRange(frameHSV,verdeBajo,verdeAlto)
 
  #Dibujamos los contornos encontrados segun su color
  time.sleep(2)
  A=dibujar(maskAzul,(255,0,0),6,450)
  R=dibujar(maskRed,(0,0,255),6,653)
  V=dibujar(maskVerde,(0,255,0),6,653)
  cv2.imshow('frame',image)
  print("de A")
  EscribirMatriz(A)
  print("de R")
  EscribirMatriz(R)
  print("de V")
  EscribirMatriz(V)
  menor=DistanciaMTotal(A,R,V)
  if(menor==2000):
     for k in range (int(15)):
      ser.write(str('1').encode())
      ser.write(str('5').encode())
  if(menor>disP and menor<1000):
    dM=menor-disP
    print(dM)
    print("*****Avanzando*****")
    for k in range (int(dM)):
      ser.write(str('1').encode())
      ser.write(str('5').encode())
#*******************Girar  a la derecha y captar imagen *************#
  print("**********girando a la derecha********************")
  for k in range (5):
    ser.write(str('7').encode())
    ser.write(str('5').encode())
  time.sleep(2)
  #Iniciamos la transmision de la camara
  imgResp = urllib.request.urlopen(url)
  imgNp = np.array(bytearray(imgResp.read()),dtype = np.uint8)
  #Almacenamos la transmision en una variable image
  image = cv2.imdecode(imgNp,-1) 
  #Transformamos la imagen de BGR a HSV
  frameHSV = cv2.cvtColor(image,cv2.COLOR_BGR2HSV)
  #Obtenemos una imagen binaria del azul
  maskAzul = cv2.inRange(frameHSV,azulBajo,azulAlto)
 
  #Obtenemos una imagen binaria del rojo
  maskRed1 = cv2.inRange(frameHSV,redBajo1,redAlto1)
  maskRed2 = cv2.inRange(frameHSV,redBajo2,redAlto2)
  maskRed = cv2.add(maskRed1,maskRed2)
 
  #Obtenemos una imagen binaria del violeta
  maskVerde = cv2.inRange(frameHSV,verdeBajo,verdeAlto)
  time.sleep(2)
  A=dibujar(maskAzul,(255,0,0),6,450)
  R=dibujar(maskRed,(0,0,255),6,653)
  V=dibujar(maskVerde,(0,255,0),6,653)
  print("de Derecho de  A")
  EscribirMatriz(A)
  print("de Derecho de R")
  EscribirMatriz(R)
  print("de Derecho de V")
  EscribirMatriz(V)
  cv2.imshow('frame',image)
  DmDe=DistanciaMTotal(A,R,V)
  print("MenorDerecha",DmDe)
  

#*******************Girar  a la izqueirda y captar imagen *************#
  print("******Girando Izquierda************")
  for k in range (10):
    ser.write(str('6').encode())
    ser.write(str('5').encode())
  time.sleep(2)
  #Iniciamos la transmision de la camara
  imgResp = urllib.request.urlopen(url)
  imgNp = np.array(bytearray(imgResp.read()),dtype = np.uint8)
  #Almacenamos la transmision en una variable image
  image = cv2.imdecode(imgNp,-1) 
  #Transformamos la imagen de BGR a HSV
  frameHSV = cv2.cvtColor(image,cv2.COLOR_BGR2HSV)
  #Obtenemos una imagen binaria del azul
  maskAzul = cv2.inRange(frameHSV,azulBajo,azulAlto)
 
  #Obtenemos una imagen binaria del rojo
  maskRed1 = cv2.inRange(frameHSV,redBajo1,redAlto1)
  maskRed2 = cv2.inRange(frameHSV,redBajo2,redAlto2)
  maskRed = cv2.add(maskRed1,maskRed2)
 
  #Obtenemos una imagen binaria del violeta
  maskVerde= cv2.inRange(frameHSV,verdeBajo,verdeAlto)
  time.sleep(2)
  A=dibujar(maskAzul,(255,0,0),6,450)
  R=dibujar(maskRed,(0,0,255),6,653)
  V=dibujar(maskVerde,(0,255,0),6,653)
  print("de Izquierdo A")
  EscribirMatriz(A)
  print("de Izquierdo R")
  EscribirMatriz(R)
  print("de Izquierdo V")
  EscribirMatriz(V)
  cv2.imshow('frame',image)
  DmIz=DistanciaMTotal(A,R,V)
  print("MenorIzquierda",DmIz)

#***********************************************************************#
  if(DmDe>DmIz):
    print("****Decision Derecha****")
    for k in range(10):
      ser.write(str('7').encode())
      ser.write(str('5').encode())
    time.sleep(2)
  else:
    print("***Decision Izquierda**")
    
  
  
  

  
  #dibujar(maskRosa,(255,0,255))

  #mostramos el video
  cv2.imshow('frame',image)
  #Rango valores 0-213 213-426 426-640
  if cv2.waitKey(1) and 0xFF == ord('s'):
    break

#Finalmente cerramos todas las ventanas que se hubieran podido abrirse
cv2.destroyAllWindows()

```


## Imágenes del Proyecto

Carrito:

![](https://github.com/FernandoCallasaca/CARRITO-ESQUIVANDO-PELOTITAS-CON-ARDUINO-Y-OPENCV/blob/main/Images/carrito.jpeg)

> Construido por nuestra compañera Fiorela.

Carrito en el circuito de pelotitas:

![](https://github.com/FernandoCallasaca/CARRITO-ESQUIVANDO-PELOTITAS-CON-ARDUINO-Y-OPENCV/blob/main/Images/carrito_pelotitas.jpeg)

> Construido por nuestra compañera Fiorela.

Circuito de pelotitas de plástico de 7cm de diámetro:

![](https://github.com/FernandoCallasaca/Problema-de-esquina-y-callejon---Robotica/blob/main/Images/car_circuit.jpeg)

> Construido por nuestra compañera Fiorela.

Diagrama de Flujo:

![](https://github.com/FernandoCallasaca/CARRITO-ESQUIVANDO-PELOTITAS-CON-ARDUINO-Y-OPENCV/blob/main/Images/diagrama_flujo.jpeg)

> Construido por nuestro compañero Rony Bolaños.

Diagrama del Circuito:

![](https://github.com/FernandoCallasaca/CARRITO-ESQUIVANDO-PELOTITAS-CON-ARDUINO-Y-OPENCV/blob/main/Images/diagrama_circuito.jpeg)

> Construido por nuestro compañero Jerry Anderson.
