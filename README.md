# Einleitung
Dies ist die Dokumenation über mein Docker-Projekt für das Modul 300. Ich habe mir zum Ziel gesetzt das ich für dieses Docker Projekt verschidene Komponente verwende. Ursprünlich wollte ich einen Reverse Proxy mit dahinter einem Apache und einem Wordpress Server. Aus dieser Idee wurde leider Nichts weil Wordpress mir hier eine Strich durch meinen Plan gemacht hatte, indem das Wordpress hinter einem Reverse Proxy noch manuele Anpassungen benötig war es für dieses Projekt nicht günstig. Daher habe ich jetzt einfach einen Reverse Proxy der 3 Web-Applikationen Schützt. 2x Mal einen einfachen Apache und 1x Mal einen eigenen Container der auch auf Apache basiert. Die ganzen HTML Files liegen extern damit wen die Container hernter gefahren werden auch noch verfügbar sind. In den Containern intern gibt es ein eigenes Netzwerk welches von Extern nicht ereichbar ist. Die Webseiten sind nur via den Reverse Proxy ereichbar. Dies bietet für die Web Apllikationen zusätzlichen Schutz.

# Servicebeschreibung
Dieser Service ist ein einfacher Webserver bei welchem die Unterseiten auf verschidenen Server liegen und so könnte man zum Beispiel einen Blog und ein Wiki auf der gleichen IP haben.

# Netzwerkübersicht
Dieses Dockerprojekt hat ein Internes Netzwerk welches nur von den Containern selbst zugegrifen werden kann und eines das auch von Extern ereichbar ist. Von Extern ist nur der nginx Reverse Proxy über den Port 8080 ereichbar. 

    +------------------------------------------------------------------------------------------------+
    ! Dockernetz - 172.28.0.0/16                                                                     !  
    ! NAT-Port: 8080                                                                                 !	
    !                                                                                                !	
    !    +--------------------+          +---------------------+          +---------------------+    !
    !    ! Web App1           !          ! Web App2            !          ! Web App3            !    !
    !    ! Container: httpd   !          ! Container: httpd    !          ! Container: eigener  !    !
    !    ! IP: 172.28.1.3/16  !          ! IP: 172.28.1.4/16   !          ! IP: 172.28.1.5/16   !    !
    !    ! Port: 80           !          ! Port: 80            !          ! Port: 80            !    !
    !    ! URL: /web1         !          ! URL: /web2          !          ! URL: /web3          !    !
    !    +--------------------+          +---------------------+          +---------------------+    !
    !                       /|\                    /|\                    /|\                        !
    !                        |                      |                      |                         !
    !                        \______________________|______________________/                         !
    !                                               |                                                !
    !                                               |                                                !
    !                                    +----------|----------+                                     !
    !                                    ! Nginx Server        !                                     !
    !                                    ! Reverse Proxy       !                                     !
    !                                    ! IP: 172.28.1.2/16   !                                     !
    !                                    ! Container: nginx    !                                     !
    !                                    ! Port: 80            !                                     !
    !                                    ! NAT: 8080           !                                     !
    !                                    +---------------------+                                     !
    !                                              /|\                                               !
    !                                               |                                                !
    +-----------------------------------------------|------------------------------------------------+
                                                    |
# Container Bauen
Um meinen eigenen Docker Container zu bauen benötigt man nur das Dockerfile. Mit dem Befehl `docker build -t flavio .` Baut man sich meinen Container zusammen. Er Basiert auf Ubuntu 18.04 und Apache.

# Umgebung Starten
Um diese Docker Umgebung zu starten muss man gewisse vorbereitungen Treffen.
1. Alle Datein Herunterladen. (docker-compose.yml,Dockerfile,nginx.conf)
2. Ein Verzeichnis Erstellen.
3. Die Heruntergeladen Datein in dieses Verzeichnis verschieben.
4. 3 Ordner erstellen mit den Namen `http1`,`http2`und`http3`. 
5. In jedes Verzeichnis mindestens eine `index.html` erstellen.
6. Im nginx.conf die Line 7 Anpassen auf die IP des Host's auf dem die Docker Umgebung läuft, alternativ kan man hier auch einen DNS Namen eintragen der auf den Host Zeigt.
7. Den Eigenen Container Bauen mit ``
8. Mit `docker-compose up -d` die ganze Umgebung starten
Fertig

Um Die Umgebung herunter zu fahren reicht ein Befehl: `docker-compose down`.

| Service      | Testfall               | Beschreibung                                                            | Erwartetes Ergebnis                                                            | Tatsächliches Ergebnis                             |
|--------------|------------------------|-------------------------------------------------------------------------|--------------------------------------------------------------------------------|----------------------------------------------------|
| Proxy        | Nginx Anzeige          | Wen man auf die IP geht bekommt man eine Webseite die vom Nginx stammt. | Nginx git den Fehler 404 zurück weil er selbst keine Webseite beinhaltet.      | Nginx gibt den Fehler 404 zurück.                  |
| Web Server 1 | Web Server 1 ereichbar | Der Web Server 1 sollte eine kleine Webseite Anzeigen                   | Die Webseite sollte angezeigt werden und auf der Seite sollte "Web 01" stehen. | Die Webseite wird angezeigt und es steht "Web 01". |
| Web Server 2 | Web Server 2 ereichbar | Der Web Server 2 sollte eine kleine Webseite Anzeigen                   | Die Webseite sollte angezeigt werden und auf der Seite sollte "Web 02" stehen. | Die Webseite wird angezeigt und es steht "Web 02". |
| Web Server 3 | Web Server 3 ereichbar | Der Web Server 3 sollte eine kleine Webseite Anzeigen                   | Die Webseite sollte angezeigt werden und auf der Seite sollte "Web 03" stehen. | Die Webseite wird angezeigt und es steht "Web 03". |
