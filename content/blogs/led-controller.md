---
author:
  name: "greg"
date: 2019-07-25
linktitle: LED Controller
title: Pretty LEDs Over Bluetooth
type:
- blog
- blogs
weight: 10
series:
- Projects
aliases:
- /blogs/led-controller/
---

When I first got my hands on a strand of LEDs, I knew that I wanted to find a way to control them from my phone, and I think the result is awesome.
{{< gfycat deficientwhiteeelelephant >}}

<br/>
### How does it work?
This project consists of two main parts:

- The LED Controller server running on an [ESP32](https://en.wikipedia.org/wiki/ESP32)
- The android app that runs on my phone
<br/>

The ESP32 is cool because it is supported by Arduino, while also having a bluetooth and a wifi chip built in, which makes it particularly useful for IoT applications. I started the project by directly controlling the LEDs from the ESP32 until I found some cool animations that I wanted to stick with. Once I figured those out, I added the bluetooth server in. However, there was a major problem: 

The LEDs relied on the ```delay()``` function to do animations, which would cause bluetooth connections to fail all the time, since delay blocks the ```loop()``` function. The solution to this is to instead count how much time has passed since the animation last animated and to progress to the next step only if the time passed is greater than the set "delay time" of the animation. Here is how I do it:
```c++
void Animation::run() {
    if (!isSetup) {
        this->setup();
        isSetup = true;
    }

    long currentTime = millis();
    if (currentTime - lastTimeRan >= delayTime) {
        this->stepFrame();
        lastTimeRan = currentTime;
    }
}
```

With this change, the bluetooth server is able to run at the same time as the animations are, since the ```loop()``` function is no longer being blocked. The bluetooth server allows the client to change the animation, color, brightness, and delay time of the animation. Once I got that part working, it was time to begin working on the android app.


{{< figure src="/img/leds/led-controller-app.png" style="width: 200px;" position="center">}}

The hardest part of making the app was handling the connection flow for bluetooth. Since the ESP32 uses Bluetooth Low Energy (BLE), the process to connect and send data was a lot different from normal bluetooth connections on Android. Also, the documentation for using BLE on Android was super out of date, so it just took a lot of trial and error before I could get the app to reliably connect to the server and to reconnect properly when the app closed. The color picker was a cool HSV color picker library that I found, since the LED Controller uses HSV instead of RGB. 


With the app completed, it was time to assemble everything and to mount the LEDs on my wall. For fun, I 3D printed a little box to put the controller in.

{{< sameline >}}
  {{< figure src="/img/leds/led-controller-print.jpg" style="width: 200px;" position="center" caption="Printing the box">}}
  {{< figure src="/img/leds/led-controller-assembly.jpg" style="width: 200px;" position="center" caption="The assembled controller">}}
  {{< figure src="/img/leds/led-controller-final.jpg" style="width: 200px;" position="center" caption="Everything put together">}}
{{< /sameline >}}

### The Final Result
After several months of on-and-off work, I finally had my phone controlled LEDs. The app is able to change color & brightness, as well as these animations & their speed:

  - Meteor
  - Color Fade
  - Color Wipe
  - Solid Color

{{< gfycat dangerouspresentachillestang >}}


### Github Repositories
 - [Android App](https://github.com/greg-bahr/led-controller-app)
 - [LED Controller](https://github.com/greg-bahr/led-controller)