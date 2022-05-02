

#include "Arduino.h"
#include "ESP8266.h"
#include "PIR.h"
#include "Servo.h"
#include "PiezoSpeaker.h"



#define WIFI_PIN_TX	11
#define WIFI_PIN_RX	10
#define PIR_PIN_SIG	2
#define SERVOSM_PIN_SIG	3
#define THINSPEAKER_PIN_POS	4




int hostPort = 80;
const int servoSMRestPosition   = 20;  
const int servoSMTargetPosition = 150; 
unsigned int thinSpeakerHoorayLength          = 6;                                                      
unsigned int thinSpeakerHoorayMelody[]        = {NOTE_C4, NOTE_E4, NOTE_G4, NOTE_C5, NOTE_G4, NOTE_C5}; 
unsigned int thinSpeakerHoorayNoteDurations[] = {8      , 8      , 8      , 4      , 8      , 4      }; 

ESP8266 wifi(WIFI_PIN_RX,WIFI_PIN_TX);
PIR pir(PIR_PIN_SIG);
Servo servoSM;
PiezoSpeaker thinSpeaker(THINSPEAKER_PIN_POS);


// define vars for testing menu
const int timeout = 10000;       
char menuOption = 0;
long time0;

// Setup the essentials for your circuit to work. It runs first every time your circuit is powered with electricity.
void setup() 
{
    // Setup Serial which is useful for debugging
    // Use the Serial Monitor to view printed messages
    Serial.begin(9600);
    while (!Serial) ; // wait for serial port to connect. Needed for native USB
    Serial.println("start");
    
    wifi.init(SSID, PASSWORD);
    servoSM.attach(SERVOSM_PIN_SIG);
    servoSM.write(servoSMRestPosition);
    delay(100);
    servoSM.detach();
    menuOption = menu();
    
}

// Main logic of your circuit. It defines the interaction between the components you selected. After setup, it runs over and over again, in an eternal loop.
void loop() 
{
    
    
    if(menuOption == '1') {
    // ESP8266-01 - Wifi Module - Test Code
    //Send request for www.google.com at port 80
    wifi.httpGet(host, hostPort);
    // get response buffer. Note that it is set to 250 bytes due to the Arduino low memory
    char* wifiBuf = wifi.getBuffer();
    //Comment out to print the buffer to Serial Monitor
    //for(int i=0; i< MAX_BUFFER_SIZE ; i++)
    //  Serial.print(wifiBuf[i]);
    //search buffer for the date and time and print it to the serial monitor. This is GMT time!
    char *wifiDateIdx = strstr (wifiBuf, "Date");
    for (int i = 0; wifiDateIdx[i] != '\n' ; i++)
    Serial.print(wifiDateIdx[i]);

    }
    else if(menuOption == '2') {
    // Infrared PIR Motion Sensor Module - Test Code
    bool pirVal = pir.read();
    Serial.print(F("Val: ")); Serial.println(pirVal);

    }
    else if(menuOption == '3') {
    // Servo - Generic Metal Gear (Micro Size) - Test Code
    // The servo will rotate to target position and back to resting position with an interval of 500 milliseconds (0.5 seconds) 
    servoSM.attach(SERVOSM_PIN_SIG);         // 1. attach the servo to correct pin to control it.
    servoSM.write(servoSMTargetPosition);  // 2. turns servo to target position. Modify target position by modifying the 'ServoTargetPosition' definition above.
    delay(500);                              // 3. waits 500 milliseconds (0.5 sec). change the value in the brackets (500) for a longer or shorter delay in milliseconds.
    servoSM.write(servoSMRestPosition);    // 4. turns servo back to rest position. Modify initial position by modifying the 'ServoRestPosition' definition above.
    delay(500);                              // 5. waits 500 milliseconds (0.5 sec). change the value in the brackets (500) for a longer or shorter delay in milliseconds.
    servoSM.detach();                    // 6. release the servo to conserve power. When detached the servo will NOT hold it's position under stress.
    }
    else if(menuOption == '4') {
    // Thin Speaker - Test Code
    // The Speaker will play the Hooray tune
    thinSpeaker.playMelody(thinSpeakerHoorayLength, thinSpeakerHoorayMelody, thinSpeakerHoorayNoteDurations); 
    delay(500);   
    }
    
    if (millis() - time0 > timeout)
    {
        menuOption = menu();
    }
    
}



// Menu function for selecting the components to be tested
// Follow serial monitor for instrcutions
char menu()
{

    Serial.println(F("\nWhich component would you like to test?"));
    Serial.println(F("(1) ESP8266-01 - Wifi Module"));
    Serial.println(F("(2) Infrared PIR Motion Sensor Module"));
    Serial.println(F("(3) Servo - Generic Metal Gear (Micro Size)"));
    Serial.println(F("(4) Thin Speaker"));
    Serial.println(F("(menu) send anything else or press on board reset button\n"));
    while (!Serial.available());

    // Read data from serial monitor if received
    while (Serial.available()) 
    {
        char c = Serial.read();
        if (isAlphaNumeric(c)) 
        {   
            
            if(c == '1') 
    			Serial.println(F("Now Testing ESP8266-01 - Wifi Module"));
    		else if(c == '2') 
    			Serial.println(F("Now Testing Infrared PIR Motion Sensor Module"));
    		else if(c == '3') 
    			Serial.println(F("Now Testing Servo - Generic Metal Gear (Micro Size)"));
    		else if(c == '4') 
    			Serial.println(F("Now Testing Thin Speaker"));
            else
            {
                Serial.println(F("illegal input!"));
                return 0;
            }
            time0 = millis();
            return c;
        }
    }
}



