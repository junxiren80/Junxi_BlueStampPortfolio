# Air Quality Tracker
For my project, I'm building an Air Quality Tracker that monitors environmental conditions and displays air quality data. The main components of my project include sensors, a microcontroller, a OLED display, and the Arduino IDE software used to collect and display the data. The sensors will measure environmental factors related to air quality, the microcontroller will process the sensor readings, and the OLED display displays the information received from the Arduino. 


You should comment out all portions of your portfolio that you have not completed yet, as well as any instructions:
```HTML 
<!--- This is an HTML comment in Markdown -->
<!--- Anything between these symbols will not render on the published site -->
```

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Junxi R | Cranbrook Schools | Electrical Engineering | Incoming Sophmore |

**Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.**

![Headstone Image](logo.svg)
  
# Final Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/F7M7imOVGug" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE


# Second Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/y3VAmNlER5Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For my second milestone, I completed the circuit/hardware and made some additional tweaks in the code to optimize performance. It's suprising on how much I learned about the code just during the first week. I'm also glad that I finished the hardware quite fast. A challenge I had was extra pixels showing up at the bottom of the OLED screen. This happened because the Arduino was running out of temporary memory (RAM) from using heavy String variables. To fix this, I changed the code to use lightweight const char* text variables instead. This saved a lot of memory space and made the extra pixels disappear. I also realised in this process that the data was not showing up in the Serial Monitor. I found out that the base code did not implement the actual Serial.print() commands needed to send the sensor numbers down the USB cable to the computer. Before my final milestone, I will need to work on my modification.


# First Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="(https://www.youtube.com/watch?v=Tj5ji4B5pC4)" title="Milestone 1" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

So far, I have worked on creating the circuit schematics in Tinkercad and putting the code into the IDE. Using Tinkercad has allowed me to test the design virtually before building the physical prototype. I have also begun my hardware. One challenge I was facing during the first week is the code that wasn't working on Tinkercad, and not being able to test the code, as I didn't have my actual parts. To complete my project, I will finish assembling the hardware and test the system under different conditions

# Schematics 

![Schematics Image](Air Quality Monitor Schematic.png)

#Code

Original Code

```c++
#include <SPI.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

// --- ADAFRUIT DHT LIBRARY CONFIGURATION ---
#include "DHT.h"
#define DHT11PIN 2
#define DHTTYPE DHT11    
DHT dht(DHT11PIN, DHTTYPE); 
// ------------------------------------------

#define SCREEN_WIDTH 128 
#define SCREEN_HEIGHT 64 
#define OLED_RESET     4  
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

#define sensor    A0 
int gasLevel  = 0;         
String quality =""; 

void sendSensor()
{
  float h = dht.readHumidity();
  float t = dht.readTemperature(); // This defaults to Celsius!
  
  if (isnan(h) || isnan(t)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }
  
  display.setTextColor(WHITE);
  display.setTextSize(1); // CHANGED: Set to standard, clean Size 1 to prevent text overlap
  
  // --- TEMPERATURE ROW ---
  // CHANGED: Moved Y-coordinate to 36 to leave a clean gap under the MQ135 values
  display.setCursor(0, 36); 
  display.print("Temp: ");
  display.print((int)t);     
  display.println(" C");

  // --- HUMIDITY ROW ---
  // CHANGED: Moved Y-coordinate to 50 so it sits cleanly at the bottom of the screen
  display.setCursor(0, 50); 
  display.print("RH:   ");
  display.print((int)h);     
  display.println(" %");
}

void air_sensor()
{
  gasLevel = analogRead(sensor);
  if(gasLevel < 151){
    quality = "Good";
  }
  else if (gasLevel >= 151 && gasLevel < 200){
    quality = "Poor";
  }
  else if (gasLevel >= 200 && gasLevel < 300){
    quality = "Bad";
  }
  else if (gasLevel >= 300 && gasLevel < 500){
    quality = "Toxic!";
  }
  else{
    quality = "Toxic!";   
  }

  display.setTextColor(WHITE);
  display.setTextSize(1); // CHANGED: Using crisp Size 1 for perfect line spacing
  
  // --- AIR QUALITY LINE ---
  // CHANGED: Positioned at the absolute top (Y=2) so nothing cuts it off
  display.setCursor(0, 2);
  display.print("Air Quality: "); 
  display.println(quality);       

  // --- RAW VALUE LINE ---
  // CHANGED: Set Y-coordinate to 16 to sit exactly one row below "Air Quality"
  display.setCursor(0, 16);
  display.print("Value:       ");       
  display.println(gasLevel);      
}

void setup() {
  Serial.begin(9600);
  pinMode(sensor, INPUT);
  
  dht.begin(); 
  
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3c)) { 
    Serial.println(F("SSD1306 allocation failed"));
  }
  
  // --- INTRO SCREEN ---
  display.clearDisplay();
  display.setTextColor(WHITE);
  display.setTextSize(2);
  display.setCursor(45, 10);
  display.println("Air");
  display.setTextSize(1);
  display.setCursor(20, 35);
  display.println("Quality Monitor");
  display.setCursor(40, 50);
  display.println("BY JUNXI"); 
  display.display();
  delay(2000); 
  display.clearDisplay();    
}

void loop() {
  display.clearDisplay();
  air_sensor();
  sendSensor();
  display.display();  
  delay(2000); 
}

```

New Code

```c++
#include <SPI.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

// --- ADAFRUIT DHT LIBRARY CONFIGURATION ---
#include "DHT.h"
#define DHT11PIN 2
#define DHTTYPE DHT11    
DHT dht(DHT11PIN, DHTTYPE); 

#define SCREEN_WIDTH 128 
#define SCREEN_HEIGHT 64 
#define OLED_RESET     4  
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

#define sensor    A0 
int gasLevel  = 0;         

// --- FIXED: Replaced heavy String object with lightweight text arrays ---
const char* quality = "Good"; 

// --- GLOBAL VARIABLES ---
float globalTemp = 0.0;
float globalHumidity = 0.0;
unsigned long lastDHTRead = 0;

void sendSensor()
{
  if (millis() - lastDHTRead >= 2500) {
    lastDHTRead = millis();
    
    float h = dht.readHumidity();
    float t = dht.readTemperature(); 
    
    if (!isnan(h) && !isnan(t)) {
      globalHumidity = h;
      globalTemp = t;
    }
  }
  
  display.setTextColor(WHITE);
  display.setTextSize(1); 
  
  display.setCursor(0, 34); 
  display.print(F("Temp: ")); // FIXED: Added F() macro to save execution RAM
  display.print((int)globalTemp);     
  display.println(F(" C"));

  display.setCursor(0, 48); 
  display.print(F("RH:   "));
  display.print((int)globalHumidity);     
  display.println(F(" %"));
}

void air_sensor()
{
  gasLevel = analogRead(sensor);
  if(gasLevel < 151){
    quality = "Good";
  }
  else if (gasLevel >= 151 && gasLevel < 200){
    quality = "Poor";
  }
  else if (gasLevel >= 200 && gasLevel < 300){
    quality = "Bad";
  }
  else{
    quality = "Toxic!";   
  }

  display.setTextColor(WHITE);
  display.setTextSize(1); 
  
  display.setCursor(0, 2);
  display.print(F("Air Quality: ")); 
  display.println(quality);       

  display.setCursor(0, 16);
  display.print(F("Value:       "));       
  display.println(gasLevel);      
}

void setup() {
  //9600 Baud
  Serial.begin(9600); 
  delay(1000); 
  Serial.println(F("--- System Starting Up ---"));
  
  pinMode(sensor, INPUT);
  dht.begin(); 
  
  // Initialize the screen hardware profile
  display.begin(SSD1306_SWITCHCAPVCC, 0x3C); 
  
  // Wipe out hardware VRAM directly
  display.clearDisplay();
  display.display();
  delay(100);
}

void loop() {
  display.clearDisplay();
  
  air_sensor();
  sendSensor();
  
  // Printing structured string outputs
  Serial.print(F("DATA -> "));
  Serial.print(F("Gas: "));      Serial.print(gasLevel);
  Serial.print(F(" | Quality: ")); Serial.print(quality);
  Serial.print(F(" | Temp: "));    Serial.print((int)globalTemp);
  Serial.print(F(" | Humidity: "));Serial.println((int)globalHumidity);
  
  display.display();  
  delay(1000); // Stable processing step interval
}
```

# Bill of Materials
Here's where you'll list the parts in your project. To add more rows, just copy and paste the example rows below.
Don't forget to place the link of where to buy each component inside the quotation marks in the corresponding row after href =. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize this to your project needs. 

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Arduino UNO R3 | Microcontroller that processes data from the sensors | $16.99 | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Temperature & Humidity Sensor | Receives temperature/humidity | $1.99 | <a href="https://www.amazon.com/Teyleten-Robot-Temperature-Humidity-Raspberry/dp/B0CPF6ZZ73/"> Link </a> |
| Gas Sensor | Air Quality Sensor | $2.99 | <a href="https://www.amazon.com/dp/B07L73VTTY"> Link </a> |
| 12C OLED Display | Displays data received by the Arduino from the sensors | $3.33 | <a href="https://www.amazon.com/dp/B0D2RMQQHR/"> Link </a> |

# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work. Here are some past BSE portfolios that are awesome examples. You can view how they set up their portfolio, and you can view their index.md files to understand how they implemented different portfolio components.
- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)

To watch the BSE tutorial on how to create a portfolio, click here.
