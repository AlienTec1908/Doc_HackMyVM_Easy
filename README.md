Okay, Ben! Das ist wieder ein klassischer Ablauf mit einigen interessanten Schritten. Hier ist der Entwurf für das "Doc" README:

# Doc - HackMyVM (Easy)

![Doc.png](Doc.png)

## Übersicht

*   **VM:** Doc
*   **Plattform:** [HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Doc)
*   **Schwierigkeit:** Easy
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 07. Mai 2024
*   **Original-Writeup:** https://alientec1908.github.io/Doc_HackMyVM_Easy/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel dieser "Easy"-Challenge war es, Root-Zugriff auf der Maschine "Doc" zu erlangen. Die Enumeration deckte einen Webserver (nginx) auf, der eine "Online Traffic Offense Management System - PHP"-Anwendung hostete. Ein bekannter Exploit (Exploit-DB 50221) für diese Anwendung ermöglichte Authentifizierungsbypass und RCE durch Datei-Upload, was zu einer initialen Shell als `www-data` führte. Im Quellcode der Anwendung (`initialize.php`) wurden Datenbank-Credentials (`bella:be114yTU`) gefunden, die auch für den Systembenutzer `bella` gültig waren. Nach dem Wechsel zu `bella` wurde die User-Flag gelesen. Eine `sudo`-Regel erlaubte `bella` das Ausführen von `/usr/bin/doc` (einem `pydoc`-Wrapper) als jeder Benutzer ohne Passwort. Durch Starten dieses Dienstes mit `sudo doc` wurde ein lokaler Webserver auf Port 7890 mit Root-Rechten gestartet. Dieser Server war anfällig für Local File Inclusion (LFI) über den `getfile?key=`-Parameter, wodurch die Root-Flag (`/root/root.txt`) ausgelesen werden konnte.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `gobuster`
*   `python2` (für Exploit-Skript)
*   `nc` (netcat)
*   `python3` (für pty-Shell-Stabilisierung)
*   `su`
*   `curl`
*   `sudo` (auf Zielsystem)
*   `doc` (`pydoc`-Wrapper, als Exploit-Vektor)
*   Standard Linux-Befehle (`ls`, `whoami`, `export`, `cat`, `ss`, `find`, `cd`, `id`, `vi`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Doc" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Web Enumeration:**
    *   IP-Findung mittels `arp-scan` (Ziel: `192.168.2.128`, Hostname `dio.hmv` oder `doc.hmv`).
    *   `nmap`-Scan identifizierte nur Port 80 (HTTP, nginx) mit der Anwendung "Online Traffic Offense Management System - PHP".
    *   `gobuster` listete diverse Verzeichnisse und PHP-Dateien der Anwendung auf.

2.  **Initial Access (als `www-data` via Web App Exploit):**
    *   Ein öffentlicher Exploit für "Online Traffic Offense Management System 1.0" (Exploit-DB 50221) wurde verwendet.
    *   Das Python2-Exploit-Skript umging die Authentifizierung, lud eine PHP-Webshell (`evil.php`) hoch und startete eine Reverse Shell zu einem Netcat-Listener des Angreifers.
    *   Erfolgreicher Shell-Zugriff als `www-data`.

3.  **Privilege Escalation (von `www-data` zu `bella`):**
    *   Als `www-data` wurde die Datei `initialize.php` (Teil der Webanwendung) gelesen.
    *   Diese enthielt Klartext-Datenbank-Credentials: Benutzer `bella`, Passwort `be114yTU`.
    *   Mit `su bella` und dem Passwort `be114yTU` wurde erfolgreich zum Systembenutzer `bella` gewechselt.
    *   Die User-Flag wurde im Home-Verzeichnis von `bella` gelesen.

4.  **Privilege Escalation (von `bella` zu `root` via Sudo/pydoc LFI):**
    *   Als `bella` zeigte `sudo -l`, dass `/usr/bin/doc` als jeder Benutzer (`ALL : ALL`) ohne Passwort ausgeführt werden durfte.
    *   Der Befehl `sudo /usr/bin/doc` startete einen lokalen Webserver (einen `pydoc`-Wrapper) auf `http://localhost:7890/` mit Root-Rechten.
    *   Dieser Server war anfällig für Local File Inclusion (LFI).
    *   Mittels `curl http://localhost:7890/getfile?key=/root/root.txt` wurde der Inhalt der Root-Flag-Datei ausgelesen.

## Wichtige Schwachstellen und Konzepte

*   **Bekannte Webanwendungs-Schwachstelle (RCE):** Ausnutzung eines öffentlichen Exploits für "Online Traffic Offense Management System 1.0".
*   **Hartcodierte Credentials:** Datenbank-Passwörter im Klartext in `initialize.php`.
*   **Passwort-Wiederverwendung:** Das Datenbankpasswort für `bella` war identisch mit dem Systempasswort.
*   **Unsichere `sudo`-Regel (`pydoc` Wrapper):** Erlaubte das Starten eines Dienstes (`pydoc`-Server) mit Root-Rechten, der selbst anfällig war.
*   **Local File Inclusion (LFI) im `pydoc`-Server:** Ermöglichte das Lesen beliebiger Dateien als Root.

## Flags

*   **User Flag (`/home/bella/user.txt`):** `HMVtakemydocs`
*   **Root Flag (gelesen via LFI):** `HMVfinallyroot`

## Tags

`HackMyVM`, `Doc`, `Easy`, `Web`, `Nginx`, `PHP`, `RCE`, `Exploit-DB`, `Credentials Disclosure`, `Password Reuse`, `sudo`, `pydoc`, `LFI`, `Privilege Escalation`, `Linux`


Ich denke, das fasst die Challenge gut zusammen. Lass mich wissen, ob alles passt!
