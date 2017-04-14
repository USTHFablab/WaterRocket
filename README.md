//  SENSOR: (SDA,SCL) = (A4,A5)
//  SERVO : (Input)   = (10)

#include <Servo.h> 
#include <SFE_BMP180.h>
#include <Wire.h>

Servo gServo;
SFE_BMP180 pressure;

void setup() {
  gServo.attach(10);      // 10 = Signal
  
  pinMode(13,OUTPUT);     // 13 = Light
  Serial.begin(9600);
  Serial.print("\nREBOOT");

  if (pressure.begin()) 
  Serial.println(" success");
  else {Serial.println(" fail\n\n"); while(1);}
  Serial.println("-----------------------------------");
}

int i=0;
float dP=0.4;   // Pressure difference that activates servo
float rdP;      // Real pressure difference
int R=0;        // Rocket ready
double T,Pa,p0,oPa;

void loop(){
  char status;
  digitalWrite(13,LOW);
  gServo.write(0);
  
  status = pressure.startTemperature();
  if (status != 0) {
    delay(status);

    status = pressure.getTemperature(T);
    if (status != 0) {

      status = pressure.startPressure(3);
      if (status != 0) {
        delay(status);
        status = pressure.getPressure(Pa,T);
        Serial.print("- Pa = ");
        Serial.print(Pa,2);
        Serial.print(" mb, R = ");
        Serial.print(R);
        Serial.print(" //");
      }
    }
  }

  rdP = Pa - oPa;
  if (i==0) {i=1;}
  else {
    if (R==0 && (oPa-Pa)>dP) {    // STATE = STAND , PRESS DROP
      R=1;                        // STATE = LAUNCH
      Serial.print("    Fire: Pa - oPa = ");
      Serial.print(rdP);
    }
    if (R==1 && (Pa-oPa)>dP) {    // STATE = LAUNCH, PRESS RISE
      R=2;                        // STATE = FALL
      Serial.print("    Stop: Pa - oPa = ");
      Serial.print(rdP);
    }
    if (R==2 && (oPa-Pa)>dP) {    // STATE = FALL  , PRESS DROP
      Serial.print("    Open: Pa - oPa = ");
      Serial.print(rdP);
      digitalWrite(13,HIGH);
      gServo.write(90);
      delay(1000);
      gServo.write(0);
      digitalWrite(13,LOW);
      R=0;
    }
  }
  Serial.print("\n");
  oPa=Pa;
  delay(100);
}
