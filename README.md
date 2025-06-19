
🌊 Medidor de Nível de Água com Alerta de Enchentes

🛑 Problema
No Brasil e em diversas partes do mundo, enchentes são tragédias recorrentes que causam perdas humanas, materiais e ambientais. A falta de sistemas de alerta eficientes e acessíveis dificulta a resposta rápida da população em áreas de risco. Muitas comunidades não têm acesso a tecnologias que avisem antecipadamente sobre o aumento do nível da água.

💡 Solução Proposta
Este projeto propõe um Sistema de Medição e Alerta de Nível de Água utilizando a plataforma Arduino UNO, sensores de distância (ultrassônicos), LEDs de alerta, buzzer sonoro, um display LCD I2C, RTC (Relógio de Tempo Real) e EEPROM para armazenamento dos eventos.

📷 Ilustração do Projeto

 ![Medidor-de-agua](https://github.com/user-attachments/assets/c2a6ca02-30f9-4946-89ed-dd341250371c)



🧰 Componentes Utilizados
Arduino UNO

Sensor Ultrassônico HC-SR04

Módulo RTC DS3231

Display LCD 16x2 com I2C

LEDs (verde, amarelo e vermelho)

Buzzer

EEPROM interna do Arduino

Protoboard e jumpers

🧭 Como Funciona
O sensor ultrassônico mede continuamente a distância do nível da água.

O LCD mostra o nível atual e a hora.

LEDs e o buzzer alertam com cores e sons:

Verde: nível baixo (seguro)

Amarelo: nível de atenção

Vermelho + buzzer: nível crítico (risco de enchente)

Os eventos críticos são registrados na EEPROM com data e hora pelo RTC.

💡 Funcionamento
- *0 a 5 cm* → LED verde aceso, mensagem Baixo, buzzer desligado.
- *6 a 10 cm* → LED amarelo aceso, mensagem Atencao, buzzer ligado.
- *11 cm ou mais* → LED vermelho aceso, mensagem Perigo, buzzer ligado, evento salvo na EEPROM.
- O RTC fornece horário no display e para os registros.
- O LCD exibe nível e hora.

 🔬 Acesse a Simulação Online


🔗 Simular no Wokwi ==> https://wokwi.com/projects/434123898181502977

🎥 Vídeo Demonstrativo 



📺 Assista ao vídeo explicativo aqui ==> https://youtu.be/ID-Zg_hF6O8?si=NQBlu5XZkGzBMZ94



🧾 Código-Fonte (Arduino .ino)

#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <RTClib.h>
#include <EEPROM.h>

// Inicialização dos componentes
LiquidCrystal_I2C lcd(0x27, 16, 2);
RTC_DS3231 rtc;

// Pinos
const int trigPin = 9;
const int echoPin = 10;
const int ledVerde = 4;
const int ledAmarelo = 2;
const int ledVermelho = 7;
const int buzzer = 3;

void setup() {
  Serial.begin(9600);
  lcd.init();
  lcd.backlight();

  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(ledVerde, OUTPUT);
  pinMode(ledAmarelo, OUTPUT);
  pinMode(ledVermelho, OUTPUT);
  pinMode(buzzer, OUTPUT);

  if (!rtc.begin()) {
    lcd.print("Erro no RTC");
    while (1);
  }

  if (rtc.lostPower()) {
    rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));
  }

  lcd.setCursor(0, 0);
  lcd.print("  SafePlace");
  lcd.setCursor(0, 1);
  lcd.print(" Monitorando...");
  delay(2000);
}

// Função que salva os dados na EEPROM
void salvarEvento(float distancia) {
  DateTime agora = rtc.now();
  int addr = EEPROM.read(0);
  if (addr > 100) addr = 1;

  EEPROM.write(addr++, (byte)distancia);
  EEPROM.write(addr++, agora.hour());
  EEPROM.write(addr++, agora.minute());
  EEPROM.write(addr++, agora.second());
  EEPROM.write(0, addr);

  Serial.print("Evento salvo: ");
  Serial.print(distancia);
  Serial.print(" cm em ");
  Serial.print(agora.hour());
  Serial.print(":");
  Serial.print(agora.minute());
  Serial.print(":");
  Serial.println(agora.second());
}

void loop() {
  long duracao;
  float distancia;

  // Mede distância
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  duracao = pulseIn(echoPin, HIGH);
  distancia = duracao * 0.034 / 2;

  Serial.print("Distancia: ");
  Serial.print(distancia);
  Serial.println(" cm");

  DateTime agora = rtc.now();
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Nivel:");

  // Lógica de alerta
  if (distancia <= 5) {
    lcd.print(" Baixo");
    digitalWrite(ledVerde, HIGH);
    digitalWrite(ledAmarelo, LOW);
    digitalWrite(ledVermelho, LOW);
    digitalWrite(buzzer, LOW);
  }
  else if (distancia <= 10) {
    lcd.print(" Atencao");
    digitalWrite(ledVerde, LOW);
    digitalWrite(ledAmarelo, HIGH);
    digitalWrite(ledVermelho, LOW);
    digitalWrite(buzzer, HIGH);
  }
  else {
    lcd.print(" Perigo");
    digitalWrite(ledVerde, LOW);
    digitalWrite(ledAmarelo, LOW);
    digitalWrite(ledVermelho, HIGH);
    digitalWrite(buzzer, HIGH);
    salvarEvento(distancia);
  }

  // Exibe hora atual
  lcd.setCursor(0, 1);
  if (agora.hour() < 10) lcd.print("0");
  lcd.print(agora.hour());
  lcd.print(":");
  if (agora.minute() < 10) lcd.print("0");
  lcd.print(agora.minute());
  lcd.print(":");
  if (agora.second() < 10) lcd.print("0");
  lcd.print(agora.second());

  delay(1000);
}
