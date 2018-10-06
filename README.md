# Mini Autonomous Car
 In these project, an arduino rule based autonomous car was created. The vehicle has three capabilities.
- watch line 
![Alt Text](https://github.com/UlucFVardar/RuleBased-3D-printed-Mini-Car/blob/master/images/g1.gif?raw=true)

- escape from obstacles
![Alt Text](https://github.com/UlucFVardar/RuleBased-3D-printed-Mini-Car/blob/master/images/g2.gif?raw=true)

- watch line (when there is an obstacles on the road stop)
![Alt Text](https://github.com/UlucFVardar/RuleBased-3D-printed-Mini-Car/blob/master/images/g3.gif?raw=true)



#### Code is also loaded here. will be updated as soon as possible with English explanations!
```c
//-----------@Author by UlucFurkanVardar


#include <SharpIR.h>          //IR Sharp sensörü kullanabilmek için kullandığımız kütüphane, bu kütüphaneyi  kütüphane düzenleyicisinden indirebilir.
#include <Servo.h>            //Servoları kullanabilmek için gerekli olan kütüphane 

SharpIR sensor(GP2Y0A21YK0F, A0);      // IRSharptan bir sensör objesi tanımladık ve bu markanın üç farklı ürününü ayırt edebilmek için elimizde olan ürünün data sheetindeki ID sini kullandık.
Servo LeftWheel;                       //Continues servoları kullanırken Sol tarafta 90-180 arası ileri aracı ileri götürecektir.
Servo RightWheel;                      //Continues servoları kullanırken Sag tarafta  0-90  arası ileri aracı ileri götürecektir.

//Projemizde arduino pro mini kullanıyoruz ve giriş çıkış pinlerini hangi pine neyin takıldığını anlayabilmek için pinlerimize isim vererek devam ediyoruz.
const int BuzzerPin = 3;
const int ButtonPin = 2;
const int LeftSensor = 4;
const int CenterSensor = 7;
const int RightSensor = 8;
const int LeftServoPin = 5;
const int RightServoPin = 6;
// Kod boyunca kullandığımız gereki olan değişkenler.
int distance = 0;       //IR sharp sensörün mesafe ayarlaması için.
int ButtonCounter = 0;  //Açılışta modu ayarlayabilmemizi sağlayan Butonunn tıklanmasını sayan counter.
int LeftS = 0;          //çizgi izleme modülünün gerekli değişkenleri.
int CenterS = 0;
int RightS = 0;         // 0 siyahı, 1 beyazı temsil ediyor.
//Buzzerın notaları için gerekli olanlar.
int Number_of_notes = 8;
int C = 262;
int D = 294;
int E = 330;
int F = 349;
int G = 392;
int A = 440;
int B = 494;
int C_ = 523;
int notalar[] = {C, D, E, F, G, A, B, C_};


void setup() {
  Serial.begin(9600);               //PinMode ve Serial ayarlamaları.
  pinMode(ButtonPin, INPUT);
  pinMode(13, OUTPUT);
  pinMode(RightSensor, INPUT);
  pinMode(CenterSensor, INPUT);
  pinMode(LeftSensor, INPUT);
  digitalWrite(13, LOW);
  digitalWrite(ButtonPin, HIGH);

  LeftWheel.attach(LeftServoPin);    //Servo tanımlamarı.
  RightWheel.attach(RightServoPin);

  delay(600);

  if ( digitalRead(ButtonPin) == 0) {//açılışta buyona tıklanmasuını sayan eğer tıklanma sayılırsa 13. gömülü olan led bir şere yanıp sönecek ve bir melodi çalacak.
    ButtonCounter++;
    digitalWrite(13, HIGH);
    delay(50);
    digitalWrite(13, LOW);
    tone(BuzzerPin, notalar[0]);
    delay(500);
    noTone(BuzzerPin);
    delay(20);
  }
  delay(500);
  if (digitalRead(ButtonPin) == 0) {//açılışta buyona tıklanmasuını sayan eğer tıklanma sayılırsa 13. gömülü olan led bir şere yanıp sönecek ve bir melodi çalacak.
    ButtonCounter++;
    digitalWrite(13, HIGH);
    delay(100);
    digitalWrite(13, LOW);
    tone(BuzzerPin, notalar[1]);
    delay(500);
    noTone(BuzzerPin);
    delay(20);
  }
  Serial.print("Buton Counter : "); //kontrol amaçlı ButtonCounter ekrana basılıyor.
  Serial.println(ButtonCounter);
  delay(1000);
}

void loop() {
  if (ButtonCounter == 1) {       //Eğer bir kere tıklandı ise çizgi izleme modunda araç çalışacak.
    line_tracking();
  }
  else if (ButtonCounter == 2) {  //iki kere tıklandıysa otonom sürüş modunda çalışacak.
    autonomous_driving();
  }
  else if (ButtonCounter == 0) {  //tıklanmadan açıldıysa çizgi izlerken önüne engel çıkarsa durarak çalışacak.
    autonomous_Line_tracking();
  }
  else {      //aksi halde gömülü led yanığ sönecek araç duracak ve tüm meledilerini sıra ile çalacak.
    digitalWrite(13, HIGH);
    delay(100);
    digitalWrite(13, LOW);
    delay(100);
    RightWheel.write(90);
    LeftWheel.write(95);
    for (int i = 0; i < Number_of_notes; i++)
    {
      tone(BuzzerPin, notalar[i]);
      delay(500);
      noTone(BuzzerPin);
      delay(20);
    }
    noTone(BuzzerPin);
  }

  if (digitalRead(ButtonPin) == 0) { //araç ilerler haldeysen butona tıklanırsa araç duracak ve melodi çalmaya başlıyacak.
    ButtonCounter = 5;
  }

}
void autonomous_Line_tracking() {

  distance = sensor.getDistance();
  if (distance > 12) {
    line_tracking();
    noTone(BuzzerPin);

  }
  else {
    RightWheel.write(90);
    LeftWheel.write(95);
    tone(BuzzerPin, notalar[7]);
    delay(200);

  }
}
void autonomous_driving() {
  /*
    otonom ilerlemenin mantığı nedir?
    basit bir şekilde önüne engel çıktığında o yönde ilerlemeyi bırakması.
    ardından hangi tarafa döneceiğinin mantıklı kararını verme.
    bir tarafa dönüş başlatma taki önü açılana dek.
    Bizim robotumuzun yanlarında başka sensörleri olmadığı için mantıklı karar yerine biz aracımızı sola göndürüyoruz.
  */
  distance = sensor.getDistance();    // eklediğimiz kütüphaneden yararlanark daha önce mesafe hesaplamaya yarayan fonksiyonu çağırıyoruz.
  if (distance <= 12) {               // mesafenin 12ye eşit veya kücük olduğu durumlarda aracamız dönüyor.
    RightWheel.write(50);
    LeftWheel.write(95);
  }
  else {                              // aksi halde düz gitmeye devam ediyor.
    LeftWheel.write(130);
    RightWheel.write(50);
  }
}

void line_tracking() {
  /*
    çizgi izlemenin mantığı nedir?
    bir robot çizgi izlemesi için programlanırken düşünülmesi gereken çizgiden ÇIKTIĞI anlardır
    çıktı anlar düzeltilmedilir.
    eğer doğru düzeltmeler yapılırsa robor çizgiyi takip ederek ilerlemeye devam edecektir.
    bu durumda robotun çizgiden çıkmaya başladığı durumları sensörlerimizle değerlendirmeliyiz.
  */
  LeftS = digitalRead(LeftSensor);
  CenterS = digitalRead(CenterSensor);
  RightS = digitalRead(RightSensor);
  //her bir loop turunda güncel sensör verileri alnmalı

  // 1 . soldan çizgiden çıkar ise
  if (LeftS == 1 && RightS == 0 ) {
    //sag dön
    LeftWheel.write(130);
    RightWheel.write(90);
  }
  //2. sagdan çizgiden çıkıyor ise
  else if (RightS == 1 && LeftS == 0 ) {
    //sola dön
    RightWheel.write(50);
    LeftWheel.write(95);
  }
  //Aksi halde aracın düz gitmesi gerekiyor.
  else {
    LeftWheel.write(130);
    RightWheel.write(50);
  }
}
```
