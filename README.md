# Protokoll 6 <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/30/HTL_Kaindorf_Logo.svg/300px-HTL_Kaindorf_Logo.svg.png" alt="">  
  
Professor: SX  
Übungsort: Kaindorf   
Übungsdatum: 18.12.2018  
Anwesend: Vollmaier Alois, Vezonik Sarah, Wegl Patrick, Wesonig Mercedes, Winter Matthias, Winter Thomas

## Aufgabenstellung  
Das Ziel war es, einen Modbusfähigen Temperatursensor einzurichten, welcher dann auf dem PC im Therminal ausgelesen wird.
  
## Grundlagen zu Modbus  
Das Modbus-Protokoll wurde 1979 zur Komunizierung mit SPS Geräten entwickelt. In der Automatisierungstechnik wird dieses Protokoll sehr gerne verwendet da es sich um einen offenen Standard handelt, welcher lösungen mit RS-232, RS-485 Busse und TCP/IP-Netzwerke Ermöglicht. Weiters ist Modbus ein zustandsloses Protokoll.  
  
Der Kommunikationsablauf beruht auf einem Server/Client Prinzip. Der Client (zum Beispiel ein PC) sendet einen Request zum Server (zum Beispiel ein Aktor oder Sensor). Dieser antworter mit einer Response.  
<img src="https://raw.githubusercontent.com/winthm14/Protokoll-5/master/Server%3AClient.tif" alt="">  
  
Bei der eigentlichen Datenübertragung werden drei Varianten unterschieden:  
 * **Modbus ASCII**  
    - Rein Textuelle byteweise Übertragung von Daten. Frames beginnen mit einem Doppelpunkt.  
 * **Modbus RTU**  
    - Binäre byteweise Übertragung von Daten.  
 * **Modbus TCP**  
    - Übertragung der Daten in TCP Paketen
### Modbus-Gateway  
Ein Modbus-Gateway ist in der Lage verschiedene Modbus-Varianten miteinander zu verbinden, also zum Beispiel die Verbindung eines über die UART-Schnittstelle erreichbaren Sensors mit einem über TCP/IP erreichbaren PC.  
Das Modbus Application Layer Protocol definiert dabei als Frame sogenannte **Protocol Data Units (PDU)**. Diese enthalten noch kein Adressierungsschema, da unterschiedliche Varianten (UART, TCP, ...) auch unterschiedliche Adressierungsarten verwenden.  
Die zusätzlichen Spezifikationen für die jeweiligen Varianten definieren dann auch zusätzliche Frame-Felder für die Adressierung und Fehlererkennung, wodurch dann die **Application Data Unit (ADU)** entsteht.  
  
<img src="https://www.researchgate.net/profile/Naixue_Xiong3/publication/281692567/figure/fig2/AS:331936288526339@1456151186841/MODBUS-Protocol-PDU-and-ADU.png" alt="">   
  
### Daten-Modell  
Das Modbus Daten-Modell unterscheidet vier Tabellen (Adressräume) für:  
* **Discrete Inputs**: Einzelnes Bit, kann nur gelesen werden. **Beispiele**: Taster, Endschalter,...  
* **Coils**: Einzelnes Bit, kann geleschen und beschrieben werden. **Beispiele**: LED, Relais,...  
* **Input Registers**: 16-Bit Wert, kann nur gelesen werden. **Beispiele**: Temperatursensor, ADC,...  
* **Hold-Registers**: 16-Bit Wert, kann beschrieben und gelesen werden. **Beispiele**: PWM-Einheit, DAC,...  
  
## Lösungsansätze  
### Modbus-TCP  
Hiermit wäre es möglich einen kabellosen Temperatursensor über W-LAN ein zurichten. Hierfür würde die Möglichkeit bestehen einen kleinen Einplatienencomputer wie zum Beispiel den Raspberry Pi Zero als Modbus Gateway zu verwenden. Zusätzlich könnte man zum Auswerten eine Android App schreiben und die Messwerte Grafisch ausgeben lassen.  
### Modbus-ASCII  
Die zweite Lösung wäre, einen Kabelgebundenen Temperatur Sensor einzurichten welcher mit dem Modbus-ASCII Protokoll arbeitet. Hierfür könnte man einen UART zu RS485 Umsetzer nach dem Arduino Nano setzen und diesen dann dierekt an eine SPS anschließen. In diesem Fall wäre der Arduino der Client und die SPS der Server.  
Diese Lösung wird auch in einer etwas abgeänderten Form in unserer Laboreinheit verwendet. Hierfür werden die UART Signale nicht auf RS485 übersetzt sondern mit dem UART auf USB converter umgesetzt um die Messdaten direkt am PC im Therminal ausgeben zu lassen.  
<img src="https://user-images.githubusercontent.com/43165765/55355146-cd3c0a80-54c7-11e9-881b-f95c7439baf9.png" alt="">  
  
## Datenanfrage (Request)  
Dieses nachfolgende Datenframe wurde im Unterricht verwendet um die Request zum Auslesen des Sensors zu senden. 
```
:010400000001 _ _ <CR><LF>
```  

**Beschreibung des Modbus ASCII Frame's**  
  
:|01|04|0000|0001|_ _|<CR><LF>
  
```:``` -> Start Frame  
```01``` -> Adresse des Geräts am Bus  
```04``` -> Read Input Register  
```0000``` -> Inputregister 1 für die Temperatur  
```0001``` -> Anzahl der Gewählten Input Register  
```_ _``` -> LRC/Prüfsumme  
```<CR><LF>``` -> End-Frame   
  
## Antwort (Response)  
Dieses nachfolgende Datenframe wurde im Unterricht verwendet um die Response vom Microcontroller zum PC zu senden
```
:010402xxxx _ _ <CR><LF>
```  

**Beschreibung des Modbus ASCII Frame's**  
  
:|01|04|02|xxxx|_ _|<CR><LF>
  
```:``` -> Start Frame  
```01``` -> Adresse des Geräts am Bus  
```04``` -> Read Input Register  
```02``` -> Anzahl der Bytes  
```xxxx``` -> Temperaturwert  
```_ _``` -> LRC/Prüfsumme  
```<CR><LF>``` -> End-Frame  
  
## Übertragung von den Temperaturwerten  
Die Temperaturwerte werden als 16Bit Werte übertragen. Weiters werden die werte in Festkommacodierung übertragen somit sind links und rechts vom Komma 8 Bit.
Um nun vom Temperaturwert z.B 23,5°C zum hex Wert zu kommen muss man den wert zuerst mit 256 Multiplizieren und danach in eine Hexadezimalzahl umwandeln -> 23,5 * 256 = 6016 => 1780hex  
Somit würde die Response wie folgt aussehen:

```
:0104021780 _ _ <CR><LF>
```   
  
## Der ADC
Zum Messen von Spannungen werden sogenannte ADC's(Analog Digital Converter) verwendet. Im falle des Atmega328P Mikroprozessor wird die Methode der successiven approximation (schrittweise Annäherung) verwendet. Die Auflößung dieses ADC's ist 10Bit wobei die letzten 2 Bits nicht brauchbar sind da diese durch Rauschen (elektromagnetische Einflüsse aus der Umwelt) stark verfälscht werden. Die Frequenz mit der der ADC betrieben wird sollte zwischen 50kHz und 200kHz liegen.  
Zum messen wird immer eine Referenzspannung benötigt. Hierbei ist zu beachten dass die zu messende größe nicht größer als die Referenzspannung sein darf.  
Mögliche Referenzspannungen: 
* **Bandgapspannung (Interne Referenz, sehr genau)**  
* **Versorgungsspannung**  
* **Externe Referenzspannung an einem Pin**  

Der ADC-Wert wird in zwei 8-Bit Register Gespeichert. Das erste Register heißt ADCH für high und das zweite heißt ADCL für low.  
Das Ergebniss kann nach links und nach rechts geschoben werden. In unserem Fall ist es sinnvoll den Wert nach links zu schieben da man die Letzten beiden Bits onehin ungenau sind und somit vernachlessigt werden können. Somit genügt es wenn nur der ADCH-Wert ausgelesen wird.  
  
## Auswertung des ADC Werts  
  
Um nun einen Brauchbaren Temperaturwert zu bekommen gibt es eine Formel welche wie folgt Lautet:   
**ADC=(V_in*1024)/(V_ref)**  
**ADC** ist in diesem Fall der ADC wert aus dem ADCH Register, **1024** ist die Auflößung des ADC's und **V_ref** ist die Referenzspannug. In unserem Fall die 1,1V Bandgapspannung des Mikrocontrollers.
Somit kommen wir bei einer Temperatur von 25°C auf einen ADC Wert von 73hex.







