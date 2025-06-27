# BlueStamp Lie Detector
My project will be a close replica of a simple lie detector. It will include a GSR sensor that can check the electrical conductivity of your skin to see how much your sweating, as well as a pulseSensor that will check your hearbeat and monitor for sudden changes. These will help indicate whether a person is telling the truth; saying a lie will often cause nervousness. When the sensors detect unusual signals recieved, the buzzer will sound indicating that a lie has been told.

You should comment out all portions of your portfolio that you have not completed yet, as well as any instructions:
```HTML 
<!--- This is an HTML comment in Markdown -->
<!--- Anything between these symbols will not render on the published site -->
```

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Alex W | Dougherty Valley | Mechanical Engineering | Incoming Freshmen

**Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.**

![Headstone Image](Headshot.png)

<!---# Final Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/0fZ_ZQBOVEA?si=D5DX-H6xp88KZkWD" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE



# Second Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/y3VAmNlER5Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your second milestone, explain what you've worked on since your previous milestone. You can highlight:
- Technical details of what you've accomplished and how they contribute to the final goal
- What has been surprising about the project so far
- Previous challenges you faced that you overcame
- What needs to be completed before your final milestone -->

# First Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/0fZ_ZQBOVEA?si=D5DX-H6xp88KZkWD" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

For my first milestone, my plan is to successfully build the hardware part of the lie detector:
- I have wired all the components together
- I did research on the GSR sensor and learned how to use it
- I began to work on my code and have some simple code on getting readings from the GSR sensor
- I faced challenges with understanding the wiring
- I also had a problem in privacy & security which didn't allow me to connect the arduino software to the USB
- My plan now is to begin to code the lie detector and actually understand all the coding I'm doing

# Schematics 
Here's where you'll put images of your schematics. [Tinkercad](https://www.tinkercad.com/blog/official-guide-to-tinkercad-circuits) and [Fritzing](https://fritzing.org/learning/) are both great resoruces to create professional schematic diagrams, though BSE recommends Tinkercad becuase it can be done easily and for free in the browser. 
![Headstone Image](IMG_2448.png)

# Code
Here's where you'll put your code. The syntax below places it into a block of code. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize it to your project needs. 

```c++
#define USE_ARDUINO_INTERRUPTS true
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <PulseSensorPlayground.h>

// Pin definitions
const int Pulse_Pin   = A0;
const int GSR_Pin     = A2;
const int Buzzer_Pin  = 9;
const int LED_Pin     = 13;

// Object declarations
PulseSensorPlayground pulseSensor;
LiquidCrystal_I2C lcd(0x27, 16, 2); // Use 0x3F if 0x27 doesn't work

// Calibration data
int baselineBPM = 0;
bool calibrated = false;
int gsrBaseline = 0;
int gsrThreshold = 10;

void setup() {
  Serial.begin(9600);
  pinMode(Buzzer_Pin, OUTPUT);
  pinMode(LED_Pin, OUTPUT);

  // LCD setup
  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Heart Monitor");

  // PulseSensor setup
  pulseSensor.analogInput(Pulse_Pin);
  pulseSensor.setThreshold(520);
  pulseSensor.blinkOnPulse(LED_Pin);
  pulseSensor.begin();

  // === Calibrate BPM ===
  Serial.println("🔁 Calibrating BPM...");
  lcd.setCursor(0, 1);
  lcd.print("Calibrating BPM...");
  
  long totalBPM = 0;
  int count = 0;
  unsigned long startTime = millis();

  while (millis() - startTime < 5000) {
    if (pulseSensor.sawStartOfBeat()) {
      int bpm = pulseSensor.getBeatsPerMinute();
      if (bpm > 0) {
        totalBPM += bpm;
        count++;
      }
    }
    delay(20);
  }

  if (count > 0) {
    baselineBPM = totalBPM / count;
    calibrated = true;
    Serial.print("✅ Baseline BPM: ");
    Serial.println(baselineBPM);

    lcd.setCursor(0, 1);
    lcd.print("Baseline BPM:    ");
    lcd.setCursor(13, 1);
    lcd.print(baselineBPM);
  } else {
    Serial.println("❌ Could not detect heartbeat.");
    lcd.setCursor(0, 1);
    lcd.print("Calibration Fail ");
  }

  delay(1000);

  // === Calibrate GSR ===
  Serial.println("🔁 Calibrating GSR...");
  lcd.setCursor(0, 1);
  lcd.print("Calibrating GSR..");
  delay(1000);

  long gsrTotal = 0;
  for (int i = 0; i < 100; i++) {  // Reduced from 500 to 100 samples
    gsrTotal += analogRead(GSR_Pin);
    delay(2); // 2ms × 100 = ~200ms
  }
  gsrBaseline = gsrTotal / 100;
  Serial.print("Baseline GSR: ");
  Serial.println(gsrBaseline);

  lcd.setCursor(0, 1);
  lcd.print("Baseline GSR:    ");
  lcd.setCursor(13, 1);
  lcd.print(gsrBaseline);

  delay(1000);
  lcd.clear();
}

void loop() {
  if (!calibrated) return;

  // === BPM Reading ===
  if (pulseSensor.sawStartOfBeat()) {
    int bpm = pulseSensor.getBeatsPerMinute();

      // === GSR Reading (non-blocking sample) ===
    long gsrSum = 0;
    for (int i = 0; i < 50; i++) {  // Reduced from 500 to 50 samples
      gsrSum += analogRead(GSR_Pin);
      delay(2); // 2ms × 50 = ~100ms
    }
    int gsrAvg = gsrSum / 50;
    int gsrDiff = gsrAvg - gsrBaseline;
    Serial.print("BPM: ");
    Serial.print(bpm);
    Serial.print(" | GSR: ");
    Serial.print(gsrAvg);
    Serial.print(" | ΔGSR: ");
    Serial.println(gsrDiff);

    // === Top Row: BPM + GSR ===
    lcd.setCursor(0, 0);
    lcd.print("BPM:");
    lcd.print(bpm);
    lcd.print(" GSR:");
    lcd.print(gsrAvg);
    lcd.print("   "); // clear trailing chars

    // === Bottom Row: Stress Detection ===
    lcd.setCursor(0, 1);
    if (bpm > baselineBPM + 10 || abs(gsrDiff) > gsrThreshold) {
      lcd.print("Status: Stress!     ");
      tone(Buzzer_Pin, 1000);
      delay(200);
      noTone(Buzzer_Pin);
    } else {
      lcd.print("Status: Normal      ");
      noTone(Buzzer_Pin);
    }
  }

  delay(20);
}
```

# Bill of Materials
Here's where you'll list the parts in your project. To add more rows, just copy and paste the example rows below.
Don't forget to place the link of where to buy each component inside the quotation marks in the corresponding row after href =. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize this to your project needs. 
https://www.amazon.com/seeed-studio-Seeedstudio-Grove-sensor/dp/B012TNYDE4/ref=asc_df_B012TNYDE4?mcid=07fd999dade6393caa277e2e14e2d7a0&hvocijid=2338467771319518032-B012TNYDE4-&hvexpln=73&tag=hyprod-20&linkCode=df0&hvadid=721245378154&hvpos=&hvnetw=g&hvrand=2338467771319518032&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9032171&hvtargid=pla-2281435179258&th=1
| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| GSR Sensor | uses electrical conductivity to detect how much sweat a person is producing | $35.20 | <a href="https://www.amazon.com/seeed-studio-Seeedstudio-Grove-sensor/dp/B012TNYDE4/ref=asc_df_B012TNYDE4?mcid=07fd999dade6393caa277e2e14e2d7a0&hvocijid=2338467771319518032-B012TNYDE4-&hvexpln=73&tag=hyprod-20&linkCode=df0&hvadid=721245378154&hvpos=&hvnetw=g&hvrand=2338467771319518032&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9032171&hvtargid=pla-2281435179258&th=1"> Link </a> |
| PulseSensor | uses led light reflection to check the heartbeat of a person | $Price | <a href="https://www.amazon.com/PulseSensor-com-Original-Pulse-Sensor-project/dp/B01CPP4QM0/ref=asc_df_B01CPP4QM0?mcid=cd807f38cef133699200888b12260125&hvocijid=2315411131699841154-B01CPP4QM0-&hvexpln=73&tag=hyprod-20&linkCode=df0&hvadid=721245378154&hvpos=&hvnetw=g&hvrand=2315411131699841154&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9032171&hvtargid=pla-2281435179778&psc=1"> Link </a> |
| Arduino Super Starter Kit Uno R3 | basic starter kit needed for any simple arduino project | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |

# Starter Project

<iframe width="560" height="315" src="https://www.youtube.com/embed/yE2564JcbFw?si=SatDaVpdTPBzAajv" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

For my starter project, I built a Retro Arcade Console. I chose this project because the project included soldering iron into the chip and connecting it with other parts as well as using nuts and screws to build it. It seems pretty complicated and I saw it as a great way to learn and explore more. In the end, I completed the build and created a working gaming console with display screens, a scoreboard, and 6 buttons used to play various games. It also includes a buzzer and can be powered on with both a USB and batteries. Some challenges I faced while building this was that my soldering pen didn't work, so that I had to use my neighbor's. Although this caused a slight delay in my progress, I ultimately finished this project. My next goal is to start my intensive project and build the physical componment by the next milestone.

![Headstone Image](IMG_2349.png)


# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work. Here are some past BSE portfolios that are awesome examples. You can view how they set up their portfolio, and you can view their index.md files to understand how they implemented different portfolio components.
- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)

To watch the BSE tutorial on how to create a portfolio, click here.
