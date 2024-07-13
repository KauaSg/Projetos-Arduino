#include <LiquidCrystal.h>
#include "DHT.h"
#include <IRremote.h>

#define PIN_RECEIVER 11
#define DHTPIN 2 
#define DHTTYPE DHT22 

DHT dht(DHTPIN, DHTTYPE);

IRrecv receiver(PIN_RECEIVER);

LiquidCrystal lcd(8, 7, 6, 5, 4, 3);

unsigned long ultimaAtualizacao = 0;
const long intervaloAtualizacao = 3000;

bool exibirTemperaturaCond = true;
bool exibirUmidadeCond = false;
bool exibirDistCond = false;
bool exibirOleoCond = false;
bool exibicaoManual = false;
unsigned long inicioExibicao = 0;

int trig = 12;
int echo = 13;
int pinoPot = A0;
int valorPot = 0;

void setup() {
  pinMode(trig, OUTPUT);
  pinMode(echo, INPUT);
  Serial.begin(9600);
  dht.begin();
  lcd.begin(16, 2);
  receiver.enableIRIn();
}

void loop() {
  if (receiver.decode()) {
    traduzirIR();
    receiver.resume();
    exibicaoManual = true;
    inicioExibicao = millis();
  }

  unsigned long currentMillis = millis();
  if (exibicaoManual && currentMillis - inicioExibicao >= 4000) {
    exibicaoManual = false;
    alternarExibicao();
  }

  if (!exibicaoManual && currentMillis - ultimaAtualizacao >= intervaloAtualizacao) {
    ultimaAtualizacao = currentMillis;
    alternarExibicao();
  }
}

void traduzirIR() {
  switch (receiver.decodedIRData.command) {
    case 48:
      exibirTemperaturaCond = true;
      exibirUmidadeCond = false;
      exibirDistCond = false;
      exibirOleoCond = false;
      break;
    case 24:
      exibirTemperaturaCond = false;
      exibirUmidadeCond = true;
      exibirDistCond = false;
      exibirOleoCond = false;
      break;
    case 122:
      exibirTemperaturaCond = false;
      exibirUmidadeCond = false;
      exibirDistCond = true;
      exibirOleoCond = false;
      break;
    case 16:
      exibirTemperaturaCond = false;
      exibirUmidadeCond = false;
      exibirDistCond = false;
      exibirOleoCond = true;
      break;
    default:
      exibirInstrucoes();
      break;
  }
  atualizarDadosSensores();
}

void alternarExibicao() {
  if (exibirTemperaturaCond) {
    exibirTemperaturaCond = false;
    exibirUmidadeCond = true;
  } else if (exibirUmidadeCond) {
    exibirUmidadeCond = false;
    exibirDistCond = true;
  } else if (exibirDistCond) {
    exibirDistCond = false;
    exibirOleoCond = true;
  } else if (exibirOleoCond) {
    exibirOleoCond = false;
    exibirTemperaturaCond = true;
  }
  atualizarDadosSensores();
}

void exibirInstrucoes() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Pressione:1=Temp");
  lcd.setCursor(0, 1);
  lcd.print("2=Umi 3=Vol 4=Nivel");
  exibicaoManual = true;
  inicioExibicao = millis();
}

void exibirTemperatura() {
  float t = dht.readTemperature();
  lcd.clear();
  lcd.setCursor(0, 0);
  if (t < 10) {
    lcd.print("Temp. BAIXA");
  } else if (t <= 20) {
    lcd.print("Temperatura OK");
  } else {
    lcd.print("Temp. ALTA");
  }
  lcd.setCursor(0, 1);
  lcd.print("Temp. = ");
  lcd.print(t);
  lcd.print("C");
}

void exibirUmidade() {
  float h = dht.readHumidity();
  lcd.clear();
  lcd.setCursor(0, 0);
  if (h < 60) {
    lcd.print("Umidade BAIXA");
  } else if (h <= 80) {
    lcd.print("Umidade OK");
  } else {
    lcd.print("Umidade ALTA");
  }
  lcd.setCursor(0, 1);
  lcd.print("Umidade = ");
  lcd.print(h);
  lcd.print("%");
}

void exibirDistancia() {
  float dist = lerDistancia();
  lcd.clear();
  lcd.setCursor(0, 0);
  if (dist < 133) {
    lcd.print("Volume: Alto");
  } else if (dist <= 266) {
    lcd.print("Volume: Ok");
  } else {
    lcd.print("Volume: Baixo");
  }
  lcd.setCursor(0, 1);
  lcd.print("Volume: ");
  lcd.print(dist);
  lcd.print("cm");
}

void exibirOleo() {
  valorPot = analogRead(pinoPot);
  int nivelPetroleo = map(valorPot, 0, 1023, 0, 100);

  lcd.clear();
  if (nivelPetroleo <= 9) {
    lcd.print("Nivel: OK");
    lcd.setCursor(0, 1);
    lcd.print("Nivel: ");
    lcd.print(nivelPetroleo);
  } else if (nivelPetroleo < 50) {
    lcd.print("Nivel: Alerta");
    lcd.setCursor(0, 1);
    lcd.print("Nivel: ");
    lcd.print(nivelPetroleo);
  } else {
    lcd.print("Nivel: PERIGO");
    lcd.setCursor(0, 1);
    lcd.print("Nivel: ");
    lcd.print(nivelPetroleo);
  }
}

float lerDistancia() {
  digitalWrite(trig, LOW);
  delayMicroseconds(2);
  digitalWrite(trig, HIGH);
  delayMicroseconds(10);
  digitalWrite(trig, LOW);

  float duracao = pulseIn(echo, HIGH);
  float dist = duracao / 50.8; 

  return dist;
}

void atualizarDadosSensores() {
  float h = dht.readHumidity();
  float t = dht.readTemperature();
  float dist = lerDistancia();

  valorPot = analogRead(pinoPot);
  int nivelPetroleo = map(valorPot, 0, 1023, 0, 100);

  if (isnan(t) || isnan(h)) {
    Serial.println("Erro no sensor DHT");
    lcd.setCursor(0, 0);
    lcd.print("Erro no sensor!");
    lcd.setCursor(0, 1);
    lcd.print("Verifique conexao");
  } else {
    if (exibirTemperaturaCond) {
      exibirTemperatura();
    } else if (exibirDistCond) {
      exibirDistancia();
    } else if (exibirUmidadeCond) {
      exibirUmidade();
    } else if (exibirOleoCond) {
      exibirOleo();
    }

    Serial.print("Umidade: ");
    Serial.print(h);
    Serial.print(" %       ");
    Serial.print("Temperatura: ");
    Serial.print(t);
    Serial.println(" *C");
    Serial.print("Distancia = ");
    Serial.print(dist);
    Serial.println(" cm");
    Serial.print("Nivel de petroleo: ");
    Serial.println(nivelPetroleo);
  }
}
