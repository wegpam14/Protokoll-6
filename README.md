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
Die zweite Lösung wäre, einen Kabelgebundenen Temperatur Sensor einzurichten welcher mit dem Modbus-ASCII Protokoll arbeitet. Hierfür könnte man einen UART zu RS485 Umsetzer nach dem Arduino Nano setzen und dies dann dierekt an eine SPS anschließen. In diesem Fall wäre der Arduino der Client und die SPS der Server.  
Dieese Lösung wird auch in einer etwas abgeänderten Form in unserer Laboreinheit verwendet. Hierfür werden die UART Signale nicht auf RS485 übersetzt sondern mit dem UART auf USB converter umgesetzt um die Messdaten direkt am PC im Therminal ausgeben zu lassen.
