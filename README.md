## RESET PHOTON
[reset photon](https://docs.particle.io/tutorials/device-os/led/core/#safe-mode)

# Project2

[Click Here](https://particle.hackster.io/yatinagarwal/light-detecting-email-sender-db18f4)

## LED - Show Message or Local Weather 

Required Parts:
<br>- Particle Photon
<br>- LED Module [link](https://www.amazon.com/gp/product/B01EJ1AFW8/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1)
<br>- OPTIONAL: PowerShield (Battery) [link](https://www.amazon.com/gp/product/B06XJ64G8G/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1)
<!---
[link](https://docs.particle.io/tutorials/hardware-projects/maker-kit/#tutorial-3-conference-room-monitor)
--->

### Step 1: Set up your Photon
- Using Your Computer: Go to particle.io/setup and follow the instructions to create an account and set up your Photon.
- Using Your Phone & Download the Particle Mobile App [iPhone](https://itunes.apple.com/us/app/particle-build-photon-electron/id991459054?ls=1&mt=8) & [Android](https://play.google.com/store/apps/details?id=io.particle.android.app) to create an account and set up your Photon.

<img src="1.jpg" width="500">

<hr>

### Step 2: Connect Photo Sensor & Resistor to Photon
- Connect your sensor to a digital or analog input on the Photon. Follow these examples for hooking up common sensors.
```
Sensor Pin  | Photon Pin
       pin1 | A0 
       pin2 | A5
       OUT  | D0 
       
Resistor Pin  | Photon Pin
         pin1 | A0 
         pin2 | GRND
```
#### Set up the hardware

Should look like this

<img src="2.jpg" width="500">
<br>
<hr>

### Step 3: Setup BLYNK APP

- Download iOS or Android App and setup account on BLYNK (link)[https://blynk.io/en/getting-started]
- Create New Project Named: LED
- Add Terminal at Top
- Add 2 Buttons Below Terminal
  Should look like below:
  

Program each object
- Tap on Terminal 
  * Change Label to: Message
  * Change input to V1
  * Tap OK
- Tap of Button1 
  * Change label to: Temp/Text
  * Change output ot V1, 0 in first box and 1 in second box to right of V2
  * Mode Switch
  * On/Off Labels
    Left: Temp
    Right: Text
- Tap of Button2
  * Change label to: Power
  * Change output ot V3, 4 in first box and 3 in second box to right of V3
  * Mode Switch
  * On/Off Labels
    Left: Turn On
    Right: Text Off
  

       

### Step 4: Create a WebHook

- Go to https://console.particle.io/integrations 
- Click on New Integration
- Change "Request Format" JSON
- Click WebHook > CUSTOM TEMPLATE
- Paste below text and replace user and token with above user key and API Token
```
{
    "event": "exceedlight",
    "url": "https://api.pushover.net/1/messages.json",
    "requestType": "POST",
    "noDefaults": false,
    "rejectUnauthorized": false,
    "json": true,
    "query": {
        "user": "YOUR_USER_KEY",
        "token": "YOUR_API_TOKEN_KEY",
        "title": "Exceed Light",
        "message": "{{SPARK_EVENT_VALUE}}"
    }
}
```

- Click Create WebHook

<hr>

### Step 5: Create Particle App

- Go to https://build.particle.io/build/new 
- Title: LightDetect
- Paste Below Code
```
int led = D7; // This is where your can use can plug in a different LED. The other side should go to a resistor connected to GND.

int photoresistor = A0; // This is where your photoresistor is plugged in. The other side goes to the "power" pin (refer to diagram).

int power = A5; // This is the other end of your photoresistor. The other side is plugged into the "photoresistor" pin (above).
// The reason we have plugged one side into an analog pin instead of to "power" is because we want a very steady voltage to be sent to the photoresistor.
// That way, when we read the value from the other side of the photoresistor, we can accurately calculate a voltage drop.

int analogvalue; // Here we are declaring the integer variable analogvalue, which we will use later to store the value of the photoresistor.

void setup() {

    // First, declare all of our pins. This lets our device know which ones will be used for outputting voltage, and which ones will read incoming voltage.
    pinMode(led,OUTPUT); // Our LED pin is output (lighting up the LED)
    pinMode(photoresistor,INPUT);  // Our photoresistor pin is input (reading the photoresistor)
    pinMode(power,OUTPUT); // The pin powering the photoresistor is output (sending out consistent power)

    // Next, write one pin of the photoresistor to be the maximum possible, so that we can use this for power.
    digitalWrite(power,HIGH);

    // We are going to declare a Particle.variable() here so that we can access the value of the photoresistor from the cloud.
    Particle.variable("analogvalue", &analogvalue, INT);
    // This is saying that when we ask the cloud for "analogvalue", this will reference the variable analogvalue in this app, which is an integer variable.

}

void loop() {

    // check to see what the value of the photoresistor is and store it in the int variable analogvalue
    analogvalue = analogRead(photoresistor);


    if(analogvalue>=5){ //play with this number
      delay(100);
      char buf[200]; // need a buffer for that
      sprintf(buf,"%d",analogvalue);
      const char* p = buf;
      Particle.publish("light",buf,60,PRIVATE);
        digitalWrite(led, HIGH);
        Spark.publish("Light_limit","Exceeded",60,PRIVATE);
        Particle.publish("exceedlight", "Light Value is Exceeded", 60, PRIVATE);
        delay(5000); //delay for LED
        digitalWrite(led, LOW);
        delay(10000); //10 second gap for each observation
        // Add a delay to prevent getting tons of emails from IFTTT
    }
}


```
- Click Save
- Click Flash

That’s It! You should now see notifications when light is exceeded

If all goes well, the D7 LED on your Photon should light up when light is exceeded. 



