import cv2
import threading
import numpy as np
from PIL import ImageGrab
import pyautogui as au
import time
from random import randint
import operator

# No olviden crear una carpeta donde esten solo los simbolos del juego


# VARIABLES

xi = 537
yi = 232
xf = 1368
yf = 950

cuadrado = []
listacartas = []
countdos = 1
flag02 = False
flag01 = False



###### Detector de la pantalla de juego #########################################################################
def RecObjeto(template, image):
    test = False
    
    lista = []
    h, w = template.shape[:2]

    method = cv2.TM_CCOEFF_NORMED
    threshold = 0.88

    start_time = time.time()

    res = cv2.matchTemplate(image, template, method)
    count = 0
    # fake out max_val for first run through loop
    max_val = 1
    while max_val > threshold:
        min_val, max_val, min_loc, max_loc = cv2.minMaxLoc(res)

        if max_val > threshold:
            count += 1
            res[max_loc[1]-h//2:max_loc[1]+h//2+1, max_loc[0]-w//2:max_loc[0]+w//2+1] = 0   
            #image = cv2.rectangle(image,(max_loc[0],max_loc[1]), (max_loc[0]+w+1, max_loc[1]+h+1), (0,255,0) )
            test = True
            lista.append((max_loc[0],max_loc[1], max_loc[0]+w+1, max_loc[1]+h+1))
    
    #print(count)
    
    if test:
        return test, lista
    else:
        return False, lista
###### Detector de simbolos en las cartas #########################################################################
def simbolosmatch(img_rgb):
    import glob
    import os
    from PIL import ImageGrab
    import cv2
    import numpy as np
    count2 = 0
    #template = cv2.imread(r"C:/Users/blako/Downloads/botton_poker.jpg", 0)
    acciones = [cv2.imread(file, 0) for file in glob.glob(r'C:/Users/Elblakooo/Documents/roller flip/simbols/*.png')]
    img_gray = cv2.cvtColor(img_rgb, cv2.COLOR_BGR2GRAY)
    for template in acciones: 
        w, h = template.shape[::-1]
               

        res = cv2.matchTemplate(img_gray,template,cv2.TM_CCOEFF_NORMED)
        threshold = 0.75
        loc = np.where( res >= threshold)

        f = set()
        count2 +=1
        #print(count2)
        for pt in zip(*loc[::-1]):
            cv2.rectangle(img_rgb, pt, (pt[0] + w, pt[1] + h), (0,0,255), 2)

            sensitivity = 40
            f.add((round(pt[0]/sensitivity), round(pt[1]/sensitivity)))
            
            cv2.waitKey(0)
            cv2.destroyAllWindows()
            # Esto se tiene que mejorar deprecated
            if acciones.index(template) == 0:
                return "BNB"
            if acciones.index(template) == 1:
                return "BTC"
            if acciones.index(template) == 2:
                return "ETH"
            if acciones.index(template) == 3:
                return "LTC"
            if acciones.index(template) == 4:
                return "MOM"
            if acciones.index(template) == 5:
                return "RRR"
            else:
                return "0"







################################### Aqui comienza el juego ####################################################
while flag01 == False:
    image = np.array(ImageGrab.grab(bbox=(xi,yi,xf,yf)))
    image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
    #image = cv2.imread(r"C:/Users/Elblakooo/Documents/roller flip/frame.png",cv2.IMREAD_COLOR)
    template = cv2.imread(r"C:/Users/Elblakooo/Documents/roller flip/carta1.png", cv2.IMREAD_COLOR)

    coordenadas = RecObjeto(template, image)
    
    if coordenadas[0] == True:
        print("coordenadas")
        cuadrado = coordenadas[1]
        cv2.destroyAllWindows()
        flag01 = True
    
    cv2.imshow("tresh", image)
    if cv2.waitKey(1) & 0xff == ord('s'):
        
        break

cv2.destroyAllWindows()

for x in range(len(cuadrado)):
    listacartas.append(0)


print(listacartas)
print(cuadrado)

time.sleep(2)
############################### SEGUNDA PARTE DEL JUEGO, OSEA, CLICKEAR CARTAS #################
while flag02 == False:
    
    for x in cuadrado:
        time.sleep(0.5)
        flag03 = False
        mover = ((x[2]-x[0])/2+x[0] +xi), ((x[3]-x[1])/2+x[1] +yi)
        
        au.moveTo(mover)
        au.click()
        
        ###### UNA VEZ CLICKEADA LA CARTA, SE IDENTIFICA y SE AGREGA A LA LISTA EN EL MISMO INDEX QUE LA LISTA DE CARTAS VOLTEADAS #####
        while flag03 == False:
            print("intentando")
            time.sleep(0.4)
            image2 = np.array(ImageGrab.grab(bbox=(xi,yi,xf,yf)))
            image2 = cv2.cvtColor(image2, cv2.COLOR_BGR2RGB)
            crop1 = np.array(image2)[x[1]:x[3], x[0]:x[2]]        
            cv2.imwrite(r'C:/Users/Elblakooo/Documents/roller flip/testestest.png', crop1)
            res =  simbolosmatch(crop1)
            if res != None:
                print("puesta")
                print(res)

                if res in listacartas:
                    print("ya estaaaa")
                    indicecarta = listacartas.index(res)
                    time.sleep(0.7)
                    nv = cuadrado[indicecarta]
                    au.click(((nv[2]-nv[0])/2+nv[0] +xi), ((nv[3]-nv[1])/2+nv[1] +yi))
                    flag03 = True
                    time.sleep(1.2)
                listacartas.pop(cuadrado.index(x))
                listacartas.insert(cuadrado.index(x), res)
                flag03 = True
        print(listacartas)



      
            
       
        
