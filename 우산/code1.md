```c++
#if defined(ESP32)
#include <WiFi.h>
#include <FirebaseESP32.h>
#elif defined(ESP8266)
#include <ESP8266WiFi.h>
#include <FirebaseESP8266.h>
#endif

//Provide the token generation process info.
#include <addons/TokenHelper.h>

//Provide the RTDB payload printing info and other helper functions.
#include <addons/RTDBHelper.h>

/* 1. Define the WiFi credentials */
#define WIFI_SSID "soft"
#define WIFI_PASSWORD "ss2420052"

//#define WIFI_SSID "PISnet_4BDCD0"
//#define WIFI_PASSWORD "leeinhwan9532"

//For the following credentials, see examples/Authentications/SignInAsUser/EmailPassword/EmailPassword.ino

/* 2. Define the API Key */
#define API_KEY "AIzaSyCWJ3HGPA6KrG9zhm_eYbFroM7NNs1wjeo"

/* 3. Define the RTDB URL */
#define DATABASE_URL "https://sw02-7ded6.firebaseio.com" //<databaseName>.firebaseio.com or <databaseName>.<region>.firebasedatabase.app

/* 4. Define the user Email and password that alreadey registerd or added in your project */
#define USER_EMAIL "softplay008@gmail.com"
#define USER_PASSWORD "swplay!@#"

//Define Firebase Data object
FirebaseData fbdo;

FirebaseAuth auth;
FirebaseConfig config;

#include <SoftwareSerial.h>
#include <DFPlayer_Mini_Mp3.h>

#include <SimpleTimer.h>

SimpleTimer t1;
SimpleTimer t2;

int trig = 14;      // 송신을 D5번 핀으로 설정
int echo = 12;      // 수신을 D6번 핀으로 설정

String gate_state = "";
String mp3_onoff = "OFF";

void setup()
{
  
  Serial.begin(9600);

  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Connecting to Wi-Fi");
  while (WiFi.status() != WL_CONNECTED)
  {
    Serial.print(".");
    delay(300);
  }
  Serial.println();
  Serial.print("Connected with IP: ");
  Serial.println(WiFi.localIP());
  Serial.println();

  Serial.printf("Firebase Client v%s\n\n", FIREBASE_CLIENT_VERSION);

  /* Assign the api key (required) */
  config.api_key = API_KEY;

  /* Assign the user sign in credentials */
  auth.user.email = USER_EMAIL;
  auth.user.password = USER_PASSWORD;

  /* Assign the RTDB URL (required) */
  config.database_url = DATABASE_URL;

  /* Assign the callback function for the long running token generation task */
  config.token_status_callback = tokenStatusCallback; //see addons/TokenHelper.h

  Firebase.begin(&config, &auth);

  //Comment or pass false value when WiFi reconnection will control by your code or third party library
  Firebase.reconnectWiFi(true);
  
  pinMode(trig, OUTPUT);    // 송신으로 연결된 핀을 OUTPUT으로 설정
  pinMode(echo, INPUT);     // 수신으로 연결된 핀을 INPUT으로 설정

  mp3_set_serial(Serial);
  delay(1);
  mp3_set_volume(30);
  
  t1.setInterval(1000,fn1);
  t2.setInterval(1000,fn2);
  
}

void loop()
{
  t1.run();
  t2.run();
}

void fn1(void) {
  digitalWrite(trig, HIGH);     // 초음파센서 신호 발사
  delayMicroseconds(10);        // 딜레이
  digitalWrite(trig, LOW);      // 초음파 신호 정지

  int distance = pulseIn(echo, HIGH) * 17 / 1000;

  // 측정된 거리 값를 시리얼 모니터에 출력합니다.
  Serial.print("거리 : ");
  Serial.print(distance);
  Serial.println(" cm");

  if(gate_state=="Y") {
    mp3_onoff = "OFF"; 
  }

  if(distance<30) { 
    if(gate_state=="N") {
      Serial.println("인증을 하지 않고 들어왔다. mp3_onoff를 ON으로 설정"); 
      mp3_onoff = "ON"; 
    }
  }

  if(mp3_onoff=="ON") {
    Serial.println("MP3를 울립니다.");
    mp3_play(1);
    delay(2000);    
  } else if(mp3_onoff=="OFF") {
    Serial.println("MP3를 끕니다");
    mp3_stop();     
  }
  
  delay(1000);
}

void fn2(void) {
  gate_state = Firebase.getString(fbdo, "umbrella_gate/gate_state") ? fbdo.to<const char *>() : fbdo.errorReason().c_str();    
  gate_state = gate_state.substring(gate_state.indexOf("a")+1, gate_state.indexOf("b"));
  
  Serial.print("gate_state: ");
  Serial.println(gate_state);   // N

}


// D3 : 3
// D4 : 4
// D5 : 14
// D6 : 12
// D7 : 13
// weoms d1 r1

// D8 - 15
// D7 - 13
// D6 - 12 (온습도)
// D5 - 14 x
// D4 - 2  x
// A0 - gas

```
