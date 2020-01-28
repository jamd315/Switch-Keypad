# Using a keypad to control the Switch
This is what started the project, the idea was to take a spare keypad and some shift registers I had and use them to control the Switch.

## Pictures and testing

<details>
<summary>Click to see pictures</summary>

Front ![Front](https://i.imgur.com/uP6rby3.jpg)
Back ![Back](https://i.imgur.com/sDnNzD8.jpg)
Buttons being tested ![Buttons](https://i.imgur.com/hmooWQ5.jpg)
Beat a low level CPU in Smash ![Smash](https://i.imgur.com/EmFfic5.jpg)

</details>

Testing in Smash, I found that while I can press the A button to confirm actions such as selecting characters and starting the game, I wasn't able to use my neutral attack, while standing still and while moving in a direction.  I haven't been able to fix this yet.  Special attacks work as expected.

## The keypad
The keypad is a 12-key numeric common terminal keypad.  Because it's a common terminal keypad, and not a matrix style keypad, it's a little better suited to multiple buttons being simultaniously pressed.  The buttons on the keypad have a resistance of around 300-5000 ohms, depending on how hard the button is being pressed.  I ran 5V thorugh the common pin so that the inputs on the shift register would be pulled high when a button is pressed.  You could also use a common ground pretty easily, especially with the complimentary serial output pin on the SN74HC165.  I've included a picture of my notes below for the pins and functions of each button.

![Keypad](https://i.imgur.com/GPlO2mx.jpg)

## The shift registers
Nothing too special about the shift registers, the one's I used are the [SN74HC165](http://www.ti.com/lit/ds/symlink/sn74hc165.pdf)s I got off eBay.  I tied them low with a 10k resistor on each input.  If you're OK with less inputs, you could remove the shift registers and resistors by using GPIO pins instead, you would just use a common ground and the internal pull-up resistors.  Because I was using an Arduino Pro Micro clone, I only had 10 GPIO pins, but would have needed 12.  I drove the shift registers using SPI.

## The Microcontroller
I used a $2 clone of the Arduino Pro Micro, which is based on the ATMEGA32U4.  Because I was using SPI, I needed to use pins 14 and 15 (MISO and SCK respectively) to drive the shift registers, and then any of remaining pins could have been used for the shift/load pin, but I used 9.  Pin 8 was used with a status LED for debugging.

## The code
Originally I started working with the actual SPI registers in the 32U4 datasheet, but determined that wasn't necessary.  I used the [LUFA SPI library](http://fourwalledcubicle.com/files/LUFA/Doc/151115/html/group___group___s_p_i___a_v_r8.html) instead, because the original repo uses LUFA for USB.  The SPI connection runs at 1MHz, I had some problems running at higher frequencies (F_CPU/2 and F_CPU/4) using both USART and SPI while debugging, and 1MHz is still a very good access time for 16 bits.  I removed the original debounce code from progmem's version, because I'm not using the PORTs on the MCU.  The other part of the code is just a series of if statements to write the keypad state to the joystick.

![Translation](https://i.imgur.com/UrlW8pS.png)

## Compiling and programming

I used the Windows Subsystem for Linux (wsl) for a lot of this, of course native Linux would also work.  To use wsl, either prefix the commands with wsl (e.g. run "wsl make" to compile), or run wsl once to use the shell.  I used BitBurner to upload, which is a Windows avrdude GUI.  I added a pushbutton to short reset and ground on the microcontroller.

The first thing you'll need is LUFA.  Download LUFA the ZIP from here http://www.fourwalledcubicle.com/LUFA.php, and extract the LUFA folder to the root of the project directory.  Alternatively you can specify another location by changing the LUFA_PATH variable in the makefile.  Next you'll need the libraries binutils-avr and avr-libc to compile, and avrdude is used for uploading the code, but if you use BitBurner you can skip it.

Prep the environment:
    
    sudo apt update && sudo apt upgrade -y
    sudo apt install make binutils-avr avr-libc avrdude -y

Compiling is pretty easy:

    make

To upload, first reset the MCU by shorting the RST and GND pins, then either run this command or press write in BitBurner within 8 seconds of the reset.

    avrdude -s -c avr109 -p m32u4 -P /dev/ttyS7 -U "flash:w:./Joystick.hex"

/dev/ttyS7 coresponds to COM7 in Windows.  The Serial port used to communicate and the serial port used to upload are different, you can see what port is used to upload by resetting the MCU.

#### Bitburner settings

![BitBurner](https://i.imgur.com/S9IArF0.png)
![BitBurner](https://i.imgur.com/tIdvWYH.png)