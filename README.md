# Takedown - HackMyVM (Hard)
 
![Takedown.png](Takedown.png)

## Übersicht

*   **VM:** Takedown
*   **Plattform:** HackMyVM (https://hackmyvm.eu/machines/machine.php?vm=Takedown)
*   **Schwierigkeit:** Hard
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 30. Juli 2025
*   **Original-Writeup:** https://alientec1908.github.io/Takedown_HackMyVM_Hard/
*   **Autor:** Ben C.

## Kurzbeschreibung

Dieses Writeup dokumentiert den komplexen Penetrationstest der Maschine "Takedown". Der Weg zum initialen Zugriff führte über die Entdeckung eines virtuellen Hosts und einer Subdomain, auf der ein Kontaktformular für eine Server-Side Template Injection (SSTI) anfällig war. Durch die Ausnutzung dieser Schwachstelle konnte direkt `root`-Zugriff innerhalb eines Docker-Containers erlangt werden. Die anschließende Herausforderung bestand in einem mehrstufigen Ausbruch aus dem Container und einer horizontalen sowie vertikalen Privilegienerweiterung auf dem Host-System. Dies umfasste die Ausnutzung eines unsicher gemounteten Verzeichnisses, einer Kette von `sudo`-Fehlkonfigurationen und schließlich einer Shell-Escape-Schwachstelle in einem benutzerdefinierten Programm, um finale `root`-Rechte auf dem Host zu erlangen.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `curl`
*   `nikto`
*   `netcat`
*   `python3`
*   `chisel`
*   `ssh`
*   `openssl`
*   `sudo`
*   Standard Linux-Befehle (`ls`, `cat`, `find`, `mount`, etc.)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Takedown" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Enumeration:**
    *   Identifizierung der Ziel-IP (`192.168.2.167`) mittels `arp-scan`.
    *   Ein `nmap`-Scan auf Port 80 zeigte einen Redirect auf den virtuellen Host `shieldweb.che`.

2.  **Web Enumeration:**
    *   Nach dem Eintragen von `shieldweb.che` in die `/etc/hosts`-Datei wurde eine weitere Subdomain, `ticket.shieldweb.che`, entdeckt.
    *   Auf dieser Subdomain befand sich ein Kontaktformular, das als primärer Angriffsvektor identifiziert wurde.

3.  **Initial Access (Server-Side Template Injection):**
    *   Ein einfacher Payload (`{{7*7}}`) bestätigte eine SSTI-Schwachstelle im Kontaktformular.
    *   Ein fortgeschrittener Python-SSTI-Payload wurde verwendet, um Befehle auszuführen (`{{...os.popen('id')...}}`). Dies enthüllte, dass die Anwendung als `root` innerhalb eines Docker-Containers lief.
    *   Durch einen persistenten Reverse-Shell-Payload wurde eine interaktive Shell als `root`-Benutzer im Alpine-Linux-Container erlangt.

4.  **Privilege Escalation (Container Escape zu Host):**
    *   Innerhalb des Containers wurde ein beschreibbares Verzeichnis `/script` entdeckt, das vom Host-System gemountet wurde.
    *   Die Hypothese war, dass ein Cronjob auf dem Host Skripte aus diesem Verzeichnis ausführt.
    *   Ein Reverse-Shell-Skript wurde in `/script` platziert. Nach kurzer Zeit wurde dieses vom Host ausgeführt und lieferte eine Shell als Benutzer `love` auf dem zugrundeliegenden Debian-Host.

5.  **Privilege Escalation (Host: `love` -> `mitnick` -> `tomu` -> `root`):**
    *   **love -> mitnick:** Der Benutzer `love` konnte per `sudo` ein binäres Programm `/home/mitnick/sas` als Benutzer `mitnick` ausführen. Dieses Programm hatte eine `run`-Funktion, die missbraucht wurde, um eine Shell als `mitnick` zu starten.
    *   **mitnick -> tomu:** Als `mitnick` wurden ein SSH-Schlüssel, eine verschlüsselte Datei und ein öffentlicher Schlüssel gefunden. Die verschlüsselte Datei wurde mit `openssl` und einem erratenen privaten Schlüssel (`idpom`) entschlüsselt, um das Passwort für den Benutzer `tomu` zu erhalten.
    *   **tomu -> root:** Der Benutzer `tomu` konnte per `sudo` ein Programm `/opt/Contempt/Contempt` als `root` ausführen. Dieses Programm hatte eine `vi`-ähnliche Shell-Escape-Funktion. Durch die Eingabe von `:!sh` konnte eine `root`-Shell auf dem Host-System erlangt werden.

## Wichtige Schwachstellen und Konzepte

*   **Server-Side Template Injection (SSTI):** Eine kritische Schwachstelle, die Remote Code Execution ermöglichte. Verschärft wurde dies, da die Anwendung im Container als `root`-Benutzer lief.
*   **Unsichere Docker-Konfiguration (Container Escape):** Ein vom Host gemountetes, beschreibbares Verzeichnis, das von einem unsicheren Cronjob auf dem Host überwacht wurde, ermöglichte den Ausbruch aus dem Container.
*   **Privilege Escalation via Sudo-Kette:** Eine mehrstufige Ausnutzung von Fehlkonfigurationen in den `sudo`-Regeln, die es erlaubten, schrittweise die Identität zu wechseln und die Rechte zu erweitern.
*   **Shell-Escape in Sudo-Binary:** Die finale Eskalation zu `root` war durch eine unsichere, benutzerdefinierte Anwendung möglich, die eine Flucht zur System-Shell erlaubte.

## Flags

*   **User Flag (`/home/tomu/user.txt`):** `612701a03669485d94bc687449fdab39`
*   **Root Flag (aus `/root/root.sh`):** `1e271c5ce97e76ae8417a95c74085fba`

## Tags

`HackMyVM`, `Takedown`, `Hard`, `SSTI`, `Docker Escape`, `Container`, `Sudo`, `Privilege Escalation`, `Linux`, `Web`
