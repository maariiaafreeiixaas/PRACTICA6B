# Pràctica 6

## Practica 6B: Lectura de etiqueta RFIF

### Objectiu
Comprendre el funcionament del bus spi

### Codi

```cpp
#include <SPI.h>
#include <MFRC522.h>
#include <WiFi.h>

#define RST_PIN 9  // Pin de reset
#define SS_PIN 10  // Pin del SS (Slave Select)

MFRC522 mfrc522(SS_PIN, RST_PIN); // Inicialitza el lector RFID

// WiFi
const char* ssid = "nom_de_la_teva_xarxa_wifi";
const char* password = "contrasenya_de_la_teva_xarxa_wifi";

// Email
const char* host = "smtp.gmail.com";
const int port = 465;
const char* emailFrom = "tucorreu@gmail.com";
const char* passwordEmail = "la_teu_contrasenya_de_correu";
const char* emailTo = "destinatari@gmail.com";
const char* subject = "Accés Autoritzat";

WiFiClient client;

void setup() {
  Serial.begin(9600); // Inicialitza la comunicació serial
  SPI.begin(); // Inicialitza la comunicació SPI
  mfrc522.PCD_Init(); // Inicialitza el lector RFID

  // Connecta't a la xarxa WiFi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.println("Connectant a la xarxa WiFi...");
  }
  Serial.println("Connectat a la xarxa WiFi");
}

void loop() {
  // Intenta llegir una targeta RFID present
  if (mfrc522.PICC_IsNewCardPresent() && mfrc522.PICC_ReadCardSerial()) {
    // Verifica si l'UID coincideix amb l'autoritzat
    if (checkUidAuthorization(mfrc522.uid.uidByte, mfrc522.uid.size)) {
      sendEmail(subject, "La targeta RFID autoritzada ha estat utilitzada.");
      delay(1000); // Espera 1 segon abans de tornar a llegir
    } else {
      Serial.println("Accés no autoritzat.");
    }
  }
}

// Funció per verificar si l'UID coincideix amb l'autoritzat
bool checkUidAuthorization(byte *uid, byte uidSize) {
  // Aquí pots implementar la teva lògica per verificar l'UID autoritzat
  // Per exemple, comparar l'UID amb un UID autoritzat emmagatzemat en una matriu
  return true; // Retorna true si és autoritzat, false si no ho és
}

// Funció per enviar un correu electrònic
void sendEmail(const char* subject, const char* message) {
  client.connect(host, port);
  client.println("EHLO example.com");
  client.println("AUTH LOGIN");
  client.println(base64::encode(emailFrom));
  client.println(base64::encode(passwordEmail));
  client.println("MAIL FROM:<" + String(emailFrom) + ">");
  client.println("RCPT TO:<" + String(emailTo) + ">");
  client.println("DATA");
  client.println("From: " + String(emailFrom));
  client.println("To: " + String(emailTo));
  client.println("Subject: " + String(subject));
  client.println();
  client.println(message);
  client.println(".");
  client.println("QUIT");
  client.stop();
}
```

### Descripció de la Sortida pel Port Sèrie

El programa descriu un sistema que llegeix el UID (Identificador Únic) de les targetes RFID mitjançant un lector i el mostra pel port sèrie. La sortida pel port sèrie contindrà informació sobre la detecció de targetes RFID.

Exemple de Sortida pel Port Sèrie:

Lectura del UID
Card UID: 04 3A 7B 2F
Card UID: 08 2B 1D 4E

### Funcionament

La llibreria <SPI.h> és necessària per utilitzar el bus, que permet la comunicació entre el microcontrolador i dispositius externs i <MFRC522.h> proporciona les funcions necessàries per interactuar RFID, permetent la lectura i escriptura de targetes.

S'inicia la comunicació sèrie i seguidament el bus SPI, permetent la comunicació amb el dispositiu.
Després es prepara per a la lectura de la targeta i mostra el missatge "Lectura del UID" pel port sèrie, indicant que el sistema està llest per llegir les targetes.

Es comprova si hi ha una nova targeta RFID a prop del lector per seleccionar-la i llegir el seu UID.
Envia el text "Card UID:" pel port sèrie.
El bucle for itera sobre cada byte del UID de la targeta que serveix per afegeir espai o zero pel formateig del UID en hexadecimal i enviar-lo pel port sèrie.
Finalment s'atura la comunicació amb la targeta actual, deixant el lector preparat per a una nova targeta.