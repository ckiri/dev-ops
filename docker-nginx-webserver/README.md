# Nginx Webserver in einem Dockercontainer

----
## Aufgabenstellung

Es soll ein ein Docker Image aus einem `Dockerfile` erstellt werden.
In diesem soll ein Nginx Webserver von grund auf erstellt werden.
Die Datei `index.html` muss hierbei über `-v` (volume mount) oder `--mount` (bind mount)
im virtuellen Filesystem des Docker Containers gemounted werden, sodass Veränderungen
in dieser Datei automatisch beim refreshen der Seite übernommen werden.
Die Seite soll über Port 80 (`localhost:80`) erreichbar sein.

----
## Vorgehensweise

Um einen Docker Container starten zu können dient das Image als Grundlage.
Dieses muss erst geschrieben werden und anschließend gebaut werden.

### 1. Dockerfile erstellen

Als Grundlage für den Webserver wird Alpine-Linux verwendet.
Mit `FROM` wird das neuste Alpine Image als Grundlage genommen:

```Dockerfile
FROM	alpine:latest
```

Als nächstes müssen Konfigurationen in Alpine vorgenommen werden um den
Server auf dieser Grundlage luafen zu lassen. Mit `RUN` wird eine Shellsession
im Container gestartet. Hierbei ist zu erwähnen, dass nun möglichst alle
Befehle die mit einer Shell durchgeführt werden sollen, hier angehängt werden.
Denn bei jedem `RUN` wird eine neue session gestartet und dies kostet Performance:

```Dockerfile
RUN	apk add --no-cache nginx && \
	mkdir -p /var/www/html
```

`apk` ist der Package Manager von Alpine-Linux. Mit `add` werden Pakete installiert.
Die Flag `--no-cache` verhindert lokales Caching und sorgt dafür das der Container
möglichst klein bleibt. `mkdir -p` erstellt den angegebenen Ordner und die
dazu notwendigen "Parent"-Ordner.

Mit `COPY` werden Dateien aus dem Host System in den Container kopiert.
Hier wird bei der Source, also dem ersten Argument ein relativer Pfad angegeben. D.h.
liegt die zu kopierende Datei in dem Verzeichnis des Dockerfiles, wird diese kopiert.
Beim Target (2. Argument) wird der Zielpfad absolut angegeben. Da die Standartconfig
von Nginx jedoch keine Dateien referenziert muss diese erstellt und dann in den Container
an den Standartconfig Pfad kopiert werden:

```Dockerfile
COPY	nginx.conf /etc/nginx/nginx.conf
```

`EXPOSE` git den Port an, auf welchem der Container verfügbar ist. Dies ist aber Container
intern und nicht bezogen auf das Hostsystem.

```Dockerfile
EXPOSE	80
```

`CMD` führt die in eckigen klammern stehenden Befehle aus, ohne dabei eine Shellsession zu starten.
Hierbei wird Nginx gestartet wobei die die Dämonisierung deaktiviert wird.

```Dockerfile
CMD 	["nginx", "-g", "daemon off;"]
```

Das vollständige Dockerfile:

```Dockerfile
FROM 	alpine:latest

RUN	apk add --no-cache nginx && \
	mkdir -p /var/www/html

COPY	nginx.conf /etc/nginx/nginx.conf

EXPOSE	80

CMD 	["nginx", "-g", "daemon off;"]
```

----
### 2. Docker Image bauen

Das zuvor erstellte Image kann nun gebaut werden. Die Flag `-t` vergibt dem Docker Image einen Tag.
Dieser kann dann zum einfacheren starten, stoppen und entfernen des Containers verwendet werden.
Der Punkt `.` nach dem Tag ist wichtig, da dieser den Pfad des Dockerfiles spezifiziert. Der Punkt
steht hier für den Pfad des aktuellen Ordners:

```Bash
docker build -t nginx:webserver .
```

----
### 3. Container starten

Zum starten des Containers muss mit der Flag `-p` das Portmapping des Hostsystems zum Container
angegeben werden. Der erste Port ist hier der des Containers und der zweite der des Hostsystems.
mit `-v` wird ein Volume Mount vorgenommen. Hierbei können Dateinen/ordner vom Hostsystem im
Container gemountet werden. Die Struktur ist ein absoluter Pfad bestehen aus `Source:Target`.
Mit `--name` kann dem Container ein Name vergeben werden. `-d` spezifiziert das zu verwendende Image.

```Bash
docker run -p 80:80 -v "$(pwd)"/index.html:/var/www/html/index.html --name webserver -d nginx:webserver
```
