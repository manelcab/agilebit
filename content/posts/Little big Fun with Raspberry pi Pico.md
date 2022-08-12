---
title: Little big fun with raspberry pi Pico
subtitle: How to build a "mini" bright fan controlled by a motor driver and raspberry pi Pico
category:
  - PI
author: lenambac
date: 2022-08-10T18:39:56.800Z
featureImage: /uploads/fan.jpg
---

Hi all, in this post we are going to build a "mini" fan where we can adjust the speed motor and show visuals effects by neopixels (WS2812b).

In the following [post](https://www.agilebit.io/raspberry-pi-pico-and-neopixels) there are more information about how to manage neopixels

The fan will have the following controls:
- General switch
- Normal/Turbo switch
- Speed potentiometer
- Function button

The normal/turbo switch connects the fan motor to the driver motor or directly to the batteries, so in the first case you can adjust the speed of the motor (by the potencionmeter) and in the second case you can achieve the maximum motor speed.


There are twelve WS2912b leds around the fan controlled by the raspberry pi Pico, by using the function button you can switch the several visual effects of them



## 1. Components

Components list:

| \#  | Description                                                                                                    | Quant. |
| --- | -------------------------------------------------------------------------------------------------------------- | ------ |
| 1   | Neopixels - WS2812B                                                                                            | 12     |
| 2   | Raspberry pi pico                                                                                              | 1      |
| 3   | [DRV8833](https://www.amazon.es/gp/product/B08GM2XLMD/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)         | 1      |
| 4   | DC mini Motor                                                                                                  | 1      |
| 5   | Potenciometer                                                                                                  | 1      |
| 6   | [Switch](https://www.amazon.es/gp/product/B00F4MG7HY/ref=ppx_yo_dt_b_asin_title_o00_s01?ie=UTF8&psc=1)         | 1      |
| 7   | [Switch 6 pins](https://www.amazon.es/gp/product/B07Z8Z81T7/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1)  | 1      |
| 8   | [Battery holder](https://www.amazon.es/gp/product/B00QLHE5VQ/ref=ppx_yo_dt_b_asin_title_o08_s00?ie=UTF8&psc=1) | 1      |
| 9   | batteries  AAA                                                                                                 | 3      |



## 2. schematic

![schematic](/uploads/schematic.png)

# Power
Raspberry pi Pico can be powered in several ways because it avoids a long range input voltage (1.8v - 5.5v), for this case I chose three AAA common batteries but of course you can use other options like a 18650 battery.
In the [datasheet](https://datasheets.raspberrypi.com/pico/pico-datasheet.pdf) you can find more information about how to power it in section 4.5.

# Motor Driver
About the DRV8833 motor driver I increased the pwm frequency to reduce the noise produced by the motor at medium/low speeds.
IN1/IN2 inputs are used to control OUT1/OUT2 where the first two are connected to pins 14 and 15 of pi Pico.

It is possible also to control the motor direction, although in this case the reverse direction will not produce any wind ;).
The ULT pin has to be connected to Power to disable the sleep mode.

# Visual effects
The led brightness has been reduced to 0.04, it is comfortable to see with this value.


## 3. Code

```
from machine import ADC, Pin
import time, utime
import array, rp2
from rp2 import PIO, StateMachine, asm_pio

# Author: MCV. @lenambac
# 20220805 v1.0

#Motor config
pin = machine.Pin(15)
pwm = machine.PWM(pin)
pwm.freq(40000)
pin2 = machine.Pin(14)
pwm2 = machine.PWM(pin2)
pwm2.freq(40000)

# Potenciometer config
adc = ADC(Pin(26))
poten_dutie = 45000 # Var to register the dutie from potenciomenter

#Neopixels config (WS2812)
NUM_LEDS = 12
brightness = 0.04
PIN_NUM = 0 # Pin donde estan los neopixels

## Boton
button = Pin(13, Pin.IN, Pin.PULL_DOWN)
clickA=0   #First click time
clickB=0   #Second click time
elapsed=30000 #Interval to avoid multiple clicks between clickA & clickB
button_number=0 # State machine for botton



############################################
# Functions for Motor control
############################################

def forward(mydutie):
    print("fordward:",mydutie)
    pwm.duty_u16(mydutie)
    pwm2.duty_u16(0)
    
def backward(mydutie):
   print("backward", ydutie)
   pwm.duty_u16(0)
   pwm2.duty_u16(mydutie)
   
def stop():
   print("stop")
   pwm.duty_u16(0)
   pwm2.duty_u16(0)
   
def test():
   print("Test: poten_dutie:",poten_dutie)
   forward(poten_dutie)
   utime.sleep(3)
   stop()
   backward(poten_dutie)
   utime.sleep(3)
   stop()


def potenciometer():
    global poten_dutie
    poten_dutie=adc.read_u16()
    print("potenciomenter_dutie:",poten_dutie)
    
    #forward(dutie)

############################################
# Functions for WS2812
############################################

@asm_pio(sideset_init=PIO.OUT_LOW, out_shiftdir=PIO.SHIFT_LEFT,
autopull=True, pull_thresh=24)
def ws2812():
    T1 = 2
    T2 = 5
    T3 = 3
    label("bitloop")
    out(x, 1) .side(0) [T3 - 1]
    jmp(not_x, "do_zero") .side(1) [T1 - 1]
    jmp("bitloop") .side(1) [T2 - 1]
    label("do_zero")
    nop() .side(0) [T2 - 1]

# Create the StateMachine with the ws2812 program, outputting on Pin(0).
sm = StateMachine(0, ws2812, freq=8000000, sideset_base=Pin(PIN_NUM))
# Start the StateMachine, it will wait for data on its FIFO.
sm.active(1)

# Display a pattern on the LEDs via an array of LED RGB values.
pixel_array = array.array("I", [0 for _ in range(NUM_LEDS)])
#pixel_array = array.array("I", [0 for _ in range(ONE_LED)])


############################################
# Functions for RGB update
############################################


def updatePixel_mcv(brightness=1): # dimming colors and updating state machine (state_mach)
    dimmer_array = array.array("I", [0 for _ in range(ONE_LED)])
    for ii,cc in enumerate(pixel_array):
        r = int(((cc >> 8) & 0xFF) * brightness) 
        g = int(((cc >> 16) & 0xFF) * brightness) 
        b = int((cc & 0xFF) * brightness)
        print("Antes de dimmer ii:", ii ," cc:", cc)
        dimmer_array[ii] = (g<<16) + (r<<8) + b 
    sm.put(dimmer_array, 8) # update the state machine with new colors
    
    

def updatePixel(brightness=brightness): # dimming colors and updating state machine (state_mach)
    dimmer_array = array.array("I", [0 for _ in range(NUM_LEDS)])
    for ii,cc in enumerate(pixel_array):
        r = int(((cc >> 8) & 0xFF) * brightness) 
        g = int(((cc >> 16) & 0xFF) * brightness) 
        b = int((cc & 0xFF) * brightness) 
        dimmer_array[ii] = (g<<16) + (r<<8) + b 
    sm.put(dimmer_array, 8) # update the state machine with new colors
    
def set_led_color(color):
    for ii in range(len(pixel_array)):
        pixel_array[ii] = (color[1]<<16) + (color[0]<<8) + color[2]
        

def set_24bit(ii, color): # set colors to 24-bit format inside pixel_array
    pixel_array[ii] = (color[1]<<16) + (color[0]<<8) + color[2] # set 24-bit color


############################################
# Functions for RGB Coloring
############################################

def colour_loop():
    for x in range(0, 255, 50):
        for y in range(0, 255, 50):
            for z in range(0, 255, 50):
    #for x, y, z in zip(range(1,255), range(1,255), range(1,255)):
                my_color = (x, y, z)
                set_24bit(0,my_color)
                set_24bit(1,my_color)
                set_24bit(2,my_color)
                set_24bit(3,my_color)
                set_24bit(4,my_color)
                set_24bit(5,my_color)
                set_24bit(6,my_color)
                time.sleep(0.1) # wait 50ms
                updatePixel() # update pixel colors
                print("x:",x, "y", y, "z",z)


def kit_line(in_colour, times, delay):
    my_color = cyan
    for i in range(times):
        for i in range(NUM_LEDS):
            print("line: ",i);
            set_24bit(i,in_colour)
            updatePixel() # update pixel colors
            time.sleep(delay) # wait 50ms
            set_24bit(i,off)
            
        for i in range(NUM_LEDS):
            print("line2: ",NUM_LEDS-i-1);
            set_24bit(NUM_LEDS-i-1,in_colour)
            updatePixel() # update pixel colors
            time.sleep(delay) # wait 50ms
            set_24bit(NUM_LEDS-i-1,off)

def circle(in_colour, times, delay):
    my_color = cyan
    for i in range(times):
        for i in range(NUM_LEDS):
            print("line: ",i);
            set_24bit(i,in_colour)
            updatePixel() # update pixel colors
            set_24bit(i,off)
            time.sleep(delay) # wait 50ms

def circle_two(in_colour, times, delay):
    my_color = cyan
    for i in range(times):
        for i in range(NUM_LEDS):
            print("line: ",i);
            set_24bit(i,in_colour)
            
            if i > 1:
                set_24bit(i-2,off)       
            time.sleep(delay) # wait 50ms
            updatePixel() # update pixel colors
            
            
def race_two(in_colour, times, delay):
    my_color = cyan
    j=0
    for i in range(times):
        for i in range(NUM_LEDS):
            print("line1: ",i)
            set_24bit(i,in_colour)
            #Second led
            j=int(i+NUM_LEDS/2)
            if j >= NUM_LEDS: ## Vol dir que hem arribat al darrer LED, llavors a J li treiem NUM_LEDS
                j=j-NUM_LEDS
            print("line2:",j)
            set_24bit(j,in_colour)                
            
            if i > 1:
                set_24bit(i-1,off)
            if j > NUM_LEDS/2:
                set_24bit(j-1,off)
                
                
            time.sleep(delay) # wait 50ms
            updatePixel() # update pixel colors

def fill_in(in_colour, times, delay):
    my_color = cyan
    FULL = NUM_LEDS;
    for i in range(times):
        for i in range(FULL):
            print("line: ",i);
            set_24bit(i,in_colour)
            if i != FULL-1 :
                set_24bit(i-1,off)
            updatePixel() # update pixel colors
            time.sleep(delay) # wait 50ms
        FULL=FULL-1;            

############################################
# Color based on RGB (R,G,B)
############################################

red = (255,0,0)
green = (0,255,0)
blue = (0,0,255)
white = (255,255,255)
purple = (255,0,255)
pink =(255, 192, 203)
off = (0,0,0)
harlequin = (63, 255, 0)
darkred = (139, 0, 0)
cyan = (0, 255, 255)
ultramarine = (18, 10, 143)
turquesa = (10, 245, 142)
fucsia = (255,99,189)
strong_fucsia = (255,0,147)
lila = (28, 0, 59)


############################################
# Button control
############################################

def review_button():
    global clickA
    global clickB
    global button_number
    
    if button.value():
            clickA=time.ticks_us()
            if clickA - clickB > elapsed:
                #print ("A",clickA)
                #print ("B",clickB)
                print("Botton pressed", time.ticks_us())
                clickB=clickA
                button_number+=1
                #print(button_number)
                return button_number

def update_rgb(res):
    if res==1:
        circle(turquesa, 1, 0.05)
    if res==2:
        circle(cyan, 2, 0.04)
    if res==3:
        circle(green, 3, 0.03)
    if res==4:
        circle_two(ultramarine, 4, 0.02)
    if res==5:
        circle_two(darkred, 5, 0.01)
    if res==6:
        race_two(darkred, 2, 0.01)
    
        
    
while True:
    
    potenciometer() # Read potenciometer and update fan speed var: poten_dutie
    print("Loop:",poten_dutie)
    forward(poten_dutie)
    
    res=review_button()
    if button_number > 5:
        button_number=1
    update_rgb(res)
        
    
        
    
    
    #time.sleep(1)
    
    # Test Motor speeds
    a=1;
    if a ==0:
        for i in range(50):
          test()
```
