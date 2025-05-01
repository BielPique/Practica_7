# PRÁCTICA 7: BUSES DE COMUNICACIÓN III (I2S)

En esta práctica tenemos como objetivo principal comprender el funcionamiento del bus I2S, el cual se usa para transferir señales de sonido digitales. Para ello hemos trabajado 2 ejercicios donde se implementa este tipo de bus, uno usando la memoria interna y otro usando una tarjeta SD externa.


## EJERCICIO PRÁCTICO 1

En este ejercicio se busca acceder a la matriz de los datos de sonido que se encuentran en la RAM interna de nuestra ESP32-S3. Para ello, hemos realizado un montaje donde usamos una placa de conexión de audio MAX98357 I2S para decodificar la señal digital a señal analógica, y de la placa al altavoz. Una vez hecho el montaje hemos modificado el código otorgado por la práctica donde una vez declaradas las librerías y las variables globales, se inicia la comunicación serie a 115200 baudios, para seguidamente inicializar el decodificador AAC y crear la salida I2S junto con la configuración de ganancia y pines (en este caso la ganancia a 0.125 y los pines en GPIO40, GPIO39 y GPIO38). Finalmente se comienza la reproducción del audio (aac->begin(in,out)). La última parte del código consiste en un loop() donde si el decodificador sigue funcionando, se continúa procesando el audio hasta que no se detecta nada (por que el audio terminó o hubo algún error) y se detiene.

El código es el siguiente:

```
#include "AudioGeneratorAAC.h" 
#include "AudioOutputI2S.h" 
#include "AudioFileSourcePROGMEM.h" 
#include "sampleaac.h" 
AudioFileSourcePROGMEM *in; 
AudioGeneratorAAC *aac; 
AudioOutputI2S *out; 
void setup(){ 
Serial.begin(115200); 
in = new AudioFileSourcePROGMEM(sampleaac, sizeof(sampleaac)); 
aac = new AudioGeneratorAAC(); 
out = new AudioOutputI2S(); 
out -> SetGain(0.125); 
out -> SetPinout(40,39,38); 
aac->begin(in, out); 
} 
void loop(){ 
if (aac->isRunning()) { 
aac->loop(); 
} else { 
aac -> stop(); 
Serial.printf("Sound Generator\n"); 
delay(1000); 
} 
}
```

Por el serial monitor sale:

![image](https://github.com/user-attachments/assets/87b0ab33-20b9-4d88-ba01-b9eb1384df1f)

## EJERCICIO PRÁCTICO 2

En este segundo ejercicio hemos trabajado con una tarjeta SD externa, ya que necesitábamos leer un archivo .wav o un .mp3 y reenviar la señal de audio digital al MAX98357. Para ello hemos modificado el montaje del apartado anterior, donde hemos añadido un lector de tarjetas SD y modificando un segundo código otorgado en la práctica hemos conseguido una realización óptima de este apartado. 
En el código, una vez declaradas las librerías necesarias, se hace la declaración de los pines (en este caso, GPIO10, GPIO11, GPIO12 y GPIO13 harán la comunicación SPI para la tarjeta SD mientras que los pines GPIO4, GPIO5 y GPIO6 harán la comunicación I2S). Una vez declarados, el código configura el  pin CS (GPIO10) de la SD, inicia el bus SPI, inicia la comunicación série, monta el sistema de archivos SD, establece los pines del bus I2S para la salida del audio, y finalmente ajusta el volumen y carga el archivo desde la SD para su reproducción.


El código es el siguiente:
```
#include "Audio.h"
#include "SD.h"
#include "FS.h"

// Definir los pines para ESP32-S3
#define SD_CS         10  
#define SPI_MOSI      11  
#define SPI_MISO      13  
#define SPI_SCK       12  
#define I2S_DOUT      6   
#define I2S_BCLK      5   
#define I2S_LRC       4   

Audio audio; 

void setup(){ 
  pinMode(SD_CS, OUTPUT); 
  digitalWrite(SD_CS, HIGH); 
  SPI.begin(SPI_SCK, SPI_MISO, SPI_MOSI); 
  Serial.begin(115200); 
  SD.begin(SD_CS); 
  audio.setPinout(I2S_BCLK, I2S_LRC, I2S_DOUT); 
  audio.setVolume(10); // 0...21 

  audio.connecttoFS(SD, "patito_juan_short.wav"); 
} 
 
void loop(){ 
    audio.loop(); 
} 
 
// optional 
void audio_info(const char *info){ 
    Serial.print("info        "); Serial.println(info); 
} 
void audio_id3data(const char *info){  //id3 metadata 
    Serial.print("id3data     ");Serial.println(info); 
} 
void audio_eof_mp3(const char *info){  //end of file 
    Serial.print("eof_mp3     ");Serial.println(info); 
} 
void audio_showstation(const char *info){ 
    Serial.print("station     ");Serial.println(info); 
} 
void audio_showstreaminfo(const char *info){ 
    Serial.print("streaminfo  ");Serial.println(info); 
} 
void audio_showstreamtitle(const char *info){ 
    Serial.print("streamtitle ");Serial.println(info); 
} 
void audio_bitrate(const char *info){ 
    Serial.print("bitrate     ");Serial.println(info); 
} 
void audio_commercial(const char *info){  //duration in sec 
    Serial.print("commercial  ");Serial.println(info); 
} 
void audio_icyurl(const char *info){  //homepage 
    Serial.print("icyurl      ");Serial.println(info); 
} 
void audio_lasthost(const char *info){  //stream URL played 
    Serial.print("lasthost    ");Serial.println(info); 
} 
void audio_eof_speech(const char *info){ 
    Serial.print("eof_speech  ");Serial.println(info); 
} 
```
