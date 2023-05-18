# Docker Continuous Integration
 
## Beschreibung

Docker Image, zur Verwendung eines CI/CD Prozesses bei dem eine statische Codeanalyse durchgeführt wird.

---

## Aufgabenstellung

Es soll ein `Dockerfile` erstellt werden. Aus diesem wird dann ein Image gebaut.
Hierbei soll ein `docker-compose.yml`-File zum Bau des Images und starten des Containers
verwendet werden.

**Anforderungen**:
* [x] Automatischer Bau des Images via `docker-compose`
* [x] Start des Containers & clonen eines Git Repos (**URL** in [docker-compose.yml](https://github.com/ckiri/docker-ci/blob/2f78dcce76924d896d141129edbc5b64abebf93d/docker-compose.yml)).
* [x] Gradle Plugin **"Checkstyle"**(Google Java Style Guide) analysiert den Quellcode.
* [x] [Checkstyle](https://github.com/ckiri/docker-ci/blob/2f78dcce76924d896d141129edbc5b64abebf93d/google_checks.xml)-Datei befindet sich im Verzeichnis des [Dockerfiles](https://github.com/ckiri/docker-ci/blob/3db6df033d504eaa9697f4268e1d455c76729356/Dockerfile).
* [x] Angabe Pfad zum Quellcode im Repository in `docker-compose.yml` 
* [x] Ausführung von Checkstyle erfolgt über ein [Gradle-Skript](https://github.com/ckiri/docker-ci/blob/2f78dcce76924d896d141129edbc5b64abebf93d/build.gradle).
* [x] Angabe der Pfade & URLs sollen als Parameter in `docker-compose.yml` angegeben werden(hier in [.env](https://github.com/ckiri/docker-ci/blob/2f78dcce76924d896d141129edbc5b64abebf93d/.env)).
* [x] Auftretende Fehler sollen via `docker logs` ausgegeben werden.
* [x] Fehler beim Ausführung von Checkstyle sollen ebenfalls in `docker logs` ausgegeben werden.
* [x] **Pfade** und **URLs** dürfen **NICHT** im Image ***hinterlegt*** sein!

---

Folgend nun Befehle zur Nutzung von Docker-Containern. Annahme ist hierbei das Container von einem
User ohne Root rechte ausgeführt/gestartet wird. Daher auch die Nutzung von `sudo`.

## Starten des Containers & Bau des Images

Um den automatisierten Bau und Stylecheck des Projekts zu starten, müssen `src` Direktiven des
Java Projekts als Umgebungsvariable oder Parameter in `docker-compose.yml` angegeben werden 
(*Pfad der Quellcode-Datei immer von der Wurzel des Projekts aus angeben*):

```bash
sudo docker-compose up -d 
```

---

## Container logs auslesen

Auslesen der Container logs ID oder Name (falls vorhanden) angeben:

```bash
sudo docker logs docker-ci
```

---

## Container stoppen & entfernen

Stoppen des Containers:

```bash
sudo docker stop docker-ci
```

Entfernen des Containers:

```bash
sudo docker rm docker-ci
```

## Container neu starten

Hierbei wird das Image aber nicht neu gebaut!

```bash
sudo docker restart docker-ci
```

---

## Löschen des gebauten Images

Entfernen des Images (falls Image z.B. neu gebaut werden muss):

```bash
sudo docker rmi dockerci:checkstyle
```

---

## Gradle Skript

Die statische Codeanalyse wird mithilfe eines Gradle Skripts durchgeführt. In diesem
Skript wird die `checkstyle.xml`-Datei hinterlegt. Hier ist es wichtig das Plugin `checkstyle`
anzugeben. Benötigt wird ebenfalls das Repository `mavenCentral()` sowie `checkstyle` dependencie.

```gradle
plugins {
    id 'java'
    id 'checkstyle'
}

repositories {
    mavenCentral()
}

dependencies {
    checkstyle 'com.puppycrawl.tools:checkstyle:10.9.3'
}
...
```

Als nächstes wird das checkstyle Plugin konfiguriert. Es wird der Pfad der `checkstyle.xml`-Datei
angegeben. Ebenso die Version des Plugins sowie andere Optionen. In diesem Fall ob z.B. Checkstyle
Fehler erkannt werden oder nicht.

```gradle
...
checkstyle {
    configFile = file("${rootDir}/config/checkstyle/checkstyle.xml")
    toolVersion = "10.9.3"
    ignoreFailures = false
}
...
```

zum Schluss werden noch die Pfade zu den Direktiven des zu überprüfenden Quellcodes angegeben. 
Um hartes hinterlegen von Pfaden im Skript zu vermeiden, können Umgebungsvariablen verwendet
werden. Dazu aber später mehr.

```gradle
...
sourceSets {
    main {
        java {
            srcDirs = ["${System.getenv('MAIN_SRC_DIR')}"]
        }
    }

    test {
        java {
            srcDirs = ["${System.getenv('TEST_SRC_DIR')}"]
        }
    }
}
```
---

## Dockerfile

Um die im vorherigen Abschnitt erwähnten Umgebungsvariablen für das Gradle Build-Skript verfügbar
zu machen, müssen diese an den Container "übergeben" werden. Die Variablen werden zuerst als
Argumente `ARG` von der Command-Line als Parameter oder wiederum als Umgebungsvariable `ENV` vom
Hostsystem übergeben:

```Dockerfile
...
ARG	REPO_URL	
ARG	MAIN_SRC_DIR	
ARG	TEST_SRC_DIR

# Argumente werden als Umgebungsvariablen im Container bereitgestellt
ENV	REPO_URL=${REPO_URL}		
ENV	MAIN_SRC_DIR=${MAIN_SRC_DIR}
ENV	TEST_SRC_DIR=${TEST_SRC_DIR}
...
```

Nun werden die erforderlichen Softwarepakete mit `apk` installiert. `Git` zum style checken des
Repositories & `JDK` zum bauen des Java Projekts. `Gradle` wird nicht benötigt, aber dazu
später mehr. Es wird außerdem ein Verzeichnis `/app` erstellt indem später alle Dateien befinden
werden.

```Dockerfile
...
RUN	apk add --no-cache git openjdk11 && \
	mkdir /app 
...
```

Das Verzeichnis `/app` wird als `WORKDIR` verwendet. Dies ist nun der AUsgangspunkt für alle weiteren
Befehle.

```Dockerfile
...
WORKDIR	/app
...
```

Es wird nun das Repository geklont, hierfür wird wiederum auf eine Umgebungsvariable zugegriffen
um ein hartes hinterlegen der URL zu vermeiden. Die Umgebungsvariable wurde als `ARG` dem Dockerfile
übergeben. Außerdem wird noch das Im Gradle-Skript benötigte Verzeichnis in welches die Checkstyle
Datei gemountet wird, erstellt.

```Dockerfile
...
RUN	cd /app && \
	git clone -o origin ${REPO_URL} . && \
	mkdir -p config/checkstyle
...
```

Zum Schluss wird noch via `CMD` das Gradle Skript zum Checken des Projektes ausgeführt:

```Dockerfile
...
CMD	["./gradlew", "check"]
...
```

---

## Docker-Compose

Mit der `docker-compose.yml`-Datei lassen sich lange listen von Parametern für den Bau bzw. den Start
eines Containers einfacher und übersichtlicher darstellen. Außerdem ist hiermit das erstellen eines
containers wesentlich reproduzierbarer.

Am Anfang eines Compose-Files steht immer `services`. Es gibt hierbei immer einen oder mehrere Services.
In diesem Fall gibt es einen Service `checkstyle:`. Hier wird z.B. das Bauen des Images unter `build:`
spezifiziert. Mit `dockerfile` wird das zu verwendende Dockerfile zum Bau festgelegt.
Außerdem werden hier die Umgebungsvariablen für den Container unter `args:` bereitgestellt.
Weitere Sachen wie der Name des Containers zum einfacheren starten, stoppen und entfernen kann hier
festgelegt werden. Ebenfalls der Tag des gebauten Images. Zum Schluss werden noch die benötigten Daten
für das Gradle Skript als volume-mount in den Container gemountet:

```yaml
services:
  checkstyle:
    build:
      dockerfile: Dockerfile
      args:
        - REPO_URL=$REPO_URL
        - MAIN_SRC_DIR=$MAIN_SRC_DIR
        - TEST_SRC_DIR=$TEST_SRC_DIR
    container_name: docker-ci
    image: dockerci:checkstyle
    volumes:
      - ./google_checks.xml:/app/config/checkstyle/checkstyle.xml
      - ./build.gradle:/app/build.gradle
      - ./gradlew:/app/gradlew
      - ./gradle:/app/gradle
```

Die in docker-compose Datei angegebenen Umgebungsvariablen sind in der Datei `.env` angegeben.
Wird docker-compose aufgerufen, so wird automatisch `.env` nach Umgebungsvariablen "durchsucht".

```bash
REPO_URL=https://github.com/ckiri/example-project.git
MAIN_SRC_DIR=src/main/java
#TEST_SRC_DIR=src/first
```
`#TEST_SRC_DIR=src/first` wurde auskommentiert da für die Aufgabenstellung keine Tests gecheckt werden
müssen.
