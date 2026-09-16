===========================================================================
 ESP32 Radio - wersja dla plytek z koscia flash XMC (bootloader z latka)
 Firmware: v1.0.5_E  -  identyczne jak w zwyklym instalatorze
===========================================================================

DLA KOGO JEST TA WERSJA
-----------------------
Tylko dla plytek, ktore po wgraniu zwyklej wersji z instalatora WWW wpadaja
w petle restartow z takim logiem na porcie szeregowym:

    E (31) bootloader_flash: XMC flash startup fail
    E (31) boot.esp32s3: failed when running XMC startup flow, reboot!

Jesli Twoja plytka startuje normalnie - NIE uzywaj tych plikow, zostan przy
standardowej wersji z instalatora.


DLACZEGO TAK SIE DZIEJE
-----------------------
Bootloader ESP-IDF ma wbudowana procedure startowa dla kosci flash marki XMC.
Wykonuje sekwencje budzenia kosci, po czym czyta jej numer identyfikacyjny
(RDID) i porownuje z lista modeli znanych Espressif. Akceptowane sa tylko
kosci o RDID zaczynajacym sie od 20 40, 20 41 lub 20 50.

Zgloszona plytka ma RDID = 00207018, czyli producent 0x20 (XMC) zgadza sie,
ale model 0x70 nie ma prawa przejsc przez te liste. Bootloader traktuje to
jako blad krytyczny i resetuje uklad - i tak w kolko. Sama kosc flash dziala
przy tym zupelnie normalnie, co widac po tym, ze uklad bez problemu wczytuje
z niej bootloader.

WAZNE - dwie rzeczy, ktore NIE sa przyczyna i nie pomagaja:
  - Tryb flasha (QIO/DIO). Ta procedura odpala sie bezwarunkowo, zanim
    bootloader w ogole dojdzie do ustawiania trybu. Wczesniejsza wersja
    "xmc-dio" nie pomagala i zostala wycofana.
  - Czestotliwosc flasha. Na tej plytce 80 MHz dziala poprawnie, zejscie na
    40 MHz niczego nie zmienia.
  - Wylaczenie obslugi XMC w menuconfig tez NIE pomaga: bootloader odrzuca
    wtedy KAZDA kosc producenta 0x20.


CO JEST ZMIENIONE
-----------------
Podmieniony jest DOKLADNIE JEDEN plik - bootloader. Ma nalozona latke, ktora:
  - ponawia sekwencje budzenia kosci do 4 razy z coraz dluzszymi
    opoznieniami (zamiast jednej proby),
  - wypisuje odczytany RDID przed i po kazdej probie,
  - gdy kosc nadal nie pasuje do listy - pozwala systemowi wystartowac
    zamiast resetowac uklad.

Jesli flash naprawde bylby uszkodzony, start zatrzyma sie chwile pozniej, na
odczycie tablicy partycji albo obrazu aplikacji - wiec nie jest gorzej niz
teraz, a w praktyce plytka po prostu rusza.

Firmware, tablica partycji i panel WWW sa BIT W BIT takie same jak w zwyklym
instalatorze v1.0.5_E. Kosc, ktora przechodzi standardowe sprawdzenie, idzie
dokladnie ta sama sciezka co zawsze.


KTORY PLIK WYBRAC
-----------------
Pelne obrazy (16 MB), wgrywane w calosci pod adres 0x0:

  esp32-radio-FULL-xmc-retry.bin            ST7789 / ILI9341 (wersja domyslna)
  esp32-radio-FULL-ili9488-xmc-retry.bin    ILI9488 320x480
  esp32-radio-FULL-lcd19-xmc-retry.bin      Waveshare ESP32-S3-LCD-1.9
  esp32-radio-FULL-1732s019-xmc-retry.bin   ESP32-1732S019

Wybierz ten sam wariant wyswietlacza, ktorego uzywasz w zwyklym instalatorze.
Wgranie pliku od innego wyswietlacza nie uszkodzi plytki, ale ekran zostanie
pusty albo obraz bedzie posuniety.

Osobno dolaczony jest sam bootloader:

  bootloader-xmc-retry.bin                  tylko bootloader, adres 0x0

Ten plik przyda sie, gdy masz juz wgrane radio zwyklym instalatorem i chcesz
podmienic wylacznie bootloader, bez ruszania ustawien i sieci Wi-Fi.


JAK WGRAC
---------
Przez przegladarke (Chrome lub Edge):

  1. Wejdz na https://espressif.github.io/esptool-js/
  2. Polacz sie z plytka i wybierz jej port COM.
  3. Wskaz wybrany plik i ustaw adres:  0x0
  4. Przy pierwszym wgraniu (albo po bootloopie) warto najpierw wyczyscic
     caly flash.
  5. Uruchom wgrywanie, a po jego zakonczeniu nacisnij RESET na plytce.

WAZNE: nie wymuszaj trybu flasha ani czestotliwosci w opcjach wgrywania -
zostaw wartosci domyslne. Recznie ustawiony tryb podmienia bajty w naglowku
obrazu bez przeliczenia jego sumy kontrolnej, przez co przy starcie pojawia
sie "SHA-256 comparison failed". Nie blokuje to startu, ale zasmieca log.

Z linii polecen:

    esptool --chip esp32s3 write-flash 0x0 esp32-radio-FULL-xmc-retry.bin

Starsze esptool (4.x) uzywa podkreslenia zamiast myslnika:

    esptool.py --chip esp32s3 write_flash 0x0 esp32-radio-FULL-xmc-retry.bin


PO WGRANIU
----------
Plytka wystawi siec Wi-Fi o nazwie "ESP32-Radio", haslo: 12345678
Polacz sie z nia i otworz w przegladarce http://192.168.4.1 , zeby podac dane
swojej sieci domowej.

W logu na porcie szeregowym (115200) zobaczysz linie takie jak:

    W (25) bootloader_flash: XMC: RDID=00207018 spoza listy strict, ...
    W (35) bootloader_flash: XMC: proba 1/4, RDID=00207018
    E (180) bootloader_flash: XMC flash startup fail (RDID=...) - PATCH:
            kontynuuje start mimo to

To jest OCZEKIWANE zachowanie tej wersji, nie blad - zaraz po tych liniach
system startuje normalnie.


JESLI NADAL NIE DZIALA
----------------------
Przyslij na forum caly log z portu szeregowego (115200) od momentu wlaczenia
zasilania. Najwazniejsza jest w nim wartosc RDID oraz to, czy start zatrzymuje
sie przed, czy juz po linii "Loaded app from partition".


AKTUALIZACJE W PRZYSZLOSCI
--------------------------
"Aktualizacja firmware przez WiFi" w panelu WWW pobiera zwykla wersje
firmware - to jest w porzadku, bo nie rusza bootloadera, wiec poprawka XMC
zostaje na miejscu. Przy wiekszych aktualizacjach (zmiana tablicy partycji
albo panelu WWW) poproś na forum o odswiezony plik z tego katalogu.
