---
title: Raspberry pi Pico and neopixels (WS2812)
subtitle: How to play with neopixels and raspberry pi pico
category:
  - PI
author: lenambac
date: 2021-08-26T04:27:56.800Z
featureImage: /uploads/pico_neopixel.png
---

[Raspberry PI Pico](https://www.raspberrypi.com/documentation/microcontrollers/raspberry-pi-pico.html) is one of the last devices of Raspberry  PI Company. It is a microcontroller (MCU) which includes the new [RPI2040](https://www.raspberrypi.com/documentation/microcontrollers/rp2040.html#welcome-to-rp2040)] chip designed by the company, it brings high performance, low cost and ease of use to the microcontroller space.

![ws2812](/uploads/pipico.png)


On the other hand [neopixels](https://learn.adafruit.com/getting-started-with-raspberry-pi-pico-circuitpython/neopixel-leds) can be used to make rings, strips, boards, etc ... with miniature LEDs. They are chainable from one to the next so you can power and program a long line of Neopixels together.
The WS2812 Integrated Light Source (neopixel) has integrated red, green and blue LEDs alonside a driver chip into a tiny surface-mount package controlled through a single wire.
Tyey can be used individually, chained into longer strings or assemnlbed into still more interesting form-factors.

![ws2812](/uploads/ws2812.jpg)

We are going to use some WS2812 leds with Raspberry PI Pico, through [mycropython](https://micropython.org/) code it is possible to create amazing light effects. Let's go!.


Components list:

| \#  | Description                                          | Quantity | 
| --- | ---------------------------------------------------- | -------- | 
| 1   | Neopixels - WS2812B                                  |    12    | 
| 2   | Raspberry pi pico                                    |     1    | 


## 1. Install Thonny
The software interface to code it is Thonny, [download](https://projects.raspberrypi.org/en/projects/getting-started-with-the-pico/2) and install it in your computer.

Follow all the steps indicated in the getting started guide like microPython firmware, etc ...

Once you have tested your device with Blink example, then go on with the next steps.

## 2. Connect NeoPixels to Raspberry PI PICO

The Pico board can control a considerable number of NeoPixels LEDs, but if you want to use a lot of them then you need an external power supply.

Also, it is possible to use a 18650 Lithium battery to power Pico Board and neopixels, but in this case use WS2812B which can operate at anywhere about 3.3V to 5V.

Make the following connections:
- Board GND to NeoPixel GNS
- Board VBUS to NeoPixe 5V (Power)
- Board GP0 to NexoPixel signal

![pico_neopixels](/uploads/circuitpython_Cat_NeoPixels_bb.jpg)



## 3. Application code

In the following example you can run a nice visual effect, extend and improve with your ideas to do amazing effects. 

Variables:
- NUM_LEDS: Specify the number of neopixels attached
- BRIGHTNESS: Specify the brightness, recommended values [0.04-0.5]
- PIN_NUM: Pin 0 (GP0)


![Thonny](/uploads/thonny.png)


```
import array, time, utime
from machine import Pin, ADC
import rp2
from rp2 import PIO, StateMachine, asm_pio

# Configure the number of WS2812 LEDs.
NUM_LEDS = 12
BRIGHTNESS = 0.04
PIN_NUM = 0 # Pin where are attached the neopixels GP0

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


#Color based on RGB (R,G,B)
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
# Functions for RGB Coloring
############################################

def set_24bit(ii, color): # set colors to 24-bit format inside pixel_array
    pixel_array[ii] = (color[1]<<16) + (color[0]<<8) + color[2] # set 24-bit color
    

def updatePixel(brightness=BRIGHTNESS): # dimming colors and updating state machine (state_mach)
    dimmer_array = array.array("I", [0 for _ in range(NUM_LEDS)])
    for ii,cc in enumerate(pixel_array):
        r = int(((cc >> 8) & 0xFF) * brightness) 
        g = int(((cc >> 16) & 0xFF) * brightness) 
        b = int((cc & 0xFF) * brightness) 
        dimmer_array[ii] = (g<<16) + (r<<8) + b 
    sm.put(dimmer_array, 8) # update the state machine with new colors

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
    
while True:
    
    kit_line(green, 2, 0.07)
    kit_line(ultramarine, 2, 0.07)
```

## Demo

![demo_neo](/uploads/demo_neo.jpg)


## Nex steps

Some next steps that can be implemented:
- Use neopixel and FastLED libraries
- Use buttons, potenciometres or other devices to modify the visual effects


## References:

- https://www.luisllamas.es/arduino-led-rgb-ws2812b/
- https://makersportal.com/blog/ws2812-ring-light-with-raspberry-pi-pico
- https://learn.adafruit.com/getting-started-with-raspberry-pi-pico-circuitpython/neopixel-leds
- https://dronebotworkshop.com/pi-pico-circuitpython/
- https://html-color.codes/navy
- https://htmlcolorcodes.com/es/
- https://wokwi.com/



