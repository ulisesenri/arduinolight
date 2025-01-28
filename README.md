//created by Henry Zuniga CSULB
#include <DHT.h>// temp and humid sensor Libary which is INPUT
#include <Servo.h> // servo Motor OUTPUT
#include <IRremote.h>// infared sensor libary

#define Type DHT11

int wait = 1000; // gives program assemble
int printWait = 1000; // keep monitor print slow
int sensPin = 7;
int buttonRead =  3;// on board
float humidity;
float tempC;
float tempF;
DHT HT(sensPin,Type);
int servoPin = 11;
int servoAngle ;
Servo Bob; // name of my motor
IRrecv IR(2); // infared sensor pin 2
int weatherOption = 0;
int remoteOption = 0;
int previousAngleW;
int previousAngleR; // keep servo in angle before
int neutralW = 0; // so servo arm does not fall off
int neutralR = 0; // so servo arm does not fall off
int on = 50;
int off = 135;
int startUpCountAngle = 0;


void setup() {
  // put your setup code here, to run once:
  IR.enableIRIn();
  Serial.begin(9600);
  HT.begin();// HUM and TEMP sensor 
  pinMode(buttonRead,INPUT);
  //delay (wait);
  Bob.attach(servoPin,490,1300);// tells arduino BOB is connected to pin chosen 9
  //Bob.writeMicroseconds(1500);
}

void loop() 
{
 // put your main code here, to run repeatedly:
    if(IR.decode())
  {
    Serial.println(IR.decodedIRData.decodedRawData, HEX);
    IR.resume();
  }
  
  //START up arm protect------------------------------------------------
 if(startUpCountAngle == 0)
 {
   servoAngle = 90;
 }
  //END arm protect-----------------------------------------------------

 
  // the START of type setting
  // ------------------------------------------------
  if(IR.decodedIRData.decodedRawData == 0xBB44FF00 )// weatherOption
  {
    startUpCountAngle = 1;// make sure startup is off
    weatherOption = 1;
    remoteOption = 0;

  }

  if(IR.decodedIRData.decodedRawData == 0xBF40FF00 )// remoteOption
  {
    startUpCountAngle =1;// make sure startup is off
    remoteOption = 1;
    weatherOption = 0;
  }
  // the END of type setting
  // ------------------------------------------------

if(weatherOption == 1)
{
  // the START of HUM and TEMP readings
  // ------------------------------------------------
  humidity = HT.readHumidity();
  tempC = HT.readTemperature();
  tempF = HT.readTemperature(true);
  // the END of HUM and TEMP readings
  // ------------------------------------------------

  // JKelectronics

  //the START of intructions based on WEATHER
  //------------------------------------------------
  if (digitalRead(buttonRead) == HIGH)
  {
    tempF = 90;
  }

  Serial.print("Humidity:");
  Serial.print(humidity);
  Serial.print("  Temp in Celcius:");
  Serial.print(tempC);
  Serial.print("  Temp in farh:");
  Serial.print(tempF);
  Serial.print(remoteOption);
  Serial.print(weatherOption);
  Serial.println();
  //delay(printWait);

  if(tempF > 80)
  {
    servoAngle = on;
    previousAngleW = servoAngle;
    neutralW++;
  }

  else if(tempF < 71)
  {
    servoAngle = off;
    previousAngleW = servoAngle;
    neutralW++;
  }
  else if(neutralW == 0)
  {
    servoAngle = 90;
  }
  else  
  {
    servoAngle = previousAngleW;
  }
  //the End of itnructions based on WEATHER
  //------------------------------------------------

}

  // the START of Infared sensors work
  // ------------------------------------------------

if(remoteOption == 1)
{
  
if(IR.decodedIRData.decodedRawData == 0xBA45FF00 )
{
  servoAngle = on;//on
  previousAngleR = servoAngle;
  neutralR ++;
}

else if(IR.decodedIRData.decodedRawData == 0xB946FF00) 
{
  servoAngle = off;//off
  previousAngleR = servoAngle;
  neutralR++;
}
else if(neutralR == 0 )
{
  servoAngle = 90;
}
else 
{
  servoAngle = previousAngleR;
}
// the END of Infared sensors work
// ------------------------------------------------
  // on. BA45FF00
  // off B946FF00

    //weather   BB44FF00
    //remote   BF40FF00

}
  // the START of BOB's work
  // ------------------------------------------------
  Bob.write(servoAngle);
  // the END of BOB's work
  // ------------------------------------------------

}

 

