#include <Arduino.h>
#include <ESP8266WiFi.h>
#include <SoftwareSerial.h>
#include <ArduinoJson.h>
#include <Firebase_ESP_Client.h>
#include <NTPClient.h>
#include <WiFiUdp.h>

#include "addons/TokenHelper.h"

#include "addons/RTDBHelper.h"

SoftwareSerial nodemcu(D6, D5); //D6 = Rx & D5 = Tx

#define WIFI_SSID "iot"
#define WIFI_PASSWORD "#12345Abcde"

#define API_KEY "AIzaSyB4jXTqNsS9jfG5WdVvqwLEm4P9rBZt_6c"

#define DATABASE_URL "https://karnaphuli-default-rtdb.asia-southeast1.firebasedatabase.app/"

FirebaseData fbdo;
FirebaseAuth auth;
FirebaseConfig config;

String databasePath;

String pHPath = "/pH";
String tempPath = "/temperature";
String turbPath = "/turbidity";
String timePath = "/timestamp";
String tdsPath= "/tds";

String weekDays[7]={"Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"};

String months[12]={"January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"};

String parentPath;

FirebaseJson json;

WiFiUDP ntpUDP;
NTPClient timeClient(ntpUDP, "pool.ntp.org");

int timestamp;

float pH;
float temperature;
float turbidity;
int tds;

String currentDate;
String formattedTime;

// Timer variables (send new readings every three minutes)
unsigned long sendDataPrevMillis = 0;
unsigned long timerDelay = 10000;

// Initialize WiFi
void initWiFi() {
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Connecting to WiFi ..");
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print('.');
    delay(1000);
  }
  Serial.println(WiFi.localIP());
  Serial.println();
}

// Function that gets current epoch time
unsigned long getTime() {
  timeClient.update();
  unsigned long now = timeClient.getEpochTime();
  return now;
}

bool signupOK = false;

void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  nodemcu.begin(9600);

  initWiFi();
  timeClient.begin();

  config.api_key = API_KEY;

  config.database_url = DATABASE_URL;

    if (Firebase.signUp(&config, &auth, "", "")){
    Serial.println("ok");
    signupOK = true;
  }
  else{
    Serial.printf("%s\n", config.signer.signupError.message.c_str());
  }

  Firebase.reconnectWiFi(true);
  fbdo.setResponseSize(4096);

  // Assign the callback function for the long running token generation task */
  config.token_status_callback = tokenStatusCallback; //see addons/TokenHelper.h

  // Assign the maximum retry of token generation
  config.max_token_generation_retry = 5;

  // Initialize the library with the Firebase authen and config
  Firebase.begin(&config, &auth);

  databasePath = "/Qualities";

    timeClient.begin();
    timeClient.setTimeOffset(21600);
}

void loop() {
  // put your main code here, to run repeatedly:
  StaticJsonBuffer<1000> jsonBuffer;
  JsonObject& data = jsonBuffer.parseObject(nodemcu);

  if (data == JsonObject::invalid()) {
    //Serial.println("Invalid Json Object");
    jsonBuffer.clear();
    return;
  }
   pH=data["pH"];
   temperature=data["temperature"];
   tds=data["tds"];
   turbidity=data["turbidity"];
   
  Serial.print("pH:");
  Serial.println(pH);
  Serial.print("temp:");
  Serial.println(temperature);
  Serial.print("tds :");
  Serial.print(tds);
  Serial.println("ppm");
  Serial.print("turbidity:");
  Serial.println(turbidity);

    if (Firebase.ready() && signupOK && (millis() - sendDataPrevMillis > timerDelay || sendDataPrevMillis == 0)){
    sendDataPrevMillis = millis();

    //Get current timestamp
    timestamp = getTime();
    realtime();

    parentPath= databasePath + "/" + String(timestamp);

    json.set(pHPath.c_str(), String(pH));
    json.set(tempPath.c_str(), String(temperature));
    json.set(turbPath.c_str(), String(turbidity));
    json.set(timePath, String(formattedTime));
    json.set(tdsPath, String(tds));
    
    Serial.printf("Set json... %s\n", Firebase.RTDB.setJSON(&fbdo, parentPath.c_str(), &json) ? "ok" : fbdo.errorReason().c_str());
  }
  
}

void realtime()
{
  timeClient.update();

  time_t epochTime = timeClient.getEpochTime();
  
  formattedTime = timeClient.getFormattedTime();

  int currentHour = timeClient.getHours();

  int currentMinute = timeClient.getMinutes();
  
  int currentSecond = timeClient.getSeconds();

  String weekDay = weekDays[timeClient.getDay()]; 

  struct tm *ptm = gmtime ((time_t *)&epochTime); 

  int monthDay = ptm->tm_mday;

  int currentMonth = ptm->tm_mon+1;

  String currentMonthName = months[currentMonth-1];

  int currentYear = ptm->tm_year+1900;

  currentDate = String(currentYear) + "-" + String(currentMonth) + "-" + String(monthDay);
}
