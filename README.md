# Interface Board

This handles the joystick inputs, reads current battery level, and manages safe-shutdown (by button or when the power is too low.) It's a programmable (in arduino IDE) ATMEGA328P chip, so you can modify how it works, if you want. On the pi-side a driver turns interrupt + i2c-read into joystick events. It also has kernel config for shutdown + i2c. It's all based on [this](https://github.com/piloChambert/RPI-I2C-Joystick).

We will probably make a few PCBs for different form-factors.

## notes

### pi-side

You can troubleshoot i2c stuff with [this info](https://www.abelectronics.co.uk/kb/article/1092/i2c-part-3---i-c-tools-in-linux).

**/boot/config.txt**

```
dtoverlay=gpio-poweroff,gpiopin=4,active_low="y"
dtparam=i2c_arm=on
```

Hook these up to the board:

- `SDA` BCM GPIO 2 (physical pin 3)
- `SCL` BCM GPIO 3 (physical pin 5)
```

Will also need to setup driver to connect to i2c and translate it into yystick.


### charging

Read more [here](https://forum.arduino.cc/t/battery-level-check-using-arduino/424054/5)

LiPo at 3.7V roughly reads these voltages (read from arduino A pin):

```
4.2 V --- 100%
4.1 V --- 090%
4.0 V --- 080%
3.9 V --- 060%
3.8 V --- 040%
3.7 V --- 020%
3.6 V --- 000%
```

Arduino code to read/scale value:

```cpp
int value = 0;
float voltage;
float perc;

void loop(){
  value = analogRead(A0);
  voltage = value * 5.0/1023;
  perc = map(voltage, 3.6, 4.2, 0, 100);
  // do something with perc here
  
  delay(500);
}
```

### arduino i2c

Read more [here](https://www.arduino.cc/en/Tutorial/LibraryExamples/MasterWriter). We are going to make arduino speak it's own i2c for different inputs, on request, and also fire an input-interupt pin, so we can respond on pi async.

Here is [info about interrupts on pi](https://roboticsbackend.com/raspberry-pi-gpio-interrupts-tutorial/)

