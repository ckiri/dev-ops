# Vagrant Box

## Beschreibung
Es soll analog zu Aufgabe 2: [Docker-Nginx_webserver]() ein Dockerfile erstellt werden welches es
erlaubt eine statische `index.html`-Datei auszuliefern. Mithilfe von Vagrant soll eine Virtuelle
Maschine erstellt werden die über Ansible provisioniert wird. Wie in Aufgabe 2, sollen Änderungen
in `index.htlm` automatisch übernommen werden. Hierfür wird ein `/vagrant` share Ordner verwendet
werden.

## Vorraussetzung
Um die Provisiunierung starten zu können, wird Vagrant, Ansible sowie VirtualBox als Provider für
für Vagrant benötigt.

Debian based Distros:
```bash
sudo apt install ansible vagrant virtualbox
```

Arch based Distros:
```bash
sudo pacman -S ansible vagrant virtualbox
```

RHEL based Distros:
```bash
sudo yum install ansible vagrant virtualbox
```

## Ausführen
Beim start von Vagrant wird das Ansible `playbook.yml` automatisch ausgeführt:
```bash
sudo vagrant up
```

Falls Ansible nicht automatisch gestartet wurde:
```bash
sudo vagrant provision
```

## Dokumentation
(WIP)