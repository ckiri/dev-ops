# Ansible Playbook für Ubuntu 22.04 Server

## Beschreibung
Ansible Playbook zum Managen (Automatisches Setup) eines Ubuntu 22.04 Servers. Das Playbook
führt einen Cron-Job durch, öffnet bestimmte Ports des Servers und verhindert authentifizierungen via
Passwort.

## Vorraussetzungen
Folgende Einstellungen/Setup müssen vorgenommen werden um das Playbook ausführen zu können:

### Bei der Verwendung innerhalb eines Ubuntu 22.04 Systems/VM

Installation von `Ansible`:
```bash
sudo apt install ansible
```

Im `playbook.yml` muss folgendes stehen:
```yaml
...
Hosts: localhost
...
```

### Bei der Verwendung mit Vagrant

[`Vagrant`](https://www.vagrantup.com/) installieren um den Ubuntu Server als Virtuelle Maschine local laufen zu lassen.
[`Virtualbox`](https://www.virtualbox.org/) wird als provider für Vagrant benötigt.
```bash
apt install ansible vagrant virtualbox
```

Im `playbook.yml` muss folgendes stehen:
```yaml
...
Hosts: ubuntu
...
```

Ist `ubuntu` bei Hosts eingetragen, verwendet Ansible
die im `inventory` hinterlegte IP-Adresse.

## Ausführun & Testen

### Ubuntu 22.04
Das gesammte Playbook laufen lassen:
```bash
sudo ansible-playbook playbook.yml
```

Einzelne "Rollen" spielen:
```bash
sudo ansible-playbook PFAD/ZUR/ROLLE/main.yml
```

### Vagrant
Ubuntu Server starten:
```bash
sudo vagrant up
```

Falls playbook nicht ausgeführt wird:
```bash
sudo vagrant provision
```
