```C++
#include <SimpleTimer.h>

#include <SoftwareSerial.h>
#include <DFPlayer_Mini_Mp3.h>

SoftwareSerial mp3Serial(2, 3); // 앞의것 3번에 , 뒤의 것 2번에
SoftwareSerial myserial(7, 8);

int trigPin = 5;  
int echoPin = 6;
int distance = 0;

// ----------------------------------------

long previousTime = 0;  
unsigned long cnt=0;        
long interval = 10000;    

// ----------------------------------------

SimpleTimer t1;
SimpleTimer t2;

void setup() {
  Serial.begin(115200);
  mp3Serial.begin(9600);
  
  myserial.begin(115200);

  delay(3000);
  myserial.println("AT");
  delay(1000);

  // SIM 카드 핀(전원 핀) 설정
  pinMode(9, OUTPUT);
  digitalWrite(9, HIGH);
  
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  mp3_set_serial(mp3Serial);
  delay(1);
  mp3_set_volume(30);
  
  t1.setInterval(1000,fn1);
  t2.setInterval(1000,fn2);
}

void loop() {
  t1.run();
  t2.run();
}


void fn1(void) {
  digitalWrite(trigPin, HIGH);      // 초음파센서 신호 발사
  delayMicroseconds(10);            // 딜레이
  digitalWrite(trigPin, LOW);       // 초음파 신호 정지

  distance = pulseIn(echoPin, HIGH) * 17 / 1000;

    Serial.print("거리 : ");
    Serial.print(distance);
    Serial.println(" cm");
}


void fn2(void) {
  if(distance < 10) {        // 사람이 있으면
    Serial.println("사람이 감지되었습니다.");
    
    previousTime = millis();  // 카운트 증가
    cnt++;    
  } else {
    Serial.println("감지된 것이 없습니다.");
  }
      
  if (millis() - previousTime > interval) {  
     previousTime = millis();  
     cnt = 0;
  }

  if(cnt>1) {
    if(distance<10) {
      Serial.println("사람이 감지되었습니다.");
      mp3_play(1);
      sendSMS("1025029532");
      delay(2000);
    }
  }
  
  Serial.print("cnt: ");
  Serial.println(cnt); 
}

void sendSMS(const char* phoneNumber) {
  // SMS 메시지 보내기 AT 명령 전송
  myserial.print("AT+CMGF=1\r");
  delay(1000);
  
  myserial.print("AT+CMGS=\"");
  myserial.print(phoneNumber);
  myserial.print("\"\r");
  delay(1000);
  myserial.write("C0ACB78CC7740020AC10C9C0B418C5C8C2B5B2C8B2E4002E"); 
  delay(1000);
  
  myserial.write(26);
  delay(10000);
 
  // 응답 확인
  while (!myserial.available()) {
    // 응답을 기다리는 동안 대기
  }
 
  // 응답 출력
  while (myserial.available()) {
    Serial.write(myserial.read());
  }
}
```
