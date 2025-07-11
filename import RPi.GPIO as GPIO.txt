import RPi.GPIO as GPIO
import time 

GPIO.setwarnings(False)
GPIO.setmode(GPIO.BCM)

CapteurG = 14
CapteurD = 15
LEDG = 15
LEDD = 7

MoteurG1 = 18
MoteurG2 = 23
MoteurD1 = 8
MoteurD2 = 1

GPIO.setup(CapteurG, GPIO.IN)
GPIO.setup(CapteurD, GPIO.IN)

GPIO.setup(LEDG, GPIO.OUT)
GPIO.setup(LEDD, GPIO.OUT)

GPIO.setup(MoteurG1, GPIO.OUT)
GPIO.setup(MoteurG2, GPIO.OUT)
GPIO.setuo(MoteurD1, GPIO.OUT)
GPIO.setup(MoteurD2, GPIO.OUT)

while True:
    if(GPIO.read(CapteurG)==False and GPIO.read(CapteurD)==False) :
        GPIO.output(MoteurG1, True)
        GPIO.output(MoteurG2, False)
        GPIO.output(MoteurD1, True)
        GPIO.output(MoteurD2, False)
        GPIO.output(LEDG,False)
        GPIO.output(LEDD,False)
        time.sleep(0.8)
    elif(GPIO.read(CapteurG)==True and (GPIO.read(CapteurD)==False):
         
        GPIO.output(MoteurG1, False)
        GPIO.output(MoteurG2, False)
        GPIO.output(MoteurD1, True)
        GPIO.output(MoteurD2, False)
        GPIO.output(LEDG,True)
        GPIO.output(LEDD,False)
        time.sleep(0.8)
    elif(GPIO.read(CapteurG)==False and (GPIO.read(CapteurD)==True):
        GPIO.output(MoteurG1, True)
        GPIO.output(MoteurG2, False)
        GPIO.output(MoteurD1, False)
        GPIO.output(MoteurD2, False)
        GPIO.output(LEDG,False)
        GPIO.output(LEDD,True)
        time.sleep(0.8)
        else:
         GPIO.output(MoteurG1,False)
         GPIO.output(MoteurG2,False)
         GPIO.output(MoteurD1,False)
         GPIO,output(MoteurD2,False)
         GPIO,output(LEDG,True)
         GPIO.output(LEDD,True)
        time.sleep(0.8)
     except KeyboardInterrupt:
        stop_robot()
        GPIO.cleanup()
