# Lectura-RFID-con-buzzer-en-NodeMCU
En este programa se realiza un programa que detecta una tarjeta RFID y suena un buzzer


```ruby

/*---------------------Bibliotecas------------------------------*/
#include <ESP8266WiFi.h> //biblioteca para la comunicación WiFi con NodeMCU(ESP8266)
//Bibliotecas para sensor RFID
#include <SPI.h> //Para la comunicación serial de rfid, protocolo SPI 
#include <MFRC522.h> //Biblioteca para el sensor RFID Modelo MFRC22
#include <EasyBuzzer.h> //Biblioteca para buzzer
/*---------------------PINES----------------------------------*/
#define RST_PIN D3          // PIN de comunicacion con MFRC522
#define SS_PIN D4          // PIN de comunicacion con MFRC522
#define pinBuzzer D1      //PIN BUZZER

/*---------------------Variables para RFID---------------------*/                                    
byte block;
byte len;
byte i;
uint8_t j;
uint8_t k;
byte buffer1[18];
byte buffer2[18];
/*---------------------Creacion de instancias---------------*/
MFRC522 mfrc522(SS_PIN, RST_PIN); //instancia para MFRC522 


//Frecuencia y duración del zumbido en hertz y milisegundos respectivamente
unsigned int frequency = 1000;
unsigned int duration = 1000;
unsigned int beeps1 = 1;
unsigned int beeps2 = 2;

void setup() {
  Serial.begin(9600);                                           // Inicia la comunicacion Serial con la PC
  SPI.begin();                                                  // inicia la comunicacion con el bus SPI 
  mfrc522.PCD_Init();                                           // Inicia la instancia de MFRC522 card
  Serial.println(F("Leer Datos Personales en el dispositivo:"));    //Muestra en el puerto serial que estamos listos para leer

  //Asignación del pin al BUZZER
  EasyBuzzer.setPin (pinBuzzer);
  EasyBuzzer.beep(frequency,beeps2,done);

}


void loop() {
EasyBuzzer.update();
lectura_RFID();
//beep_chido_buzzer();
}


/*--------Funciones------*/


/*Funcion para realizar la lectura RFID */
void lectura_RFID(){
  
     // Prepare key - all keys are set to FFFFFFFFFFFFh at chip delivery from the factory.
  MFRC522::MIFARE_Key key;
  for ( i = 0; i < 6; i++) key.keyByte[i] = 0xFF;
  MFRC522::StatusCode status;
/*-------------------------------------------------------------------------------------*/
  //Rsetear el bucle si no se presenta una nueva tarjeta en el sensor esto guarda todo el proceso cuando está inactivo
  if ( ! mfrc522.PICC_IsNewCardPresent()) {
    return;
  }
/*Selecciona una de las tarjetas esto devuelve falso si la lectura no se realiza correctamente  y si esto sucede
detenemos el código*/
  if ( ! mfrc522.PICC_ReadCardSerial()) {
    return;
  }
  Serial.println(F("**Tarjeta detectada:**"));
  beep_chido_buzzer();
  //EasyBuzzer.beep(frequency,beeps2,done);
/*-------------------------------------------------------------------------------------*/
  mfrc522.PICC_DumpDetailsToSerial(&(mfrc522.uid));
  Serial.print(F("Nombre: "));
  block = 4;
  len = 18;
  status = mfrc522.PCD_Authenticate(MFRC522::PICC_CMD_MF_AUTH_KEY_A, 4, &key, &(mfrc522.uid)); //line 834 of MFRC522.cpp file
  if (status != MFRC522::STATUS_OK) {
    Serial.print(F("Authentication failed: "));
    Serial.println(mfrc522.GetStatusCodeName(status));
    return;
  }
  status = mfrc522.MIFARE_Read(block, buffer1, &len);
  if (status != MFRC522::STATUS_OK) {
    Serial.print(F("Reading failed: "));
    Serial.println(mfrc522.GetStatusCodeName(status));
    return;
  }
  //PRINT FIRST NAME
  for (j= 0; j < 16; j++)
  {
    if (buffer1[j] != 32)
    {
      Serial.write(buffer1[j]);
    }
  }
  Serial.print(" ");
/*-----------------------------------------GET LAST NAME--------------------------------------------*/
  block = 1;
  status = mfrc522.PCD_Authenticate(MFRC522::PICC_CMD_MF_AUTH_KEY_A, 1, &key, &(mfrc522.uid)); //line 834
  if (status != MFRC522::STATUS_OK) {
    Serial.print(F("Authentication failed: "));
    Serial.println(mfrc522.GetStatusCodeName(status));
    return;
  }
  status = mfrc522.MIFARE_Read(block, buffer2, &len);
  if (status != MFRC522::STATUS_OK) {
    Serial.print(F("Reading failed: "));
    Serial.println(mfrc522.GetStatusCodeName(status));
    return;
  }
  //PRINT LAST NAME
  for (k= 0; k < 16; k++) {
    Serial.write(buffer2[k] );
  }
  /*----------------------------------------------------------------------------------------------------*/
  Serial.println(F("\n**Fin de Lectura**\n"));
  delay(1000); //change value if you want to read cards faster
  mfrc522.PICC_HaltA();
  mfrc522.PCD_StopCrypto1();
  
}

/*------------------------------Funcion donde de llamada al terminar el zumbido------------------------*/
void done() {
  EasyBuzzer.stopBeep();
}

void beep_chido_buzzer(){

      digitalWrite(pinBuzzer, HIGH);
      delay(200);
      digitalWrite(pinBuzzer, LOW);
      delay(100);
      digitalWrite(pinBuzzer, HIGH);
      delay(200);
      digitalWrite(pinBuzzer, LOW);

}

```
