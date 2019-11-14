# RpiNtripBase

Dies ist eine einfach zu konfigurierende RTK-Basisstation für Linux. Verschiedene GPS-Empfänger können angeschlossen werden,
normalerweise über USB, direkt an der seriellen Schnittstellen geht aber auch.

# Features
* NTRIP Caster auf Port 2101
* Proxy für die serielle Schnittstelle auf dem Port 2102; die Basis kann auch nach der 
  Installation mit dem uCenter konfiguriert und geupdatet werden
* Alles über systemd-services geregelt; werden automatisch gestartet, haben in eingebautes logging und starten sich bei Absturz selber neu
* Baudrate einfach änderbar
* Läuft grundsätzlich überall, wo ein Linux läuft, `systemd` verwendet wird und man `root`-Zugriff hat.

# Voraussetzungen

## Raspberry Pi
* Der RPI ist mit Raspbian geflasht, alle Passwörter sind geändert und alle Netzwerke eingerichtet
* Ein Zugriff mit `ssh` (oder Putty auf Windows) besteht (Datei `ssh` auf der Bootpartition anlegen)
* Der F9P ist als Basis konfiguriert; es sollten nur RTCM-Nachrichten ausgegeben werden.

## Andere Systeme
* Raspberry Pi Klon: Image vom Hersteller verwenden, das sollte grundsätzlich funktionieren, solange eine ausreichend moderne Version verfügbar ist. 
* Andere Linux-Systeme (wie Router, Desktop, NAS): sicherstellen, dass `systemd` verwendet wird und man `root`-Zugriff hat. Ich gehe davon aus, 
  dass wenn jemand die Basis auf so einem System aufbaut, er genug Erfahrung hat, um mit den Informationen hier zurechtzukommen. 

## Allgemein
* **Der Benutzer hat alles durchgelesen und versteht was er macht (oder macht nur das, was hier beschrieben ist...)**
* **Ein Fork ist nicht nötig, braucht nur Ressourcen auf github. Es gibt für das das kleine Sternchen, damit ist auch 
  sichergestellt, dass man das Repository wieder findet.**


# Bekannte Unzulänglichkeiten

* Nur die Basis mit einem F9P und M8T implementiert;
* Die grundsätzliche Struktur lässt alle Varianten von Empfängern zu, mangels Hardware kann ich aber nicht alle testen. Wenn jemand
  Änderungen macht, kann er mir einen Pull Request schicken, ich nehme die Commits gerne in das Repository auf.

# Struktur, Funktionsweise

Der baseProxy-Service stellt ein TCP-Socket zur Verfügung, welches eine direkte Verbindung zum seriellen Anschluss
darstellt. Auf diesen Port kann sowohl mit u-center wie auch mit dem nächstem Service, nämlich mit str2str zugegriffen
werden. Dieser Service leitet die Daten weiter (mit eventueller Konvertierung) an den ntripcaster-Service, welcher den
Ntripcaster auf Port 2101 bereitstellt. 

# Mitwirkmöglichkeiten

Es gibt grundsätzlich verschiedene Varianten, von gut nach schlecht geordnet:

1. Ein Fork machen (github-Account notwendig, Anleitungen findet man im Internet), die entsprechenden Änderungen machen, 
   das Ganze mit einer **sinnvollen** Beschreibung committen und mir ein Pullrequest schicken. Diese Variante bietet sich an, 
   wenn man eine genaue Lösung für das Problem hat
1. Mich auf den Telegramm**gruppen** rund um Cerea/AgOpenGPS anschreiben, am Besten in der F9P-Gruppe
   (https://cerea-forum.de/forum/index.php?thread/427-links-zu-messenger-gruppen/). Diese Lösung ist für diejenigen,
   welche keine Erfahrung mit Linux, systemd und ähnlichem haben und das Problem mit der Community zusammen lösen wollen.
1. Ein "Issue" erröffnen. Dazu muss ebenfalls ein github-Account erstellt werden. **Sinnvoll**, und mit **genügend Informationen**
   die gewünschten Änderungen beschreiben, am besten mit Code oder funktionierenden Beispielen. Diese Lösung bietet sich an,
   wenn man zwar weiss, was man machen will, aber sich nicht sicher ist, wie es zu implementieren ist. 
1. Keine gute Idee ist es, mich persönlich anzuschreiben. Die Wahrscheinlichkeit ist sehr gross, dass jemand das gleiche Problem 
   schon einmal gehabt hat und weiter weiss. Ich kann es auch nicht alles testen, wenn ich die Hardware nicht habe, da ist es 
   ebenfalls sinnvol, wenn mehrere Leute zusammenarbeiten. 

Dies sollte ein Gemeinschaftsprojekt sein. Also alle Erfolge und Änderungen öffentlich machen, so dass andere auch davon 
profitieren können. 

# Anpassungen/Weiterentwicklung

Die grundsätzliche Struktur soll so sein, dass dieses Projekt auf jedem Linux läuft. Zu diesem Zweck sind alle externen Projekte
als Source Code eingebunden und werden auf der Maschine selbst compiliert. Weiter werden die einzelnen Services in eigenen Dateien
definiert, dem System hinzugefügt, gestartet und aktiviert (damit sie beim Starten des Systems ebenfalls gestartet werden). Wenn 
neue Empfänger hinzugefügt werden, also den entsprechenden Service kopieren und abändern. Somit kann der Benutzer selbst wählen,
welcher Art sein Empfänger ist und welcher Service er starten will.

# Installation

Um das Ganze zu installieren, wird zuerst ein normales Raspbian-Image (ohne Desktop; Stretch Lite reicht) auf eine SD-Karte
gebrannt und konfiguriert. Anleitungen dazu findet man genug auf dem Internet, 
z.B. [https://howtoraspberrypi.com/how-to-raspberry-pi-headless-setup/] oder [https://www.dahlen.org/2017/10/raspberry-pi-zero-w-headless-setup/]

Dann wird auf dem Pi per `ssh` (oder Putty)  eingelogt und Git und Socat installiert:
```
sudo apt-get install git socat
```

Dieses Repository wird geklont:
```
git clone https://github.com/eringerli/RpiNtripBase.git
```

Dann wird die Datei `compile.sh` und `update.sh` als `root` ausgeführt:
```
cd RpiNtripBase
sudo ./compile.sh
sudo ./update.sh
```

Jetzt müssen noch die entsprechenden Services gestartet und aktiviert werden:
```
sudo systemctl enable baseProxy@115200.service
sudo systemctl start baseProxy@115200.service
sudo systemctl enable ntripcaster.service
sudo systemctl start ntripcaster.service
sudo systemctl enable logrotate-ntripcaster.timer
sudo systemctl start logrotate-ntripcaster.timer
sudo systemctl enable str2str.service
sudo systemctl start str2str.service
```
Wenn ein M8T angeschlossen wird, muss statt `str2str.service` `str2str-M8T.service` ausgeführt werden. Weiter muss die Position der Basis in der Datei
`str2str-M8T.service` angepasst werden (siehe [Konfiguration](#konfiguration)). Wenn ein öffentlicher Caster verwendet wird, bitte unter dem Punkt
[Anderer NTRIP Caster](#anderer-ntrip-caster) schauen.

Wenn der GPS-Empfänger über eine serielle Schnittstelle angeschlossen wurde (z.B. M8T mit TTL-Serial-zu-USB-Wandler), muss die Baudrate
korrekt konfiguriert sein (zwei mal!). Wenn er direkt über USB verbunden ist, ist es egal (z.B. F9P über USB). Hier hilft ausprobieren:
im u-center kann eine Text-Anzeige geöffnet werden (View -> Text Console), dann sieht man es sofort.

Um die Basis erreichen zu können, muss auf dem Router eine Portweiterleitung auf den eingestellten
Port des NTRIP-Casters und eine DynDNS-Adresse (oder ähnlich, gibt viele Anbieter) eingerichtet werden,
sodass ein Zugriff vom Rover übers Internet möglich wird. FritzBox-Besitzer können auch eine MyFritz-Addresse
verwenden. Der Port 2102 erlaubt einen direkten Zugang zum GPS-Empfänger, diesen **nicht** öffentlich zugänglich machen.

# Konfiguration
Dateien werden am einfachsten mit dem Programm `nano` bearbeitet, dieses wird mit `nano DATEI` gestartet. Um die Datei zu speichern und `nano` zu beenden, wird in `nano` `Ctrl+X` gedrückt und die anschliessende Frage mit `Y` beantwortet.

Grundsätzlich ist es einfacher, Schritt für Schritt Sachen zu verändern und diese jeweils zu testen, als alles auf einmal zu 
konfigurieren.  Wenn man bedenkt, dass dieses System dann ohne weitere Wartung lange Zeit von alleine läuft, sind diese paar 
Minuten gut investiert.

Die Konfiguration des NTRIP Casters wird in der Datei `ntripcaster.conf` gemacht. Standartmässig wird der
Caster auf dem Port 2101 gestartet, der Mountpoint ist "STALL", der Benutzername und
das Passwort je "gps". Wenn der Mountpoint verändert wird, muss er in der Datei 
str2str.service ebenfalls angepasst werden. **Achtung: das Passwort für NTRIP wird im Klartext (HTTP Basic Auth)
über das Internet übertragen. Also etwas nie eines wählen, das schon an anderen Orten verwendet wird, da es extrem einfach ist, dieses abzuhören!**

**Wenn Änderungen gemacht werden, muss das Kommando `sudo ./update.sh` neu  ausgeführt werden, um sie zu übernehmen.**

Wenn neu eingelogt wird, muss zuerst in den Ordner gewechselt werden, wohin `RpiNtripBase` heruntergeladen wurde. Dies geschieht mit 
```
cd RpiNtripBase
```

Wenn eigene Dateien/Services erstellt werden, müssen diese nach `sudo ./update.sh` manuell neu gestartet werden. Die schon vorgegebenen werden in `./update.sh` automatisch neu gestartet, vorausgesetzt, sie sind schon gestartet. Dies geschieht mit
```
sudo systemctl restart DATEI.service
```

# Fehlersuche

Um mal zu schauen, was auf der Schnittstelle vom GPS-Empfänger ankommt, kann folgendes Kommando verwendet werden:
```
/usr/bin/socat TCP:localhost:2102 -
```

Beendet wird es mit `Ctrl+C`. Dazu muss der Service `baseProxy@.service` laufen.

# Logging

Das Logging funktioniert über journald. Wenn alle Ausgaben der Programme live angezeigt werden sollen, muss 
```
journalctl -f -u baseProxy@115200.service -u ntripcaster.service -u str2str.service -u str2str-M8T.service
```
ausgeführt werden. Der Status der Services kann mit `systemctl status SERVICE` angezeigt werden. Damit werden neben dem Zustand und 
eventuellen Startschwierigkeiten auch die paar letzten Zeilen Ausgabe mit dem Zeitstempel angezeigt.

Weiter ist der ntripcaster so konfiguriert, dass er in die Datei `/tmp/ntripcaster.log` schreibt. Um diese Datei komfortabel anzuzeigen,
muss `less /tmp/ntripcaster.log` ausgeführt werden. Das zeigt eine scrollbare Ansicht der Datei an. Mit Eingabe von `End` (die Taste) wird
an das Ende gesprungen, mit `Shift+F` in Echtzeit aktualisiert. Um diesen Modus zu verlassen, wird `Ctrl+C` eingegeben. Um das Programm zu
verlassen, wird `Q` gedrückt.

Debian Stretch Lite ist so konfiguriert, dass `journald` nicht auf die SD-Karte schreibt, um diese zu schonen. Wenn es anders gewünscht wird
(nicht zu empfehlen), kann das natürlich geändert werden. Anleitungen findet man im Internet (nach `journald persistent storage` suchen).
Auch der Speicherort von `ntripcaster.log` ist so gewählt, dass keine Schreiboperationen auf die SD-Karte ausgelöst werden. Mittels dem 
`logrotate-ntripcaster.service` wird die Grösse auf max 10MB begrenzt und die Anzahl Dateien auf drei. Somit ist mit der RTK-Basis ein
wartungsfreier Dauereinsatz möglich. Wenn ein persistentes Logging gewünscht wird, kann der Speicherort auf `/var/log/` geändert werden.
Auch können alle Parameter der Rotation in `ntripcaster.logrotate` eingestellt werden.

Ein Nachteil gibt es an dieser Konfiguration: die Logdateien und Einträge werden bei jedem Herunterfahren gelöscht. Sollte aber kein
Problem darstellen, da die Basis nie automatisch neu startet oder ähnliches. Falls eine Boot-Schleife entsteht, hat es ziemlich sicher
nichts mit den Services hier hier zu tun.

# Andere Baud-Raten
Wenn der F9P per USB angeschlossen wird, wird die Baudrate nicht verwendet und kann auf dem Standart belassen werden.
Wenn ein USB-RS232-Wandler und ein anderer GPS-Empfänger verwendet wird, eventuell schon.

Es ist sehr wichtig, dass der alte Service gestoppt und deaktiviert wird, dies geschieht mit:
```
sudo systemctl stop baseProxy@AlteBaudrate.service
sudo systemctl disable baseProxy@AlteBaudrate.service
```

Um den neuen Service zu starten und aktivieren, wird folgendes eingegeben:
```
sudo systemctl start baseProxy@NeueBaudrate.service
sudo systemctl enable baseProxy@NeueBaudrate.service
```

Falls man nicht mehr weiss, wie man es konfiguriert hat, kann mit `systemctl` alle laufenden Services angezeigt werden.
Lieber einmal zuviel `systemctl disable ...` eingeben, als man hat nach dem Reboot zwei baseProxy-Services gleichtzeitig
am laufen. Diese konkurieren um die serielle Schnittstelle, was nicht funktionieren kann.

# Konfiguration mit uCenter

Um die Basis mit dem uCenter zu konfigurieren, wird zuerst `str2str.service` abgeschalten:
```
sudo systemctl stop str2str.service
```

Dann wird im uCenter eine neue Verbindung per TCP hergestellt, also mit `tcp://IP-BASIS:2102`. Die Basis kann nun normal konfiguriert werden.
Um die Basis wieder per NTRIP erreichbar zu machen, muss `str2str.service` wieder gestartet werden:
```
sudo systemctl start str2str.service
```

Es sollte auch ohne abschalten von `str2str.service` funktionieren (**auf eigene Gefahr!**) , dann werden aber alle Daten über NTRIP gespiegelt, was nicht so intelligent ist, 
vor allem, wenn der Empfänger geupdatet wird (der Rover wird gerade mitkonfiguriert, kein erwünschtes Verhalten). Ebenfalls sollte die Verbindung von uCenter wieder getrennt werden, da der Empfänger bei Verbindung allerlei zusätzliche Informationen sendet, 
die das Datenvolumen auf dem Rover schnell ansteigen lassen. Aber das sollte kein grosses Problem darstellen, wenn sichergestellt ist, dass keine Clients 
am Ntrip-Caster hängen.

Falls die Baudrate geändert wird , muss wie oben beschrieben der Service `baseProxy` mit der
neuen Baudrate gestartet werden. Wenn es nur temporär ist, müssen die Kommandos mit `systemctl enable ...` und `systemctl disable ...`
nicht eingegeben werden.

# RTCM 1008

Wenn bestimmte Empfänger verwendet werden (z.B. Trimble), müssen leere RTCM-1008-Nachrichten in den Datenstrom eingefügt werden, falls
diese nicht vom GPS-Empfänger selbst erstellt werden. Wenn dies gewünscht ist, muss anstatt von ```str2str.service```
```str2str-injectrtcm1008.service``` ausgeführt und aktiviert werden. Der Rest bleibt gleich. Nachzulesen unter diesem
[Link](https://www.thecombineforum.com/forums/31-technology/331721-how-use-zed-f9p-base-station-trimble.html)

# Anderer NTRIP Caster

Falls ein anderer NTRIP Caster als der lokale verwendet werden will (z. B. rtk2go oder andere Caster im Internet), muss
die Datei `str2str-remoteCaster.service` mit anderen Login-Daten bestückt werden. Der Service `ntripcaster.service` muss dann
nicht gestartet und aktiviert werden.

Es braucht keine Portweiterleitung und DynDNS-Adresse mehr. Das würde demnach auch mit Internetanbindungen ohne öffentliche IP
(wie manche LTE-Router bzw -Modems anbieten) funktionieren. 

Der Service `str2str-remoteCaster.service` ist in der Grundkonfiguration für RTK2Go ausgelegt. Es ist so aufgebaut, dass er den
lokalen Caster nicht automatisch mitstartet. Um den Service zu verwenden, müssen die Login-Daten und je nach dem der Host geändert werden.

Dann kann das Ganze gestartet werden:
```
sudo ./update.sh
systemctl enable str2str-remoteCaster.service
systemctl start str2str-remoteCaster.service
```

Nach dem wird der Service automatisch neu gestartet, wenn die Daten in `str2str-rtk2go.service` angepasst werden und `sudo ./update.sh`
ausgeführt wird. Dieser Service kann parallel zum lokalen Caster laufen, oder dieser und der dazugehörige `str2str.service` kann mit
```
sudo systemctl stop ntripcaster.service
sudo systemctl disable ntripcaster.service
sudo systemctl stop str2str.service
sudo systemctl disable str2str.service
```
gestoppt und deaktiviert werden.

# Trinkgeld

Ich habe diese Software hauptsächlich für meine eigene Basis geschrieben, da mir die bestehenden Lösungen nicht zugesagt haben. 

Wem sie gefällt und Lust dazu hat, kann mir ein kleines Trinkgeld überweisen.

[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://paypal.me/eringerli)
