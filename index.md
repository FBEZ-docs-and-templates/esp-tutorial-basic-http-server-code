# HTTP Server Tutorial

## Introdution

In this tutorial you will create a simple http server as a mean for provisioning. The esp32* module will start as an access point (AP) and serve a simple http page which lets the user to connecto to it, browse to a specific page and enter the SSID and password of the router. 
The SSID and password are then read and showed on the terminal. In a real application, you would stop the AP and use these credentials to connect as a station (STA) to the user's router. 

### What you will learn

Through this activity, you will learn how to start the ESP32* as an AP and how to manage HTTP request. To do so, you will learn how to create an event loop and which http events are available on the esp-idf framework. You will create a GET response sending a basic HTML page and read the data from a POST request. 

## Outline

The tutorial will have the following parts:
1. Create new project (test hello world)
2. Start ESP32* as AP
   * AP vs STA
   * SSID and PASSWORD
   * Connect to ESP32* AP
3. Create an event loop
   * What is an event loop?
   * What events are available?
4. Routes
   * What is a route?
   * Let's create a GET route
   * Let's create a POST route
   * How to use data?
5. Simple provisioning
   * Putting all together


__Prerequisites__

* Vscode installed and working (you can compile and run hello_world on a evk)
* An Espressif EVK or board you can flash code on
* You know the difference between Access point and Station
* You know the basic of HTTP (what a GET and POST request are)
* You know the basic of HTML (a few simple tags)

## Create a new project

To start a new project and be ready for the next steps you need to perform the following:
1. Create a new project from `hello_world` template
2. Set the target core 
3. Set the right port
4. Build, flash and start a monitor with `hello_world`

To start, let's create a new project. Open vscode, pess `CTRL+SHIFT+p`and start typing `ESP-IDF: New Project`. Follow the instruction to create a new project using the `hello_world`template. 

Then you have you set the target and the port for the flashing stage. In this tutorial we're using an `ESP32-C3-DevKitM-1`. 
Start typing `ESP-IDF: Set Espressif Device Target` and select `esp32c3`. If you are using and evk or a USB-UART bridge, you can select `The ESP32-C3 Chip via builtin USB-JTAG`. If you're using on of the `ESP-PROG` programmer, you can chose the best option accordingly. 

Select the right port, typing `ESP-IDF: Select Port to Use (COM, tty, usbserial)`. 
At this point, you can run the command `ESP-IDF: Build, Flash and start a Monitor on your device`. 

If everything runs smoothly, you should see the compilation finish succesfully, the flashing of the firmware and the `hello_world` example running, saying `Hello world!` and restarting after 10 seconds. 

Now change the content of `hello_world_main.c` to the following code

```C
#include <stdio.h>
#include <inttypes.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include <string.h>
// #include "esp_wifi.h"


void app_main(void)
{
    printf("Hello tutorial!\n");

}
```

Build, flash and monitor again to be sure that everything is working. 

Before moving on, it's best to change the file name. To do so, rename the file `hello_world_main.c` to `basic_http_server.c`. Then we have to inform CMake that it needs to compile this file instead of `hello_world_main.c`. Open `main/CMakeLists.txt` and change the content to
```
idf_component_register(SRCS "basic_http_server.c"
                    PRIV_REQUIRES spi_flash
                    INCLUDE_DIRS "")
```

Perform a full clean (`ESP-IDF: Full Clean Project`) and build, flash and open a monitor again. 


## ESP32* as AP


### TO-CHECK - NVS

First we inizialize the nvs flash. To do so we need to include "nvs_flash.h". To tell the compiler where to find it, we should add it as PRIV_REQUIRES in the CMakeLists.txt:
```
PRIV_REQUIRES nvs_flash
```

Remember to perform a full-clean every time you change the CMakeLists.txt file. 

### LOGS

To make it easier to test all the parts of the code, it's better to make use of the logging utilities given by the `esp_log.h` header. To do so, you need to include it at the beginning of the file. 
You can create a tag string, used for logging purposes with `static const char *TAG = "Basic HTTP Server";` and call the logginf functions as `ESP_LOGI(TAG, "ESP Tutorial");`.
You can find an in-depth explaination of all the loggin function [here](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/log.html).
In this tutorial only `ESP_LOGI` will be used. 

