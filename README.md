# Checkpoint-02-edge

# 🍷 Vinheria Agnello — Sistema de Monitoramento Ambiental

> **FIAP — Engenharia de Software**  
> Disciplina: Edge Computing & Computer Systems  
> Checkpoint 02

---
https://canva.link/siwblmhfzmuen32

## 📋 Descrição do Projeto

Sistema embarcado em **Arduino** para monitoramento contínuo das condições ambientais de uma adega/depósito de vinhos. O projeto monitora **temperatura**, **umidade** e **luminosidade** em tempo real, exibindo os dados em um display LCD 16×2 e acionando alertas visuais (LEDs) e sonoros (buzzer) quando os valores saem da faixa ideal para conservação dos vinhos.

Conforme estudos da área, as condições ideais para armazenamento de vinhos são:
- 🌡️ Temperatura: entre **10°C e 15°C**
- 💧 Umidade: entre **50% e 70%**
- 💡 Luminosidade: abaixo de **30%** (escuro)

---

## ⚙️ Hardware Utilizado

| Componente         | Pino               |
|--------------------|--------------------|
| Sensor DHT11       | D2                 |
| Sensor LDR         | A0                 |
| Display LCD I2C    | A4 (SDA), A5 (SCL) |
| RTC DS1307         | A4 (SDA), A5 (SCL) |
| LED Verde          | D8                 |
| LED Amarelo        | D9                 |
| LED Vermelho       | D10                |
| Buzzer             | D7                 |
| Botão 1 (OK)       | D4                 |
| Botão 2 (Alterar)  | D5                 |

---

## 🗂️ Estrutura do Projeto

```
vinheria-agnello/
├── sketch/
│   └── vinheria.ino       # Código-fonte principal
├── assets/
│   └── circuito.png       # Imagem do circuito montado
└── README.md
```

---

## 🚀 Funcionalidades

### Inicialização
- **Animação de boas-vindas** com ilustração de taça e garrafa em caracteres customizados no LCD
- **Barra de carregamento** animada
- **Menu de configuração inicial** (3 passos) via botões:
  1. Idioma: Português / English
  2. Unidade de temperatura: Celsius / Fahrenheit
  3. Fuso horário: UTC-12 a UTC+14

### Monitoramento (loop principal)
- Leitura do DHT11 a cada segundo
- Leitura do LDR com mapeamento `0–100%` via `map()`
- **Média de no mínimo 5 leituras a cada 5 segundos** (req. 7)
- Exibição em tempo real no LCD: temperatura, umidade, luminosidade e hora local

### Sistema de Alertas (baseado na média)

#### 💡 Luminosidade
| Faixa         | LED          | Buzzer    | Mensagem LCD            |
|---------------|--------------|-----------|-------------------------|
| 0 – 30%       | 🟢 Verde      | Sem som   | — (tudo OK)             |
| 30% – 60%     | 🟡 Amarelo    | Sem som   | "Ambiente a meia luz"   |
| > 60%         | 🔴 Vermelho   | Contínuo  | "Ambiente muito CLARO"  |

#### 🌡️ Temperatura
| Faixa         | LED          | Buzzer    | Mensagem LCD            |
|---------------|--------------|-----------|-------------------------|
| 10°C – 15°C   | 🟢 Verde      | Sem som   | — (tudo OK)             |
| > 15°C        | 🟡 Amarelo    | Contínuo  | "Temp. ALTA!"           |
| < 10°C        | 🟡 Amarelo    | Contínuo  | "Temp. BAIXA!"          |

#### 💧 Umidade
| Faixa         | LED          | Buzzer    | Mensagem LCD            |
|---------------|--------------|-----------|-------------------------|
| 50% – 70%     | 🟢 Verde      | Sem som   | — (tudo OK)             |
| > 70%         | 🔴 Vermelho   | Contínuo  | "Umidade ALTA"          |
| < 50%         | 🔴 Vermelho   | Contínuo  | "Umidade BAIXA"         |

### Armazenamento — EEPROM
Cada registro (8 bytes) armazena temperatura, umidade, luminosidade e horário. O endereço faz **wrap-around** circular nos 512 bytes disponíveis.

### Log Serial
A cada 5 segundos é impresso no Monitor Serial um relatório completo com hora local, valores médios e status detalhado de cada grandeza — em PT ou EN conforme idioma configurado.

---

## 📚 Bibliotecas Necessárias

Instale pelo **Arduino IDE → Gerenciar Bibliotecas**:

| Biblioteca            | Finalidade              |
|-----------------------|-------------------------|
| `DHT sensor library`  | Leitura do DHT11        |
| `LiquidCrystal_I2C`   | Display LCD via I2C     |
| `RTClib`              | Módulo RTC DS1307       |
| `Wire` *(built-in)*   | Comunicação I2C         |
| `EEPROM` *(built-in)* | Gravação local de dados |

---

## 🔧 Como Usar

1. Instale as bibliotecas listadas acima no Arduino IDE
2. Conecte os componentes conforme a tabela de pinos
3. Carregue o arquivo `vinheria.ino` na placa
4. Ao iniciar, siga o **menu de configuração**:
   - **Botão 1 (D4):** confirma a opção atual
   - **Botão 2 (D5):** altera o valor/navega entre opções
5. Abra o **Monitor Serial** (9600 baud) para acompanhar os logs

> **Dica:** O RTC armazena sempre a hora **UTC**. O offset é aplicado apenas na exibição, portanto ajuste o RTC com a hora UTC real para obter a hora local correta independente do fuso escolhido.

---

## 🌡️ Faixas de Referência

```
Temperatura  →  OK: 10°C – 15°C  |  Baixa: < 10°C  |  Alta: > 15°C
Umidade      →  OK: 50% – 70%    |  Baixa: < 50%   |  Alta: > 70%
Luminosidade →  OK: até 30%      |  Meia luz: 30–60%  |  Muito claro: > 60%
```

---

## 💻 Código-Fonte

```cpp
/*
  =====================================================
  PROJETO VINHERIA - FIAP
  Monitoramento de Temperatura, Umidade e Luminosidade
  =====================================================
  Pinos:
    DHT11        -> D2
    LDR          -> A0
    LCD I2C      -> A4 (SDA), A5 (SCL)
    RTC DS1307   -> A4 (SDA), A5 (SCL)
    LED Verde    -> D8
    LED Amarelo  -> D9
    LED Vermelho -> D10
    Buzzer       -> D7
    Botao 1      -> D4  (confirmar / avancar)
    Botao 2      -> D5  (alterar valor / navegar)

  FUSOS UTEIS:
    Londres  (GMT/BST) -> UTC+0 (inverno) ou UTC+1 (verao)
    Brasilia (BRT)     -> UTC-3

  FAIXAS IDEAIS (conforme requisitos):
    Luminosidade: 0-30% = escuro (OK), 30-60% = meia luz, >60% = muito claro
    Temperatura : 10°C a 15°C = OK | <10°C = Baixa | >15°C = Alta
    Umidade     : 50% a 70%   = OK | <50%  = Baixa | >70%  = Alta
*/

#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <DHT.h>
#include <RTClib.h>
#include <EEPROM.h>

// ── Pinos ────────────────────────────────────────────
#define DHTPIN        2
#define DHTTYPE       DHT11
#define LDR_PIN       A0
#define LED_VERDE     8
#define LED_AMARELO   9
#define LED_VERMELHO  10
#define BUZZER        7
#define BTN1          4
#define BTN2          5

// ── Limites de temperatura (req 5, 9, 10) ────────────
#define TEMP_MIN    10.0   // abaixo = Temp. Baixa
#define TEMP_MAX    15.0   // acima  = Temp. Alta

// ── Limites de umidade (req 6, 12, 13) ───────────────
#define UMID_MIN    50.0   // abaixo = Umidade Baixa
#define UMID_MAX    70.0   // acima  = Umidade Alta

// ── Limites de luminosidade (req 1, 2, 3) ────────────
#define LUZ_ESCURO   30    // 0-30%  = escuro    -> LED Verde
#define LUZ_MEIA     60    // 30-60% = meia luz  -> LED Amarelo
                           // >60%   = muito claro -> LED Vermelho + Buzzer

// ── Intervalo de media (req 7: 5 leituras / 5s) ──────
#define INTERVALO_MS  5000
#define MIN_LEITURAS  5

// ── EEPROM ───────────────────────────────────────────
#define EEPROM_START  0
#define EEPROM_SIZE   512
#define RECORD_SIZE   8
int eepromAddr = EEPROM_START;

// ── Objetos ──────────────────────────────────────────
DHT dht(DHTPIN, DHTTYPE);
LiquidCrystal_I2C lcd(0x27, 16, 2);
RTC_DS1307 rtc;

// ── Configuracoes do usuario ─────────────────────────
int  utcOffset  = -3;
bool useCelsius = true;
int  idioma     = 0;    // 0=PT  1=EN

// ── Acumuladores para media ───────────────────────────
float somaTemp = 0, somaUmid = 0;
int   somaLuz  = 0, contLeituras = 0;
unsigned long tInicio = 0;

// =====================================================
// CARACTERES CUSTOMIZADOS
// =====================================================
byte charGrau[8] = {0x06,0x09,0x09,0x06,0x00,0x00,0x00,0x00};
byte charGota[8] = {0x04,0x04,0x0E,0x1F,0x1F,0x1F,0x0E,0x00};
byte charSol[8]  = {0x00,0x0A,0x04,0x1B,0x04,0x0A,0x00,0x00};
byte charOK[8]   = {0x00,0x01,0x03,0x16,0x1C,0x08,0x00,0x00};
byte charBell[8] = {0x04,0x0E,0x0E,0x0E,0x1F,0x00,0x04,0x00};
byte charBar[8]  = {0x1F,0x1F,0x1F,0x1F,0x1F,0x1F,0x1F,0x1F};

byte c1_11[]  = { B00010,B00010,B00010,B00010,B00001,B00000,B00000,B00000 };
byte f0_12[]  = { B00000,B00000,B00000,B00001,B00011,B00010,B00010,B00010 };
byte c0_13[]  = { B00000,B00000,B00000,B11111,B11111,B00000,B00000,B00000 };
byte c1_14[]  = { B00000,B01111,B11111,B11111,B11111,B11111,B01111,B00000 };
byte c0_15[]  = { B00000,B11111,B11111,B11111,B11111,B11111,B11111,B00000 };
byte c1_13[]  = { B01000,B01000,B01000,B01000,B10000,B00000,B00000,B00000 };

byte f1_12_frame1[] = { B00110,B00100,B00100,B00100,B10101,B10101,B01110,B00100 };
byte f1_12_frame2[] = { B00110,B00100,B00100,B00100,B11111,B11111,B01110,B00100 };
byte f1_12_frame3[] = { B00110,B00100,B11111,B11111,B11111,B11111,B01110,B00100 };

// =====================================================
// DECLARACAO DE FUNCOES
// =====================================================
void animacaoTaca();
void rodarCicloTaca();
void desenharTacaCompleta();
void animacaoLogo();
void menuConfiguracaoInicial();
void imprimirUTC();
void feedbackConfirm();
void telaInicio();
void exibirLCD(float t, float h, int luz);
void acionarAlertas(float t, float h, int luz);
void mostrarAlertaTemp(float t);
void mostrarAlertaUmid(float h);
void mostrarAlertaLuz(int luz);
void gravarEEPROM(float t, float h, int luz);
void logSerial(float t, float h, int luz);
int  horaLocal();
void desligarTudo();
void esperarSoltar(int pino);
bool lerBotao();

// =====================================================
// DEBOUNCE
// =====================================================
void esperarSoltar(int pino) {
  while (digitalRead(pino) == LOW);
  delay(150);
}

bool lerBotao() {
  while (true) {
    if (digitalRead(BTN1) == LOW) { esperarSoltar(BTN1); return true;  }
    if (digitalRead(BTN2) == LOW) { esperarSoltar(BTN2); return false; }
  }
}

// =====================================================
void setup() {
  Serial.begin(9600);

  pinMode(LED_VERDE,    OUTPUT);
  pinMode(LED_AMARELO,  OUTPUT);
  pinMode(LED_VERMELHO, OUTPUT);
  pinMode(BUZZER,       OUTPUT);
  pinMode(BTN1, INPUT_PULLUP);
  pinMode(BTN2, INPUT_PULLUP);

  Wire.begin();

  Wire.beginTransmission(0x27);
  if (Wire.endTransmission() != 0) {
    lcd = LiquidCrystal_I2C(0x3F, 16, 2);
  }

  lcd.init();
  lcd.backlight();
  lcd.clear();

  animacaoTaca();

  lcd.createChar(6, charBar);
  animacaoLogo();

  lcd.createChar(0, charGrau);
  lcd.createChar(1, charGota);
  lcd.createChar(2, charSol);
  lcd.createChar(3, charOK);
  lcd.createChar(4, charBell);

  dht.begin();

  if (!rtc.begin()) {
    lcd.setCursor(0, 0);
    lcd.print("Erro: RTC!");
    while (1);
  }
  if (!rtc.isrunning()) {
    rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));
  }

  menuConfiguracaoInicial();
  telaInicio();

  tInicio = millis();
}

// =====================================================
void loop() {
  float t = dht.readTemperature();
  float h = dht.readHumidity();
  int ldrBruto = analogRead(LDR_PIN);
  int luz = map(ldrBruto, 0, 1023, 0, 100);

  if (!isnan(t) && !isnan(h)) {
    somaTemp += t;
    somaUmid += h;
    somaLuz  += luz;
    contLeituras++;
    exibirLCD(t, h, luz);
  }

  // Req 7: media de pelo menos 5 leituras a cada 5 segundos
  if (millis() - tInicio >= INTERVALO_MS && contLeituras >= MIN_LEITURAS) {
    float mT = somaTemp / contLeituras;
    float mH = somaUmid / contLeituras;
    int   mL = somaLuz  / contLeituras;

    acionarAlertas(mT, mH, mL);
    gravarEEPROM(mT, mH, mL);
    logSerial(mT, mH, mL);

    somaTemp = somaUmid = somaLuz = contLeituras = 0;
    tInicio = millis();
  }

  delay(1000);
}

// =====================================================
// HORA LOCAL
// =====================================================
int horaLocal() {
  DateTime now = rtc.now();
  return ((int)now.hour() + utcOffset + 24) % 24;
}

// =====================================================
// DESLIGAR TODOS OS LEDS E BUZZER
// =====================================================
void desligarTudo() {
  digitalWrite(LED_VERDE,    LOW);
  digitalWrite(LED_AMARELO,  LOW);
  digitalWrite(LED_VERMELHO, LOW);
  noTone(BUZZER);
}

// =====================================================
// ANIMACAO DA TACA
// =====================================================
void rodarCicloTaca() {
  lcd.createChar(6, f1_12_frame1); desenharTacaCompleta(); delay(700);
  lcd.createChar(6, f1_12_frame2); desenharTacaCompleta(); delay(700);
  lcd.createChar(6, f1_12_frame3); desenharTacaCompleta(); delay(900);
}

void animacaoTaca() {
  lcd.createChar(0, c1_11);
  lcd.createChar(1, f0_12);
  lcd.createChar(2, c0_13);
  lcd.createChar(3, c1_14);
  lcd.createChar(4, c0_15);
  lcd.createChar(5, c1_13);
  rodarCicloTaca();
  rodarCicloTaca();
}

void desenharTacaCompleta() {
  lcd.clear();
  lcd.setCursor(3, 0); lcd.print("InVino");
  lcd.setCursor(0, 1); lcd.print("Boas-vindas");
  lcd.setCursor(12, 0); lcd.write(byte(1));
  lcd.setCursor(13, 0); lcd.write(byte(2));
  lcd.setCursor(14, 0); lcd.write(byte(3));
  lcd.setCursor(15, 0); lcd.write(byte(4));
  lcd.setCursor(11, 1); lcd.write(byte(0));
  lcd.setCursor(12, 1); lcd.write(byte(6));
  lcd.setCursor(13, 1); lcd.write(byte(5));
}

// =====================================================
// TELA INICIALIZANDO + BARRA
// =====================================================
void animacaoLogo() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print(" Inicializando  ");
  lcd.setCursor(0, 1);
  for (int i = 0; i < 16; i++) { lcd.write(byte(6)); delay(90); }
  delay(500);
  for (int i = 0; i < 3; i++) { lcd.noBacklight(); delay(160); lcd.backlight(); delay(160); }
  delay(300);
  lcd.clear();
  lcd.setCursor(0, 0); lcd.print("  Monitoramento ");
  lcd.setCursor(0, 1); lcd.print("  de Ambiente   ");
  delay(1400);
  lcd.clear();
}

// =====================================================
// MENU DE CONFIGURACAO INICIAL
// =====================================================
void menuConfiguracaoInicial() {
  lcd.clear();
  lcd.setCursor(0, 0); lcd.print("> CONFIGURACAO  ");
  lcd.setCursor(0, 1); lcd.print("B1=OK  B2=Alter.");
  delay(2000);

  // Passo 1: Idioma
  lcd.clear();
  lcd.setCursor(0, 0); lcd.print("1/3 Language:   ");
  lcd.setCursor(0, 1); lcd.print(idioma == 0 ? "> Portugues     " : "> English       ");
  while (true) {
    bool b = lerBotao();
    if (b) break;
    idioma = (idioma == 0) ? 1 : 0;
    lcd.setCursor(0, 1);
    lcd.print(idioma == 0 ? "> Portugues     " : "> English       ");
  }
  feedbackConfirm();

  // Passo 2: Unidade
  lcd.clear();
  lcd.setCursor(0, 0); lcd.print(idioma == 0 ? "2/3 Temperatura:" : "2/3 Temperature:");
  lcd.setCursor(0, 1); lcd.print(useCelsius ? "> Celsius (C)   " : "> Fahrenheit(F) ");
  while (true) {
    bool b = lerBotao();
    if (b) break;
    useCelsius = !useCelsius;
    lcd.setCursor(0, 1);
    lcd.print(useCelsius ? "> Celsius (C)   " : "> Fahrenheit(F) ");
  }
  feedbackConfirm();

  // Passo 3: Fuso
  lcd.clear();
  lcd.setCursor(0, 0); lcd.print(idioma == 0 ? "3/3 Fuso (UTC): " : "3/3 Timezone:   ");
  lcd.setCursor(0, 1); imprimirUTC();
  while (true) {
    bool b = lerBotao();
    if (b) break;
    utcOffset++;
    if (utcOffset > 14) utcOffset = -12;
    lcd.setCursor(0, 1); imprimirUTC();
  }
  feedbackConfirm();
}

void imprimirUTC() {
  lcd.print("> UTC");
  if (utcOffset >= 0) lcd.print("+");
  lcd.print(utcOffset);
  lcd.print("          ");
}

void feedbackConfirm() {
  tone(BUZZER, 1000); delay(80); noTone(BUZZER);
  lcd.setCursor(15, 1); lcd.write(byte(3));
  delay(400);
}

// =====================================================
// TELA DE INICIO
// =====================================================
void telaInicio() {
  lcd.clear();
  lcd.setCursor(0, 0); lcd.print(idioma == 0 ? "Config. salva!" : "Settings saved!");
  lcd.setCursor(0, 1);
  lcd.print(useCelsius ? "C " : "F ");
  lcd.print("UTC");
  if (utcOffset >= 0) lcd.print("+");
  lcd.print(utcOffset);
  lcd.print(idioma == 0 ? " PT" : " EN");
  delay(1800);

  for (int i = 3; i >= 1; i--) {
    lcd.clear();
    lcd.setCursor(0, 0); lcd.print(idioma == 0 ? "Iniciando em..." : "Starting in...");
    lcd.setCursor(7, 1); lcd.print(i);
    tone(BUZZER, 800 + i * 100); delay(200); noTone(BUZZER);
    delay(800);
  }

  lcd.clear();
  lcd.setCursor(0, 0); lcd.print(idioma == 0 ? "   Monitorando!" : "  Monitoring!  ");
  tone(BUZZER, 1200); delay(300); noTone(BUZZER);
  delay(800);
  lcd.clear();
}

// =====================================================
// EXIBIR LEITURA EM TEMPO REAL NO LCD
// =====================================================
void exibirLCD(float t, float h, int luz) {
  DateTime now = rtc.now();
  int hora = horaLocal();
  float tShow = useCelsius ? t : (t * 9.0 / 5.0 + 32.0);
  char  uni   = useCelsius ? 'C' : 'F';

  lcd.setCursor(0, 0);
  lcd.print("T:");
  if (tShow < 10) lcd.print(" ");
  lcd.print(tShow, 1); lcd.write(byte(0)); lcd.print(uni);
  lcd.print(" "); lcd.write(byte(1)); lcd.print(":");
  if ((int)h < 10) lcd.print(" ");
  lcd.print((int)h); lcd.print("%");

  lcd.setCursor(0, 1);
  lcd.write(byte(2)); lcd.print(":");
  if (luz < 10) lcd.print(" ");
  lcd.print(luz); lcd.print("% ");
  if (hora < 10)          lcd.print("0"); lcd.print(hora);         lcd.print(":");
  if (now.minute() < 10)  lcd.print("0"); lcd.print(now.minute()); lcd.print(":");
  if (now.second() < 10)  lcd.print("0"); lcd.print(now.second());
}

// =====================================================
// ALERTAS — baseado na media de leituras
// Req 1:  luz <= 30%          -> LED Verde (OK)
// Req 2:  30% < luz <= 60%    -> LED Amarelo + "Amb. meia luz"
// Req 3/4: luz > 60%          -> LED Vermelho + Buzzer + "Amb. muito claro"
// Req 5:  10 <= t <= 15       -> "Temperatura OK"
// Req 8/9: t > 15             -> LED Amarelo + Buzzer + "Temp. Alta"
// Req 10:  t < 10             -> LED Amarelo + Buzzer + "Temp. Baixa"
// Req 6:  50 <= h <= 70       -> "Umidade OK"
// Req 11/12: h > 70           -> LED Vermelho + Buzzer + "Umidade Alta"
// Req 13:  h < 50             -> LED Vermelho + Buzzer + "Umidade Baixa"
// =====================================================
void acionarAlertas(float t, float h, int luz) {
  desligarTudo();

  // ── Luminosidade (req 1, 2, 3, 4) ────────────────
  if (luz > LUZ_MEIA) {
    // Req 3/4: ambiente muito claro -> LED Vermelho + Buzzer continuo
    digitalWrite(LED_VERMELHO, HIGH);
    tone(BUZZER, 800);
    mostrarAlertaLuz(luz);
    noTone(BUZZER);
  } else if (luz > LUZ_ESCURO) {
    // Req 2: ambiente a meia luz -> LED Amarelo
    digitalWrite(LED_AMARELO, HIGH);
    mostrarAlertaLuz(luz);
  }
  // Req 1: escuro -> LED Verde (tratado no bloco "tudo OK" abaixo)

  desligarTudo();

  // ── Temperatura (req 5, 8, 9, 10) ────────────────
  if (t > TEMP_MAX) {
    // Req 8/9: Temp Alta -> LED Amarelo + Buzzer
    digitalWrite(LED_AMARELO, HIGH);
    tone(BUZZER, 1200);
    mostrarAlertaTemp(t);
    noTone(BUZZER);
  } else if (t < TEMP_MIN) {
    // Req 10: Temp Baixa -> LED Amarelo + Buzzer
    digitalWrite(LED_AMARELO, HIGH);
    tone(BUZZER, 1200);
    mostrarAlertaTemp(t);
    noTone(BUZZER);
  }

  desligarTudo();

  // ── Umidade (req 6, 11, 12, 13) ──────────────────
  if (h > UMID_MAX) {
    // Req 11/12: Umidade Alta -> LED Vermelho + Buzzer
    digitalWrite(LED_VERMELHO, HIGH);
    tone(BUZZER, 600);
    mostrarAlertaUmid(h);
    noTone(BUZZER);
  } else if (h < UMID_MIN) {
    // Req 11/13: Umidade Baixa -> LED Vermelho + Buzzer
    digitalWrite(LED_VERMELHO, HIGH);
    tone(BUZZER, 600);
    mostrarAlertaUmid(h);
    noTone(BUZZER);
  }

  desligarTudo();

  // ── Tudo OK: LED Verde ────────────────────────────
  bool tOK  = (t >= TEMP_MIN && t <= TEMP_MAX);
  bool hOK  = (h >= UMID_MIN && h <= UMID_MAX);
  bool lOK  = (luz <= LUZ_ESCURO);   // Req 1: escuro = OK

  if (tOK && hOK && lOK) {
    digitalWrite(LED_VERDE, HIGH);
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.write(byte(3));
    lcd.print(idioma == 0 ? " Tudo normal!" : " All good!");
    lcd.setCursor(0, 1);
    lcd.print(idioma == 0 ? "Media 5s:  OK  " : "5s avg:    OK  ");
    delay(1500);
    lcd.clear();
  } else if (lOK) {
    // Luz OK mas algum sensor de temp/umid esta em alerta:
    // acende verde so para a luz (ja foi tratado acima)
    digitalWrite(LED_VERDE, HIGH);
  }
}

// ─────────────────────────────────────────────────────
// MOSTRAR ALERTA DE TEMPERATURA
// Req 9: "Temp. ALTA" se t > 15  |  Req 10: "Temp. BAIXA" se t < 10
// ─────────────────────────────────────────────────────
void mostrarAlertaTemp(float t) {
  float tShow = useCelsius ? t : (t * 9.0 / 5.0 + 32.0);
  char  uni   = useCelsius ? 'C' : 'F';
  bool  alta  = (t > TEMP_MAX);

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.write(byte(4));
  if (idioma == 0) {
    lcd.print(alta ? " Temp. ALTA!  " : " Temp. BAIXA! ");
  } else {
    lcd.print(alta ? " Temp. HIGH!  " : " Temp. LOW!   ");
  }
  lcd.setCursor(0, 1);
  lcd.print(idioma == 0 ? "Temp.=" : "Temp.=");
  lcd.print(tShow, 1);
  lcd.write(byte(0));
  lcd.print(uni);
  delay(2500);
  lcd.clear();
}

// ─────────────────────────────────────────────────────
// MOSTRAR ALERTA DE UMIDADE
// Req 12: "Umidade ALTA" se h > 70  |  Req 13: "Umidade BAIXA" se h < 50
// ─────────────────────────────────────────────────────
void mostrarAlertaUmid(float h) {
  bool alta = (h > UMID_MAX);

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.write(byte(4));
  if (idioma == 0) {
    lcd.print(alta ? " Umidade ALTA " : " Umidade BAIXA");
  } else {
    lcd.print(alta ? " Humidity HIGH" : " Humidity LOW ");
  }
  lcd.setCursor(0, 1);
  lcd.print(idioma == 0 ? "Umid.=" : "Hum. =");
  lcd.print((int)h);
  lcd.print("%");
  delay(2500);
  lcd.clear();
}

// ─────────────────────────────────────────────────────
// MOSTRAR ALERTA DE LUMINOSIDADE
// Req 2:  "Ambiente a meia luz"  (30 < luz <= 60)
// Req 3:  "Ambiente muito claro" (luz > 60)
// ─────────────────────────────────────────────────────
void mostrarAlertaLuz(int luz) {
  bool muitoClaro = (luz > LUZ_MEIA);

  lcd.clear();
  lcd.setCursor(0, 0);
  if (muitoClaro) {
    lcd.print(idioma == 0 ? "Ambiente muito  " : "Too bright!     ");
    lcd.setCursor(0, 1);
    lcd.print(idioma == 0 ? "CLARO           " : "                ");
  } else {
    lcd.print(idioma == 0 ? "Ambiente a meia " : "Half-light      ");
    lcd.setCursor(0, 1);
    lcd.print(idioma == 0 ? "luz             " : "                ");
  }
  delay(2500);
  lcd.clear();
}

// =====================================================
// EEPROM – GRAVAR
// =====================================================
void gravarEEPROM(float t, float h, int luz) {
  DateTime now = rtc.now();
  int hora = horaLocal();

  if (eepromAddr + RECORD_SIZE >= EEPROM_SIZE)
    eepromAddr = EEPROM_START;

  EEPROM.write(eepromAddr,     (int)t);
  EEPROM.write(eepromAddr + 1, (int)((t - (int)t) * 10));
  EEPROM.write(eepromAddr + 2, (int)h);
  EEPROM.write(eepromAddr + 3, (int)((h - (int)h) * 10));
  EEPROM.write(eepromAddr + 4, luz);
  EEPROM.write(eepromAddr + 5, hora);
  EEPROM.write(eepromAddr + 6, now.minute());
  EEPROM.write(eepromAddr + 7, now.second());

  eepromAddr += RECORD_SIZE;
}

// =====================================================
// LOG SERIAL
// =====================================================
void logSerial(float t, float h, int luz) {
  DateTime now = rtc.now();
  int hora = horaLocal();
  bool pt = (idioma == 0);

  Serial.println(pt ? "===== LOG MEDIA 5s =====" : "===== 5s AVG LOG =====");
  Serial.print(pt ? "Hora (UTC" : "Time (UTC");
  Serial.print(utcOffset >= 0 ? "+" : "");
  Serial.print(utcOffset); Serial.print("): ");
  if (hora < 10)         Serial.print("0"); Serial.print(hora);          Serial.print(":");
  if (now.minute() < 10) Serial.print("0"); Serial.print(now.minute());  Serial.print(":");
  if (now.second() < 10) Serial.print("0"); Serial.println(now.second());

  Serial.print(pt ? "Temperatura : " : "Temperature : ");
  Serial.print(t, 1); Serial.println(useCelsius ? " C" : " F");
  Serial.print(pt ? "Umidade     : " : "Humidity    : ");
  Serial.print(h, 1); Serial.println(" %");
  Serial.print(pt ? "Luminosidade: " : "Light level : ");
  Serial.print(luz); Serial.println(" %");

  // Status temperatura
  if (t >= TEMP_MIN && t <= TEMP_MAX)
    Serial.println(pt ? "Temp   : OK"           : "Temp   : OK");
  else if (t > TEMP_MAX)
    Serial.println(pt ? "Temp   : ALERTA - Alta" : "Temp   : ALERT - High");
  else
    Serial.println(pt ? "Temp   : ALERTA - Baixa": "Temp   : ALERT - Low");

  // Status umidade
  if (h >= UMID_MIN && h <= UMID_MAX)
    Serial.println(pt ? "Umidade: OK"            : "Humid  : OK");
  else if (h > UMID_MAX)
    Serial.println(pt ? "Umidade: ALERTA - Alta" : "Humid  : ALERT - High");
  else
    Serial.println(pt ? "Umidade: ALERTA - Baixa": "Humid  : ALERT - Low");

  // Status luminosidade
  if (luz <= LUZ_ESCURO)
    Serial.println(pt ? "Luz    : OK - Escuro"       : "Light  : OK - Dark");
  else if (luz <= LUZ_MEIA)
    Serial.println(pt ? "Luz    : AVISO - Meia luz"  : "Light  : WARN - Half-light");
  else
    Serial.println(pt ? "Luz    : ALERTA - Mto claro": "Light  : ALERT - Too bright");

  Serial.println();
}
```

---

## 📹 Demonstração

> Link para o vídeo explicativo: *[adicionar link]*  
> Link para simulação (Wokwi/Tinkercad): *[adicionar link]*

---

## 👥 Equipe

| Nome                              | RM       |
|-----------------------------------|----------|
| Enzo S Galbiatti Biagiotti        | RM568894 |
| Leonardo Farias Novaes Aguiar     | RM570991 |
| Paulo Ricardo Siqueira de Carlos  | RM569992 |
| Heitor Ortiz Nogueira             | RM569762 |

---

## 📄 Licença

Projeto acadêmico desenvolvido para a FIAP — 2026.  
Todos os direitos reservados conforme indicado pelo material didático da disciplina.
