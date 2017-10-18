---
layout: post
title: Controlling an Adapter Plug with an ESP 8266-01 Wi-Fi Chip and MQTT&#58; A Prototype
modified:
categories: [Programming, Electronics]
description:
tags: [IoT, Electronics, ESP8266]
image:
  feature: 819s.jpg
  credit: Designed by xb100 / Freepik
  creditlink: http://www.freepik.com
comments:
share:
date: 2017-10-12T19:55:53+02:00
---

In this post we will take a look at the steps required to build a simple adapter-plug which can be turned on and off via the internet (or a local network) using a ESP8266 module and the MQTT protocol. The resulting adapter-plug will look like this:
<figure class="half">
	<img src="/images/2017-10-12-control-an-adapter-plug-with-an-esp8266-and-mqtt/electronics6.jpg" alt="">
	<img src="/images/2017-10-12-control-an-adapter-plug-with-an-esp8266-and-mqtt/outlet_side.jpg" alt="">
</figure>
<!--more-->

Recently, the low-cost ESP8266 Wi-Fi chip family is becoming increasingly popular among hobbyists.
The ESP8266 has microcontroller unit (MCU) onboard and it has a full TCP/IP stack which makes it a very interesting device for many IoT (Internet of Things) tasks. The chips are produced by the Chinese manufacturer, Espressif Systems, which is located in Shanghai. For this post I am using a simple module, the ESP8266-01, which basically only provides two general purpose input/output (GPIO) pins, but which is rather small and sufficient for the problem presented here. The figure below shows the top view of the ESP-01.
<figure class="half">
	<img src="/images/2017-10-12-control-an-adapter-plug-with-an-esp8266-and-mqtt/esp01v0.jpg" alt="">
	<img src="/images/2017-10-12-control-an-adapter-plug-with-an-esp8266-and-mqtt/esp8266-01.jpg" alt="">
</figure>
*Top view of the ESP8266-01 module. Images taken from [Sparkfun](https://www.sparkfun.com/products/13678). Images are CC by [2.0](https://creativecommons.org/licenses/by/2.0/)*
<!--more-->

## Wiring of the ESP8266-01

The first and most important thing to note is that ESP8266 chip only uses 3.3V for power and I/O. Do not attempt to connect the ESP directly to 5V, since at best this may simply not work and -- more likely -- this will cause permanent damage to the chip. You have to be especially careful if you are using FTDI USB-to-TTL adapter cables, since they typically provide a VCC of 5V. If you use your own power supply, ensure that it provides more than 200mA (this requirement certainly also holds for your adapter cables and USB-ports of your computer). If not, you might observe very strange behavior of the ESP (such as random restarts).

If you are working with FTDI USB to TTL Adapter Cables that include PL-2303HX Chipset, there might be several issues with installing the driver of the adapter for Windows 7,8 and 10. For me, the description from [http://www.ifamilysoftware.com/news37.html](http://www.ifamilysoftware.com/news37.html) solved the problems.  

Typically, the ESP8266-01 is wired as illustrated in the following diagram:

![Wiring Diagram ESP 8266-01]({{ site.url }}/images/2017-10-12-control-an-adapter-plug-with-an-esp8266-and-mqtt/esp8266_flash_prog_board_sch.png)
*Image taken from [http://www.esp8266.com/wiki/doku.php?id=getting-started-with-the-esp8266](http://www.esp8266.com/wiki/doku.php?id=getting-started-with-the-esp8266)*

As we can see, the ESP-01 version of this module has 8 pins: VCC, GND, CH_PD,TX, RX, RST, GPIO0, and GPIO1.
The following wiring is required to make everything work.
- VCC is connected to 3.3V
- CH_PD has to be pulled-up (meaning it has to be connected to 3.3V as well; typically with a 1k OHM resistor). Do not forget this step, otherwise your ESP will not function.
- GND is connected to FTDI’s GND pin
- RX and TX are required for the serial communication with your computer. Note, that the wires are crossed when they are connected to the FTDI chip, hence:
  - RX is connected to FTDI’s TX pin
  - TX is connected to FTDI’s RX pin
- The RST pin is pulled up to 3.3V (with a 1k Ohm resistor) and will be reset if it is pulled down to ground with a switch (RESET switch in figure below). Also here do not forget the pull-up resistor, otherwise your ESP resets consistently.
- GPIO-0 should be connected to ground via a switch (PROG in above diagram). This is required to be able to flash the modul. During normal operation it can be used for other purposes.
- GPIO-1 stands at your free disposal

![Wiring of the ESP8266-01 on a breadboard]({{ site.url }}/images/2017-10-12-control-an-adapter-plug-with-an-esp8266-and-mqtt/20160503_172349.jpg){: .image-center width="400px"}
<center><it>Wiring of the ESP8266-01 on a breadboard.</it></center>

## First Conversation with the ESP

In order to check if your wiring with a new ESP8266-01 works, you can use a standard terminal emulator to talk to the module, since it already comes with a flashed firmware. Note that once you flashed the ESP-01 modul yourself, it will not respond to AT commands (such as AT+GMR, AT+RST, etc.) any longer, since the initial firmware will be overwritten. Since the baud rate configuration is not consistent on the ESP-01 modul, you will most likely have to try different settings. The ones you want to try first are 9600 and 115200. Then 57600 and/or 76800 (38400 * 2). Try different baud rates, for some you might only see a sequence of unrecognizable symbols, for some nothing at all.
After resetting your ESP modul there should be a ”ready“ message at the selected baud rate if your UART Rx is wired correctly.
Once you find a working baud rate, you can send a first `AT` command to the module. The following AT command asks the module whether it is up. It should respond with OK.

{% highlight plaintext %}
  AT

  OK
{% endhighlight %}

Additionally, you can try the `AT+GMR` command, which should produce a result similar to this:
{% highlight plaintext %}
  AT+GMR

  AT version:0.40.0.0(Aug  8 2015 14:45:58)
  SDK version:1.3.0
  Ai-Thinker Technology Co.,Ltd.
  Build:1.3.0.2 Sep 11 2015 11:48:04
  OK
{% endhighlight %}

The AT command set is quite large, a table containing the commands (e.g. in order to set up the WiFi) can be found [here](https://nurdspace.nl/ESP8266#AT_Commands). You can already do quite a lot with this basic firmware installed on the ESP.

## A simple Test Program using the Arduino IDE

With a few steps it is possible to use the Arduino IDE for programming the ESP8266 moduls. Check out [this website](http://www.whatimade.today/esp8266-easiest-way-to-program-so-far/) for an installation guide. There are also many other guides for setting up the Arduino IDE for usage with the 8266 family on the web.
Once you set-up your Arduino environment, you might want to try this first simple test program that blinks a LED.

The wiring for this setup is rather easy and shown in the following diagram.

![TODO]({{ site.url }}/images/2017-10-12-control-an-adapter-plug-with-an-esp8266-and-mqtt/lochmaster.png){: .image-center}<br>
*Connecting a LED to the GPIO-2 pin of the ESP8266 modul.
Note that the connection diagram shown here is simplified and does
not include the additional connections that are typically required
(TxD, RxD) to flash a program to the device. Figure taken from [iot-playground.com](http://iot-playground.com/blog/2-uncategorised/38-esp8266-and-arduino-ide-blink-example)*

In order to see the result, connect a LED via the GPIO-2 pin and a resistor (typically between 220-330 Ohm; but do your calculations to be sure) to ground (as shown in above figure). After you have done the wiring, you can flash the ESP8266-01 with the following simple program, which should make the LED blink in short intervals.

{% highlight C %}
int ledPin = 2;
void setup() {
  pinMode(ledPin, OUTPUT);
}

void loop() {
  digitalWrite(ledPin, HIGH);   // turn the LED on (HIGH is the voltage level)
  delay(500);                   // wait for 500ms
  digitalWrite(ledPin, LOW);    // turn the LED off by making the voltage LOW
  delay(1000);                  // wait for a second
}
{% endhighlight %}

<!--![TODO]({{ site.url }}/images/2017-10-12-control-an-adapter-plug-with-an-esp8266-and-mqtt/20160503_173908.jpg)-->

## Circuit for Controlling the Adapter Plug

Now, after you are sure that your ESP is working as desired and you are comfortable in programming it, let us take a look at the circuit required for our adapter plug. If you are going to work with high voltages, extreme caution is required. Make sure that contacts with high voltages are not exposed. If you are not confident enough to work with the mains voltage, you can still make the circuit work and use it to switch lower voltages. First, let us look at the layout of the stripboard:

![Layout of the stripboard]({{ site.url }}/images/2017-10-12-control-an-adapter-plug-with-an-esp8266-and-mqtt/lochmaster2.png)


- The circuit has to be provided with a supply voltage of 5V (Top left corner). Make sure that the power supply can provide currents of at least 500mA. I found a power supply which is quite small and fits into the package of the adapter plug
- Since the ESP-01 modul requries 3.3V, we need a regulator (LM1117T) that transforms the supplied 5V voltage.
- The ESP-01 modul is plugged into the socket shown in the upper-middle part of the diagram
- The button S1 (RST) is used to reset the ESP-01.
- The button S2 has two functions: Firstly, it is used to put the ESP into program-mode (press S1 - press S2 - release S2 - release S1). During normal operation the button is used to turn on/off the outlet manually.
- he transistor T2 (e.g., D1163A or comparable) is used to drive the relay. Basically any NPN transistor can be used. However, make sure that it can provide currents sufficient for the relay. Furthermore, notice, that a diode D2 is - required in parallel to the relay to prevent circuit damages due to the inductive component.
- The LED D1 will indicate the current state of the adapter plug (on or off)
- We noticed that the ESP cannot initialize after being turned on, when the base of T2 is directly connected to the GPIO2 pin. The reason for this could be that the GPIO2 pin is set to HIGH during initialization which would directly activate the relais via T2. But somehow the ESP modul cannot tolerate any electrical loads during the initialization phase. For this reason we introduced a second transistor T1 (BC547) with a capacitor (10uF) that delays HIGH-signals on GPIO2 for about a second. When the circuit is turned on, the ESP modul will initialize and during this time set its GPIO2 pin to HIGH. With the current that flows through R5 (470 kilo-Ohm) the capacitor (10uF) will gradually charge. In the beginning the potential on the base of T1 will therefor be 0V and the Collector-Emitter current flow through the transistor will be blocked. If the voltage in the capacitor reaches a sufficient voltage, T1 will activate and T2 can then turn on the relay.
- The green pin header in the figure is connected to a FTDI device in order to program the ESP-01 via the serial interface.
- Note that the ESP-01 operates in voltage ranges of 0V – 3.3V. Hence, you have to make sure that your FTDI converter operates in the same range as well; otherwise you will destroy the ESP-01 modul.


The following images show how the circuit board and the final adapter plug could look like:
<iframe class="slideshow-iframe" src="{{ site.url}}/slides/controlAdapterPlug.html" style="width:100%" frameborder="0" scrolling="no" onload="resizeIframe(this)"></iframe>

## Source code
So now we just need some program, which can be used to control the adapter plug via MQTT.

{% highlight C++ %}
 #include <ESP8266WiFi.h>
 #include <PubSubClient.h>
 #include <string.h>
 #define SLEEP_DELAY_IN_SECONDS  10
 #define TOPIC_ROOT "gf3heTS11c"
 #define DEVICE_ID  "plug-01"
 #define SLASH "/"
 #define CMD_RELAIS_ON "device=on"
 #define CMD_RELAIS_OFF "device=off"


const char* ssid = "MySSID";
const char* password = "mypass";
const char* mqtt_server = "iot.eclipse.org";
const char* mqtt_username = "MeMyselfandI";
const char* mqtt_password = "none";
const char* mqtt_pubs_topic = "myTopic/test";
const char* mqtt_subs_topic = TOPIC_ROOT SLASH DEVICE_ID;
WiFiClient espClient;
PubSubClient client(espClient);

// GPIO-pin 2, used for activating the relais
int GPIO2 = 2;
// GPIO-pin 0, used as input for manually turning on/off the relais
int GPIO0 = 0;
// State of the relais
int state = LOW;
// Flag used for debouncing the push-button
bool debounce = false;
/*
 * Called once during the initialization phase after reset
 ****/
void setup() {
  // setup serial port
  Serial.begin(115200);
  // setup WiFi
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  // setup pins
  // GPIO2 will be used to activate the relais
  // GPIO0 will be an input that is used to turn on the relais using a button
  pinMode(GPIO2, OUTPUT);
  pinMode(GPIO0, INPUT);
  attachInterrupt(GPIO0, fallingEdge, FALLING);
}
void fallingEdge() {
  if(!debounce) {
    state = !state;
    digitalWrite(GPIO2, state);
    debounce = true;
  }
  // Do not call other functions in this interrupt (such as delay, etc.). This only causes problems.
}

void setup_wifi() {
  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}
 /*
  * MQTT callback function. It is always called with we receive a message on the subscribed topics.
  * The topic itself is given as a parameter of the function.
  ****/
void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  int i;
  char string[50];
  char lowerString[50];
  // Copy the payload into a C-String and convert all letters into lower-case
  for (i = 0; i < length; i++) {
    string[i] = (char)payload[i];
    lowerString[i] = tolower(string[i]);
  }
  string[i]=0;
  lowerString[i]=0;
  Serial.print(lowerString);

  // If the paylaod contains one of the predefined messages then turn on/off the relais
  if(strcmp(lowerString, CMD_RELAIS_ON) == 0)
    digitalWrite(GPIO2, HIGH);
  else if(strcmp(lowerString, CMD_RELAIS_OFF) == 0)
    digitalWrite(GPIO2, LOW);

  Serial.println();
}
/*
 * Can be used to reconnect to the MQTT-Broker, in case we loose the connection inbetween
 ****/
void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Attempt to connect
    if (client.connect(DEVICE_ID)) {
      Serial.println("connected");
      // Subscribe to a topic
      if(client.subscribe(mqtt_subs_topic))
        Serial.println("Subscribed to the specified topic");
      else
        Serial.println("Failed to subscribe to the specified topic");

    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" trying again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}
/*
 * This function will be called after setup() in an infinite loop
 ****/
void loop() {
  if (!client.connected()) {
    reconnect();
  }

  Serial.println("I am still alive...") ;
  // send message to the MQTT topic
  client.publish(mqtt_pubs_topic, "I am still alive...");

  // Example of a retained message
  //client.publish(mqtt_topic, "This will be retained", true);
  client.loop();
  delay(2000);
  // After two seconds we can be sure that the button is debounced. The interrupt can also
  // come during the delay, but it unlikely to happen exactly (few ns) before the delay is over.
  debounce = false;
}
{% endhighlight %}

## Usage

Once the ESP-01 is flashed with the above program, there are basically two possibilities to turn on and off the electrical outlet:
1. You may use the button S2 to toggle the outlet. With every impulse generated by the button the state of the relay is inverted.
2. MQTT: You can use a MQTT client to publish messages to a specified topic and control the outlet with these messages. In this example (see definitions in the above code), the following configuration is used:

{% highlight plaintext %}
MQTT-Broker:        iot.eclipse.org
Port:               1883
Topic:              gf3heTS11c/plug-01
Accepted Messages:  DEVICE=ON
                    DEVICE=OFF
{% endhighlight %}

## Future work

Although this prototype is already working quite well, there is still some work that can be done:
- Request current state of the outlet via MQTT
- Instead of a standard relay one could use bi-stable relay. This would save some energy and would take load off the power supply
- Addressing and communication scheme for more complex scenarios
- Smart-phone App for managing the outlets and other MQTT-capable devices
- Integration into larger smart-home solutions
