# Smart-Socket

Smart AC Charging Protection Socket is an Arduino-based smart charging system that monitors current, temperature, and voltage to prevent overcharging, overheating, and unsafe power conditions. It automatically disconnects power to improve battery life, enhance charging safety, and reduce energy waste.

#include <OneWire.h>
#include <DallasTemperature.h>
#include <LiquidCrystal_I2C.h>
#include <SPI.h>
#include <SD.h>

#define tempPin A0
#define currentPin A2
#define voltagePin A1
#define relayPin 8
#define chipSelect 10

OneWire oneWire(tempPin);
DallasTemperature sensor(&oneWire);
LiquidCrystal_I2C lcd(0x27, 16, 2);

void setup()
{
  Serial.begin(9600);
  pinMode(relayPin, OUTPUT);
  digitalWrite(relayPin, HIGH);
  sensor.begin();
  lcd.init();
  lcd.backlight();
  SD.begin(chipSelect);
  lcd.setCursor(0,0);
  lcd.print("Smart Charger");
  delay(2000);
  
}

void loop()
{
 
  sensor.requestTemperatures();
  float temperature = sensor.getTempCByIndex(0);
  int currentValue = analogRead(currentPin);
  float currentVoltage = currentValue * (5.0 / 1023.0);
  float current = (currentVoltage - 2.5) / 0.185; 
  int voltageValue = analogRead(voltagePin);
  float voltage = voltageValue * (5.0 / 1023.0);

  
  lcd.setCursor(0,0);
  lcd.print("T:");
  lcd.print(temperature);
  lcd.print("C ");
  lcd.setCursor(9,0);
  lcd.print("I:");
  lcd.print(current,1);

 
  if(temperature > 45 || abs(current) < 0.1)
  {
    digitalWrite(relayPin, LOW);
    lcd.setCursor(0,1);
    lcd.print("Charging OFF ");
  }
  else
  {
    digitalWrite(relayPin, HIGH);
    lcd.setCursor(0,1);
    lcd.print("Charging ON ");
  }

  
  File file = SD.open("data.txt", FILE_WRITE);

  if(file)
  {
    file.print("Temp=");
    file.print(temperature);
    file.print(" Current=");
    file.print(current);
    file.print(" Voltage=");
    file.println(voltage);
    file.close();
  }


  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.print(" C ");
  Serial.print("Current: ");
  Serial.print(current);
  Serial.print(" A ");
  Serial.print("Voltage: ");
  Serial.println(voltage);
  delay(2000);
}
