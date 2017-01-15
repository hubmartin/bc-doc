# Tweeting Doorbell Project

<!-- toc -->

## What is it?

This simple example will help you to know BigClown better.
The goal of this project is that a pressed button on the Core Module tweets to the Twitter account.
You can monitor your visits and see if you missed someone!

We will use Node-RED to react on MQTT message, then we send the tweet.

## What will we need?

* 1x [Core Module](https://shop.bigclown.com/products/core-module)
* 1x [Raspberry Pi 3](https://shop.bigclown.com/products/raspberry-pi-3-set)
* 1x Micro SD card ([download image](http://doc.bigclown.com/tutorial/install-rpi.html) or [buy the preloaded card](https://shop.bigclown.com/products/apacer-industrial-microsdhc-card-4gb))
* 1x [Micro USB cable](https://shop.bigclown.com/products/usb2-0-cable-am-b-micro-0-6m)

## Core Module

You need to add the code below to your `app/application.c` file.
Because all the logic is handled in the button callback, there's no need to use or create `application_taks()` function.

```c
// LED instance
bc_led_t led;
// Button instance
bc_button_t button;

// Initialization of application
void application_init(void)
{
    // Init USB
    usb_talk_init();

    // Init button and set event handler
    bc_button_init(&button, BC_GPIO_BUTTON, BC_GPIO_PULL_DOWN, false);
    bc_button_set_event_handler(&button, button_event_handler);

    // Init LED, start blinking after power-up
    bc_led_init(&led, BC_GPIO_LED, false, false);
    bc_led_set_mode(&led, BC_LED_MODE_BLINK);
}

void button_event_handler(bc_button_t *self, bc_button_event_t event)
{
    // Hide GCC warning of unused variable
    (void) self;

    // When the button is pressed
    if (event == BC_BUTTON_EVENT_PRESS)
    {
        static uint16_t event_count = 0;

        // LED example. Every second press the LED goes on
        bc_led_set_mode(&led, (event_count & 1) != 0 ? BC_LED_MODE_ON : BC_LED_MODE_OFF);

        // Publish USB talk message, this will create MQTT message in computer
        usb_talk_publish_push_button("", &event_count);

        // Count how many times the button is pressed
        event_count++;
    }
}

```

## Raspberry Pi configuration

To get the Raspberry Pi working please [follow this Raspberry Pi installation tutorial](http://doc.bigclown.com/tutorial/install-rpi.html). Connect to Rpi shell directly or by SSH.
You need the BigClown Raspberry Pi image because it contains the service to talk to Core module and MQTT.

Then we will need Node-RED to connect MQTT message from the button to the Twitter message. Please follow this [tutorial how to install and configure Node RED](http://doc.bigclown.com/tutorial/node-red.html)
