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

#include <SimpleTimer.h>

SimpleTimer t1;

#include <MFRC522.h>
#include <SPI.h>

#define SS_PIN D8
#define RST_PIN D3

MFRC522 rfid(SS_PIN, RST_PIN);

char str[15];

String tag_id;

void setup()
{

  Serial.begin(115200);
  
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

  t1.setInterval(1000,fn1);

  SPI.begin();
  rfid.PCD_Init();
}

void loop()
{
  t1.run();
}

void fn1(void) {
  
  if ( ! rfid.PICC_IsNewCardPresent())
    return;
  if ( ! rfid.PICC_ReadCardSerial())
    return;

  String content= "";
  byte letter;
  
  for (byte i = 0; i < rfid.uid.size; i++) {
    content.concat(String(rfid.uid.uidByte[i] < 0x10 ? "0" : ""));
    content.concat(String(rfid.uid.uidByte[i], HEX));
  }
  
  content.toUpperCase();

  Serial.print("UID tag : ");
  Serial.println(content);      // nfc uid

  // 스캔된 태그 uuid 아이디에 해당하는 umbrella_uidf를 가져온다. 
  tag_id = Firebase.getString(fbdo, "umbrella_uid/" + content) ? fbdo.to<const char *>() : fbdo.errorReason().c_str();
  Serial.print("tag_id : ");
  Serial.println(tag_id);

  String borrow_date = "umbrella_list/" + tag_id + "/borrow_date";         // aNOb
  String borrow_state = "umbrella_list/" + tag_id + "/borrow_state";       // aNb
  String user_number = "umbrella_list/" + tag_id + "/user_number";         // aNOb

  Firebase.setString(fbdo, borrow_date, "aNOb");
  Firebase.setString(fbdo, borrow_state, "aNb");
  Firebase.setString(fbdo, user_number, "aNOb");
}
```
