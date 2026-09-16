===========================================================================
 ESP32 Radio - wersja awaryjna dla plytek z kosciami flash XMC (tryb DIO)
 Firmware: v1.0.5_E   |   Wariant wyswietlacza: ST7789 / ILI9341 (domyslny)
===========================================================================

DLA KOGO JEST TEN PLIK
----------------------
Tylko dla plytek, ktore po wgraniu zwyklej wersji z instalatora WWW wpadaja
w petle restartow z komunikatem na konsoli szeregowej:

    E (31) bootloader_flash: XMC flash startup fail
    E (31) boot.esp32s3: failed when running XMC startup flow, reboot!

Jesli Twoja plytka startuje normalnie - NIE uzywaj tego pliku, zostan przy
standardowej wersji z instalatora (jest szybsza, patrz "Cena" nizej).


CO JEST INNEGO W TEJ WERSJI
---------------------------
Standardowa wersja uzywa flasha w trybie QIO (4 linie danych). Na czesci
kosci XMC bootloader drugiego stopnia nie przechodzi sekwencji startowej XMC
przy wlaczaniu QIO i restartuje uklad, zanim w ogole ruszy aplikacja.

Ta wersja ma podmieniony bootloader na wariant DIO (2 linie danych), ktory
w ogole nie probuje wlaczac QIO - problematyczna sciezka kodu nie jest
wykonywana. Czestotliwosc flasha bez zmian: 80 MHz.

WAZNE: wymuszanie "DIO" w esptool / w opcjach wgrywania NIE POMAGA i nie o to
tu chodzi. Naglowek obrazu w obu wersjach i tak ma juz wpisane DIO (uklad
startuje w DIO, a na QIO przelacza sie dopiero bootloader wedlug swojej
wkompilowanej konfiguracji). Trzeba podmienic sam bootloader - i dokladnie to
jest w tym pliku.

Cena: DIO to polowa przepustowosci QIO przy odczycie z flasha. W praktyce
oznacza to nieco wolniejszy start urzadzenia; odtwarzanie radia dziala tak
samo (strumien audio idzie z sieci, nie z flasha).

Poza bootloaderem to jest DOKLADNIE ta sama wersja 1.0.5_E co w zwyklym
instalatorze - ten sam kod, te same funkcje, ten sam panel WWW.


JAK WGRAC
---------
Plik esp32-radio-FULL-xmc-dio.bin to pelny obraz flasha (16 MB): bootloader +
tablica partycji + firmware + system plikow ze strona WWW. Wgrywa sie go
w CALOSCI pod adres 0x0.

  1. Wejdz na https://espressif.github.io/esptool-js/  (Chrome lub Edge)
  2. Connect -> wybierz port COM plytki
  3. W polu "Flash Address" wpisz:  0x0
  4. Wybierz plik esp32-radio-FULL-xmc-dio.bin
  5. Zaznacz "Erase all" (zalecane przy pierwszym wgraniu / po bootloopie)
  6. Program

Alternatywnie z linii polecen:

    esptool --chip esp32s3 write-flash 0x0 esp32-radio-FULL-xmc-dio.bin

Po wgraniu plytka wystawi siec Wi-Fi "ESP32-Radio-AP" - polacz sie z nia
i otworz http://192.168.4.1 , zeby podac dane swojej sieci domowej.


JESLI NADAL NIE STARTUJE
------------------------
Jesli przy DIO 80 MHz bootloop dalej wystepuje, nastepnym krokiem jest DIO
przy 40 MHz (bezpieczniejszy timing SPI). Nie ma tego w tym pliku - napisz na
forum, zbuduje osobna wersje.


UWAGA PRZY PRZYSZLYCH AKTUALIZACJACH
------------------------------------
"Aktualizacja firmware przez WiFi" w panelu WWW pobiera standardowa
(QIO-owa) wersje firmware. Sam bootloader NIE jest przy tym ruszany, wiec
plytka nadal wystartuje w DIO - ale zeby miec pewnosc pelnej zgodnosci,
przy kolejnych wersjach lepiej poprosic o zaktualizowany plik z tego
katalogu i wgrac go ponownie przez esptool-js.
