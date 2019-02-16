# calcardbackup

[go to english version of this  README...](README.md)

Dieses Bash-Skript exportiert Kalender und Adressbücher aus ownCloud/Nextcloud als .ics- und .vcf-Dateien und speichert sie in einem komprimierten Archiv. Weitere Optionen stehen zur Verfügung.

__WICHTIG__: Ab Version 0.8.0 ist keine Datei mit Benutzernamen und Passwörtern mehr notwendig, da alle benötigten Daten direkt aus der Datenbank gezogen werden.  
Falls Kalender/Adressbücher von ausgewählten Benutzern gesichert werden sollen, können diese ohne Passwörter in `users.txt` gelistet werden.

__Allen, die *calcardbackup* von einer früheren Version auf 0.8.0 aktualisieren, wird nachdrücklich empfohlen, die Datei mit den Passwörtern zu löschen, oder zumindest die Passwörter daraus zu entfernen.__

## Inhalt
- [Voraussetzungen](#voraussetzungen)
- [Schnellinstallation](#schnellinstallation)
- [Optionen](#optionen)
- [Beispiele](#beispiele)
- [Nextcloud-Snap Benutzer](#nextcloud-snap-benutzer)
- [Erwähnenswertes zur Verschlüsselungsoption](#erwähnenswertes-zur-verschlüsselungsoption)
- [Funktioniert das auch mit einer nicht funktionierenden ownCloud/Nextcloud Installation?](#funktioniert-das-auch-mit-einer-nicht-funktionierenden-owncloudnextcloud-installation)
- [Über die Option -g|--get-via-http](#über-die-option--g----get-via-http)
- [Über die Option -i|--include-shares](#über-die-option--i----include-shares)

## Voraussetzungen
- lokale Installation von ownCloud/Nextcloud >= 5.0 mit MySQL/MariaDB, PostgreSQL oder SQLite3
- der das Skript startende User muss Leserechte für das Skript selbst, die `config.php` der ownCloud/Nextcloud-Installation und alle Konfigurationsdateien haben
- *optional*: das Paket `zip`, um Backups als zip-Datei zu komprimieren (anstelle tar.gz)
- *optional*: das Paket `curl`, wenn die nicht empfohlene Option `-g|--get-via-http` gesetzt wird

## Schnellinstallation
1. das Repository auf deinen Server klonen (nicht ins webroot!) und ins Verzeichnis wechseln:  
`cd /usr/local/bin`  
`git clone https://github.com/BernieO/calcardbackup`  
`cd calcardbackup`

2. die Besitzrechte des Repository dem Webserver-User zuweisen (hier `www-data`):  
`chown -R www-data:www-data .`

3. das Skript als Webserver-User aufrufen und als Argument den Pfad zu ownCloud/Nextcloud übergeben (hier `/var/www/nextcloud`):  
`sudo -u www-data ./calcardbackup "/var/www/nextcloud"`

4. die Ausgabe des Skripts beobachten. Falls weitere Optionen benötigt werden, wird dies ausgegeben.

5. Das Backup befindet sich im Verzeichnis `backups/`.

Es gibt viele weitere Optionen, die dem Skript übergeben werden können (siehe [Optionen](#optionen) und [Beispiele](#beispiele)).

## Optionen
Alle Optionen können als Konfigurationsdatei oder über die Kommandozeile übergeben werden. Ohne Optionen, oder nur mit Option `-b|--batch` aufgerufen, benutzt das Skript die Datei `calcardbackup.conf` im Skriptverzeichnis als Konfigurationsdatei, sofern vorhanden.  
Falls keine Konfigurationsdatei über die Option `-c|--configfile` an das Skript übergeben wird, muss der Pfad zur ownCloud/Nextcloud Instanz als erstes Argument übergeben werden.  
Im Folgenden werden die verfügbaren Optionen ausführlich beschrieben:

```
Aufruf: ./calcardbackup [VERZEICHNIS] [Option [Argument]] [Option [Argument]] [Option [Argument]] ...

Argumente in Großbuchstaben sind für die jeweiligen Optionen notwendig.
Pfade (DATEI / VERZEICHNIS) sind absolute Pfade oder relative Pfade zum Arbeitsverzeichnis.

-a | --address URL
       übergibt die URL der ownCloud Installation an das Skript.
       nur erforderlich für Option '-g|--get-via-http' in Kombination mit ownCloud < 7.0
-b | --batch
       Batchmodus: außer des Pfades zum Backup erfolgt keine Ausgabe.
       Abhängig von der Konfiguration ist das:
          - absoluter Pfad zum komprimierten Backup-Archiv
       oder, falls mit Option '-x|--uncompressed' aufgerufen,
          - absoluter Pfad zum Verzeichnis mit den unkomprimierten Backupdateien
-c | --configfile DATEI
       Benutze DATEI als Konfigurationsdatei. Siehe 'examples/calcardbackup.conf.example'
       Weitere angegebene Optionen mit Ausnahme von '-b|--batch' werden ignoriert!
-d | --date FORMAT
       Benutze FORMAT als Dateinamenserweiterung für die komprimierte Backupdatei oder das
       Backupverzeichnis. FORMAT muss eine gültige Formatbeschreibung des Befehls date() sein.
       Als Standard wird -%Y-%m-%d benutzt, was zu folgender Datei- oder Verzeichnisbenennung führt:
       'calcardbackup-2017-03-23.tar.gz' oder 'calcardbackup-2017-03-23'
       Informationen über verschiedene Formate und die Syntax finden sich unter 'man date'
-e | --encrypt DATEI
       Verschlüssele das komprimierte Backup mit AES256 (gnupg).
       Die erste Zeile von DATEI wird als Passphrase benutzt werden.
-g | --get-via-http
       ACHTUNG: diese Option ist veraltet. Sie ist nur vorhanden, um Rückwärtskompatibilität
       zu gewährleisten.
       Fordere Kalender-/Addressbuchdateien über http(s) vom ownCloud/Nextcloud Server an.
       Wenn diese Option benutzt wird, ist eine Datei mit Nutzernamen und dazugehörigen
       Klartext-Passwörtern zwingend notwendig (siehe Option '-u|--usersfile')
       Dies war die Standardvorgehensweise des Skripts bis calcardbackup <= 0.7.2. Es wird davon
       abgeraten diese Option zu benutzen wegen des Sicherheitsrisikos, Passwörter im Klartext in
       einer eigenen Datei anzugeben.
-h | --help
       Gib Versionsnummer und einen kurzen Hilfetext aus
-i | --include-shares
       Sichere auch geteilte Adressbücher und Kalender. Elemente werden nur einmal gesichert: z.B. ein
       geteilter Kalender wird nicht erneut gesichert, wenn derselbe Kalender schon für einen anderen
       Nutzer gesichert wurde.
       ACHTUNG: diese Option wird ignoriert, wenn nicht in Kombination mit '-u|--usersfile' benutzt.
-ltm | --like-time-machine N
       Behalte alle Sicherungen der letzten N Tage, für die Zeit davor aber nur solche, die Montags
       erstellt wurden.
-na | --no-addressbooks
       Sichere keine Adressbücher
-nc | --no-calendars
       Sichere keine Kalender
-o | --output VERZEICHNIS
       Benutze VERZEICHNIS, um die Backups zu speichern.
       Fehlt diese Option, wird das Verzeichnis 'backups/' im Skriptverzeichnis erstellt und benutzt.
-p | --snap
       Diese Option ist verpflichtend bei nextcloud-snap (https://github.com/nextcloud/nextcloud-snap).
       calcardbackup muss in diesem Fall mit sudo gestartet werden (ausgeführt als root ohne sudo wird
       auch scheitern!).
-r | --remove N
       Lösche Backups älter als N Tage vom Backupverzeichnis (N muss eine positive Ganzzahl sein).
-s | --selfsigned
       Ignoriere ein nicht vertrauenswürdiges (z.B. selbstsigniertes) Zertifikat. Erforderlich in
       Kombination mit Option '-g' und nicht vertrauenswürdigen Zertifikaten. In jedem Fall wird cURL
       benutzt, um status.php abzurufen und damit zusätzliche Prüfungen durchzuführen.
       Falls cURL die URL aufgrund eines nicht vertrauenswürdigen Zertifikates nicht
       erreichen kann, wird calcardbackup diese zusätzlichen Prüfungen überspringen.
-u | —usersfile DATEI
       Benutze DATEI, die die Nutzernamen der Nutzer enthält, deren Daten gesichert werden sollen.
       Ein Nutzer pro Zeile. Siehe 'examples/users.txt.example'
       Da die Option '-f|--fetch-from-database' das Standardverhalten von calcardbackup >= 0.8.0 ist,
       besteht keine Notwendigkeit mehr in dieser Datei Passwörter anzugeben oder diese Datei überhaupt
       zu benutzen.
       ACHTUNG: diese Option ist zwingend erforderlich, falls das Skript mit '-g|--get-via-http'
       aufgerufen wird.
-x | --uncompressed
       komprimiere das Backup nicht
-z | --zip
       Benutze zip, um das Backup zu komprimieren, an Stelle eines gzip-tar-Archivs (tar.gz)=

ACHTUNG: die Option '-f|--fetch-from-database' (eingeführt mit calcardbackup 0.6.0) ist
         der Standard für calcardbackup >= 0.8.0 und hat daher keine Funktion mehr.
```

## Beispiele
1. `./calcardbackup /var/www/nextcloud -nc -x`  
Sichere keine Kalender (`-nc`) und speichere das Backup unkomprimiert (`-x`) im Verzeichnis `calcardbackup-YYYY-MM-DD` (Standard) in `./backups/` (Standard).

2. `./calcardbackup /var/www/nextcloud --no-calendars --uncompressed`  
Dies ist genau dasselbe Kommando wie im ersten Beispiel, allerdings mit langen Optionsnamen statt der kurzen.

3. `./calcardbackup -c /etc/calcardbackup.conf`  
Benutze die Konfigurationsdatei /etc/calcardbackup.conf (`-c /etc/calcardbackup.conf`). Alle Parameter für das gewünschte Verhalten müssen in dieser Datei angegeben werden (siehe [examples/calcardbackup.conf.example](examples/calcardbackup.conf.example)). Die Angabe weiterer Optionen entfällt, da sie ignoriert werden (mit Ausnahme von `-b|--batch`).

4. `./calcardbackup /var/www/nextcloud -b -d .%d.%H -z -e /home/tom/key -o /media/data/backupfolder/ -u /etc/calcardbackupusers -i -r 15`  
Unterdrücke alle Ausgaben außer dem Pfad zum Backup (`-b`), benutze die Dateinamenserweiterung .DD.HH (`-d .%d.%H`), benutze zip, um das Backup zu komprimieren (`-z`), verschlüssele das komprimierte Backup und benutze als Schlüssel die erste Zeile der Datei /home/tom/key (`-e /home/tom/key`), speichere das Backup im Verzeichnis /media/data/backupfolder/ (`-o /media/data/backupfolder/`), sichere nur Elemente von in der Datei /etc/calcardbackupusers angegebenen Nutzern (`-u /etc/calcardbackupusers`), schließe mit Nutzern geteilte Adressbücher/Kalender mit ein (`-i`) und lösche alle Sicherungen älter als 15 Tage (`-r 15`).

5. `./calcardbackup`  
Benutze die Datei calcardbackup.conf im Verzeichnis des Skripts als Konfigurationsdatei. Dies ist im Grunde dasselbe wie Beispiel Nr. 3, nur mit dem Standardpfad der Konfigurationsdatei.

6. `sudo ./calcardbackup /var/snap/nextcloud/current/nextcloud -p`  
Dieses Beispiel ist für Nutzer von [nextcloud-snap](https://github.com/nextcloud/nextcloud-snap). *calcardbackup* wird die Dienstprogramme von nextcloud-snap benutzen (`-p`), um alle in der Datenbank vorhandenen Kalender und Adressbücher zu sichern.

7. `./calcardbackup /var/www/nextcloud -g -u /etc/calcardbackupusers -s -i`  
Benutze die veraltete Methode, um Kalender- und Adressbuchdateien per http(s)-Anforderung vom ownCloud/Nextcloud Server herunterzuladen (`-g`, veraltet), finde die Nutzernamen und zugehörigen Klartext-Passwörter der zu sichernden Benutzer in der Datei /etc/calcardbackupusers (`-u /etc/calcardbackupusers`, verpflichtend bei Option -g), ignoriere ein nicht vertrauenswürdiges Zertifikat (`-s`, nur zwingend erforderlich bei Option -g) und sichere auch mit den Benutzern geteilte Kalender/Adressbücher (`-i`). Die Sicherung wird als komprimierte `*.tar.gz` Datei im Verzeichnis `./backups/` (Standard) gespeichert werden.  
__ACHTUNG__: die Option `-g` ist veraltet und es wird wegen der dabei notwendigen Datei mit Zugangsdaten der Nutzer stark davon abgeraten diese Option zu benutzen.

## Nextcloud-Snap Benutzer

Falls [Nextcloud-Snap](https://github.com/nextcloud/nextcloud-snap) benutzt wird, muss das Skript mit Option `-p|--snap` aufgerufen werden. *calcardbackup* wird dann das im Snap-Paket enthaltene Dienstprogramm `nextcloud.mysql-client` benutzen, um auf die Datenbank zuzugreifen.  
Damit dies funktioniert, muss *calcardbackup* mit `sudo` aufgerufen werden (als root ohne `sudo` aufgerufen wird auch fehlschlagen). Als Pfad zu Nextcloud muss der Pfad zu den Konfigurationsdateien des Snap Paketes angegeben werden. Bei einer Standardinstallation ist dies `/var/snap/nextcloud/current/nextcloud`. Siehe [Beispiel Nr. 6](#beispiele).

## Erwähnenswertes zur Verschlüsselungsoption

Falls Sie die Verschlüsselungsoption des Skripts benutzen möchten, seien Sie sich der folgenden Tatsachen bewusst:
- die Dateien werden von GnuGPG (AES256) mit einem Passwort verschlüsselt, das in einer separaten Datei angegeben wird
- das Passwort ist in einer Datei gespeichert. Andere Nutzer mit Zugang zum Server könnten das Passwort einsehen.
- *calcardbackup* soll ohne Benutzerinteraktion funktionieren. Daher kann es keine bombensichere Verschlüsselung anbieten. Ich betrachte die angebotene Verschlüsselungsmöglichkeit allerdings für die meisten Anwendungsfälle als ausreichend
- falls bombensichere Verschlüsselung benötigt wird, lassen Sie nicht *calcardbackup* die Sicherung verschlüsseln. Verschlüsseln Sie das Archiv stattdessen selbst.
- das Kommando zum Entschlüsseln lautet (Sie werden nach dem Passwort gefragt werden):  
`gpg -o AUSGABE_DATEI -d VERSCHLÜSSELTE_DATEI.GPG`

## Funktioniert das auch mit einer nicht funktionierenden ownCloud/Nextcloud Installation?

Ja, allerdings!  
*calcardbackup* benötigt lediglich die Datenbank (und Zugang zu ihr) einer ownCloud/Nextcloud installation, um Kalender/Adressbücher von der Datenbank auszulesen und sie als .ics und .vcf Datein zu sichern.  
Gehen Sie folgendermaßen vor:

1. eine Nextcloud Verzeichnis Attrappe anlegen:  
`mkdir -p /usr/local/bin/nextcloud-attrappe/config`

2. eine Datei `config.php` anlegen und mit folgenden Werten füllen:  
`nano /usr/local/bin/nextcloud-dummy/config/config.php`

    - den Typ der Datenbank wie in [config.sample.php](https://github.com/nextcloud/server/blob/v14.0.3/config/config.sample.php#L90-L101)

    - für MySQL/MariaDB/PostgreSQL:
      - die entsprechenden Datenbankwerte wie in [config.sample.php](https://github.com/nextcloud/server/blob/v14.0.3/config/config.sample.php#L103-L135)
    - für SQLite3:
      - den Pfad zum der in Schritt 1 angelegten Attrappe eines Nextcloud Verzeichnisses als 'datadirectory' wie in [config.sample.php](https://github.com/nextcloud/server/blob/v14.0.3/config/config.sample.php#L76-L82)
      - die SQLite3 Datenbank in die Verzeichnisattrappe kopieren (der Dateiname der SQLite3 Datenbank muss `owncloud.db` lauten):  
      `cp /path/to/owncloud.db /usr/local/bin/nextcloud-dummy/owncloud.db`

3. *calcardbackup* ausführen und als erstes Argument den Pfad zu der in Schritt 1 angelegten Nextcloud Verzeichnisattrappe angeben:
`/path/to/calcardbackup /usr/local/bin/nextcloud-dummy`

## Über die Option -g | -\-get-via-http

__ACHTUNG__: diese Option ist veraltet und es wird davon abgeraten sie zu benutzen wegen der Notwendigkeit, Klartext-Passwörter in einer eigenen Datei anzugeben.

Standardmäßig erstellt *calcardbackup* Sicherungsdateien von Kalendern und Adressbüchern durch Auslesen der entsprechenden Daten direkt von der Datenbank. Falls das Skript jedoch mit der Option `-g|--get-via-http` aufgerufen wird, wird *calcardbackup* die veraltete Methode benutzen, um die Sicherung zu erstellen und Kalender und Addressbücher vom ownCloud/Nextcloud Webinterface herunterladen. Hierfür ist eine Datei mit Nutzernamen und Passwörtern notwendig, die an das Skript mit der Option `-u|--usersfile` übergeben wird.

Neben des Sicherheitsrisikos Klartextpasswörter in einer Datei zu speichern, birgt diese Option die Gefahr von Timeouts, die dazu führen, dass *calcardbackup* große Adressbücher nicht sichern kann.

__Lange Rede kurzer Sinn__: alles was Sie über die Option `-g|--get-via-http` wissen müssen ist, sie niemals zu benutzen (außer Sie haben einen guten Grund, die Passwörter der zu sichernden ownCloud/Nextcloud Nutzer preiszugeben).

## Über die Option -i | -\-include-shares

__ACHTUNG__: es gibt keinen Grund diesen Abschnitt zu lesen, außer Sie möchten *calcardbackup* mit der veralteten Option `-g|--get-via-http` aufrufen, wovon abgeraten wird (siehe vorhergehender Abschnitt).

Falls, aus welchem Grund auch immer, *calcardbackup* mit der Option `-g|--get-via-http` aufgerufen wird (wovon abgeraten wird!), kann folgendermaßen vorgegangen werden, um die Passwörter der Hauptnutzer geheim zu halten:
- einen neuen Benutzer in ownCloud/Nextcloud erstellen
- diejenigen Adressbücher/Kalender, die gesichert werden sollen, mit diesem neuen Benutzer teilen
- in der Datei `users.txt` nur den Nutzernamen und Passwort von dem neu angelegten Benutzer angeben
- die Option `-i` benutzen, um geteilte Elemente auch zu sichern

__Vorteil dieser Herangehensweise__: falls die Datei `users.txt` in falsche Hände gerät, wird nur dieses neue Benutzerkonto kompromittiert sein.  
__Nachteil__: neu erstellte Kalender/Adressbücher werden nicht automatisch gesichert. Sie  müssen zuerst mit diesem neuen Benutzerkonto geteilt werden.

Aufgrund des neuen Standardverhaltens von *calcardbackup* >= 0.8.0 (die Sicherungsdateien direkt aus der Datenbank zu erstellen), ist diese Option eigentlich unnütz geworden. Falls diese Option trotzdem benutzt werden soll, um geteilte Elemente auch ohne die Option `-g` zu sichern, dann gibt es keinerlei Notwendigkeit Passwörter in der Datei `users.txt` anzugeben. Falls keine Datei mit Nutzernamen an das Skript mit der Option `-u` übergeben wird, wird die Option `-i` ignoriert werden.
