//# Tugas-Sistem-Mikrokontroler
//Pengukur Kadar Gula Dalam Darah
#include <LiquidCrystal.h>

// RS, E, D4, D5, D6, D7
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

const int maxData = 20;
int dataGula[maxData];
int indexData = 0;

void setup() {
  Serial.begin(9600);
  lcd.begin(16, 2);
  lcd.print("Monitoring Gula");

  pinMode(7, OUTPUT); // LED Biru
  pinMode(8, OUTPUT); // LED Hijau
  pinMode(9, OUTPUT); // LED Kuning
  pinMode(10, OUTPUT); // LED Merah
  pinMode(6, OUTPUT); // Buzzer
}

void loop() {
  int sensorValue = analogRead(A0);
  float kadarGula = map(sensorValue, 0, 1023, 0, 400); // simulasi nilai mg/dL

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Gula: ");
  lcd.print(kadarGula, 0);
  lcd.print(" mg/dL");

  lcd.setCursor(0, 1);
  String status = cekStatus(kadarGula);
  lcd.print(status);

  simpanData(kadarGula);
  tampilkanGrafikSerial();

  delay(3000); // update tiap 5 detik
}

String cekStatus(float gula) {
  // Reset semua indikator
  digitalWrite(7, LOW);
  digitalWrite(8, LOW);
  digitalWrite(9, LOW);
  digitalWrite(10, LOW);
  noTone(6); // stop buzzer dulu

  if (gula < 70) {
    digitalWrite(7, HIGH); // Biru
    return "Rendah";
  } else if (gula <= 140) {
    digitalWrite(8, HIGH); // Hijau
    return "Normal";
  } else if (gula <= 199) {
    digitalWrite(9, HIGH); // Kuning
    tone(6, 1000, 200);     // beep pendek (peringatan)
    return "Pra-diabetes";
  } else {
    digitalWrite(10, HIGH); // Merah
    for (int i = 0; i < 3; i++) {
      tone(6, 1000);  // bunyi
      delay(300);
      noTone(6);      // mati
      delay(200);
    }
    return "Tinggi!";
  }
}

void simpanData(float gula) {
  dataGula[indexData] = gula;
  indexData++;
  if (indexData >= maxData) {
    indexData = 0; // reset ke awal (overwriting)
  }
}

void tampilkanGrafikSerial() {
  Serial.println("\n--- Grafik Kadar Gula ---");
  int jumlahData = (indexData < maxData) ? indexData : maxData;
  for (int i = 0; i < jumlahData; i++) {
    Serial.print("[" + String(i) + "] ");
    int bar = map(dataGula[i], 0, 400, 0, 30);
    for (int j = 0; j < bar; j++) {
      Serial.print("|");
    }
    Serial.print(" ");
    Serial.print(dataGula[i]);
    Serial.println(" mg/dL");
  }
}

