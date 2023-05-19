# Vagrant Box

## Beschreibung
Es soll analog zu Aufgabe 2: [Docker-Nginx-Webserver](https://github.com/ckiri/dev-ops/tree/main/docker-nginx-webserver) ein Dockerfile erstellt werden welches es
erlaubt eine statische `index.html`-Datei auszuliefern. Mithilfe von Vagrant soll eine Virtuelle
Maschine erstellt werden die über Ansible provisioniert wird. Wie in Aufgabe 2, sollen Änderungen
in `index.html` automatisch übernommen werden. Hierfür wird ein `/vagrant` share Ordner verwendet
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

## Überprüfen
Wurde das Ansible Playbook erfolgreich "gespielt" kann das Reslutat über diese IP-Adresse:
`192.168.56.4:80` überprüft werden.

## Dokumentation

### Vagrantfile
Im `Vagrantfile` wird die Konfiguration der Virtuellen Maschine vorgenommen.


Es wird ein [Vagrant-Box](https://app.vagrantup.com/boxes/search) ausgewählt & konfiguriert. Es wird
CentOS installiert:
```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "geerlingguy/centos7"
  ...
end
```

Zum provisionieren mit Ansible nachdem die VM erstellt wurde:
```ruby
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
  end
```

Um die Datei `index.html` auf dem innerhalb der VM verfügbar zu machen, muss ein sog. synced oder
"share" Folder erstellt werden. es werden außerdem Parameter für zugriffsrechte gegeben:
```ruby
  config.vm.synced_folder "vagrant", "/vagrant", type: "virtualbox", mount_options: ["dmode=777", "fmode=666"]
```

In der Vagrant-Datei können außerdem die Host Hardwaressourcen spezifiziert werden. In diesem Fall
werden für die VM 1024MB an speicher auf dem Hostsystem reserviert:
```ruby
  config.vm.provider :virtualbox do |v|
    v.memory = 1024
    v.linked_clone = true
  end
```

Zum Schluss wird ncoh der hostname sowie die IP-Adresse der VM spezifiziert:
```ruby
  config.vm.define "centos" do |app|
    app.vm.hostname = "centos"
    app.vm.network :private_network, ip: "192.168.56.4"
  end
```

### Inventory
Im `inventroy`-File werden die Server welche über Ansible provisioniert werden sollen, eingetragen.

Die aus dem `Vagrantfile` angegebenen Server auflisten (hier nur die CentOS VM):
```ini
[centos]
192.168.56.4
```

Es können zudem auch Variablen der VMs gesetzt werden. Hier zum "debuggen" die SSH Funktion gesetzt
um sich nach dem erfolgreichen Start der VM via remote, aufzuschalten:
```ini
[multi:vars]
ansible_ssh_user=vagrant
ansible_ssh_orivate_key_file=~/.vagrant.d/insecure_private_key
```

### Playbook
Im Playbook: `playbook.yml` werden die "Plays" also die Tasks angegeben die auf den Zielsystemen
ausgeführt werden soll. Um das Playbook übersichtlicher und modularer zu machen werden einzelne
Taskgruppen in sogenannte `roles` unterteilt:
```yml
---
- name: webserver-deployment
  hosts: centos
  become: true

  roles:
    - role: setup
    - role: deployment
```
In diesem Fall wird das Playbook auf das im Inventory angegebene Zielsystem (CentOS) angewendet.
Das Playbook wurde in zwei Rollen `setup` und `deployment` unterteilt.

#### role: "setup"
Als erstes wird die Rolle `setup` gespielt. Hier wird gecheckt ob die erforderlichen Softwarepakete
installiert sind und Services gestartet sind.

Ein Task in Ansible besteht aus bestenfalls aus einem Modul. In diesem "[Modul](https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html)" können Optionen
spezifiziert werden. Da CentOS den Paketmanager `yum` verwendet, kann dieses Modul zum installieren
der Pakete genutzt werden:
```yml
- name: Ensure the necessary docker packages are installed.
  yum:
    name: 
      - docker
      - python-docker
    state: present
```
Für das in der späteren Rolle `deplyoment` wird das Python Paket: `python-docker` benötigt, um über
Ansible ein Dockerfile zu bauen. Der `state` ins Modulspeziefisch und gibt den Status der Pakete an.
Hier `present` also vorhanden. Durch die Idempotenz, wird falls die Paktet nicht vorhanden sind, diese
installiert. Falls diese aber schon vorhanden sind, nicht.

Der letzte Task in der Rolle `setup`, startet den Docker service also die Laufzeitumgebung für Docker.

#### role: "deplyoment"
Wurde die Rolle `setup` erfolgreich gespielt, so wird zur nächsten Rolle `deplyoment` übergegangen.
Es wird das Ansible Modul `docker_image` zum builden des images aus einem Dockerfile verwendet.
Nach erfolgreichem builden, wird der Container mit dem Modul `docker_container` gestartet:
```yml
- name: Ensure the container image is built.
  docker_image:
    name: nginx:webserver
    build:
      path: /vagrant
    state: present
    source: build


- name: Ensure the container is runnung.
  docker_container:
    name: webserver
    image: "nginx:webserver"
    state: started
    ports:
      - "80:80"
    volumes:
      - "/vagrant/index.html:/var/www/html/index.html"
```
