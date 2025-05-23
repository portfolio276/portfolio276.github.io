---
title: "Palo Alto GNS3 Lab"
date: 2025-05-21 13:20:30 +0100  # Warsaw timezone (UTC+1)
description: "Projekt do nauki obsługi firewalli Palo Alto w wirtualnym środowisku GNS3"
# categories: [Blogging, Tutorial]
# tags: [jekyll, chirpy, first-steps]
# author: your_author_id  # Optional - if you have authors.yml set up
pin: false  # Set to true to pin to homepage
toc: true  # Show table of contents
math: false  # Set to true if using math equations
mermaid: false  # Set to true if using diagrams
comments: false  # Enable/disable comments
image:
  path: /assets/img/paloalto1.png
  alt: ""
---

## Wstęp
Do stworzenia środowiska wykorzystany został emulator [GNS3](https://gns3.com/).
Dodatkowo potrzebne było również narzędzie do konteneryzacji - na przykład [Docker](https://www.docker.com/).

Po uruchomieniu programu GNS3 tworzymy nowy szablon i wybieramy opcję pozwalająca na zainstalowanie urządzenia z serwera GNS. Wybieramy firewall Palo Alto na emulatorze Qemu i klikamy "Install".
![GNS3 PA-VM](/assets/img/pa-vm1.png)

Instalujemy na maszynie wirtualnej GNS3, jako Qemu binary wybieramy /usr/bin/qemu-system-x86_64 i przechodzimy dalej. Tworzymy nową wersję i wskazujemy plik z obrazem .qcow2. W przypadku wersji 10.1.9 liczbę vCPU należy ustawić przynajmniej na 2, a pamięć RAM na 4500MB. Po pierwszym uruchomieniu zostaniemy poproszeni o dane do logowania (domyślnie jest to `admin:admin`). Aby zmienić hasło wchodzimy w tryb konfiguracji poprzez komendę `configure`, następnie wpisujemy `set mgt-config users admin password`, dwukrotnie podajemy nowe hasło i zapisujemy przy użyciu `commit`. 

## Początkowa konfiguracja Palo Alto oraz webterm
W trybie konfiguracji wpisujemy polecenie ustawiające statyczny adres IP na interfejsie zarządzania (management), na przykład: `set deviceconfig system ip-address 192.168.1.1 netmask 255.255.255.0 type static`. Umożliwi nam to dostęp do firewalla poprzez interfejs webowy z sieci lokalnej w następnych krokach. Skonfigurujemy jeszcze serwery DNS od Google oraz Quad9 poleceniem: `set deviceconfig system dns-setting servers primary 8.8.8.8 secondary 9.9.9.9`. Na koniec oczywiście zapisujemy konfigurację komendą `commit`. 

Aby dodać kontener webterm, który umożliwi nam dostęp do firewalla Palo Alto poprzez przeglądarkę (oraz narzędzie terminala - bardzo przydatne, chociażby banalne polecenie `ping`), wybieramy opcję `New template`, wyszukujemy `webterm` i instalujemy. Przeciągamy webterm do naszego projektu i w konfiguracji sieci ustawiamy poniższe opcje:

![](/assets/img/webterm1.png)

Oczywiście adres jak i maska sieci mogą być inne, pod warunkiem, że zachowamy spójność w daleszej konfiguracji. 

Za pomocą tej instancji będziemy zarządzać naszym firewallem dlatego dodajemy połączenie między (jedynym) interfejsem `eth0`, a `management` w PA-VM. 
Podobne działanie wykonujemy jeszcze dwukrotnie - dla serwera, gdzie adres ustawimy na `192.168.10.1/24`, a interfejs w firewallu na `eth1/1` oraz klienta z adresem `192.168.20.1/24` i interfejsem `eth1/2` po drugiej stronie.
Nasz projekt na obecną chwilę powinien wyglądać mniej więcej tak:

![Schemat Projektu](/assets/img/webterm2.png)

## Logowanie do firewalla przez przeglądarkę
Uruchamiamy webterm-1 i wybieramy opcję `Console`. W przeglądarce odwiedzamy adres `192.168.1.1` i logujemy się do panelu administracyjnego za pomocą hasła, które ustawiliśmy w terminalu Palo Alto we wcześniejszym kroku.

![Palo Alto Initial View](/assets/img/webterm3.png)

(dla wygody możemy zainstalować np TigerVNC i ustawić `vnc` jako domyślny typ konsoli w ustawieniach PA-VM)

## Zarządzanie firewallem Palo Alto
