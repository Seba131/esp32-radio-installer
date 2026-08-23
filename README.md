# ESP32 Radio — instalator webowy (GitHub Pages)

Strona wgrywa firmware przez **ESP Web Tools** (Chrome/Edge, Web Serial).  
Flash jest **kasowany automatycznie** przy instalacji — nie zmieniaj tego w manifestach.

Obsługuje **dwa warianty wyświetlacza** (przełącznik na stronie, nad przyciskiem
„Zainstaluj"): ST7789/ILI9341 (`manifest.json`, domyślny — zachowuje stare nazwy
plików dla zgodności z istniejącymi urządzeniami/OTA) i ILI9488 (`manifest-ili9488.json`).
Oba warianty współdzielą `bootloader.bin`/`partitions.bin`/`boot_app0.bin`/`littlefs.bin`
(ten sam `partitions.csv` i ta sama strona WWW) — różni się tylko `firmware*.bin`
(inny skompilowany sterownik TFT_eSPI, patrz `platformio.ini` w repo głównym).

## Wymagania sprzętowe użytkownika

- Płytka **ESP32-S3** z **16 MB** flash (np. N16R8)
- Przeglądarka **Chrome** lub **Edge** na komputerze (nie telefon)

## Publikacja nowej wersji (checklist)

Wykonuj w **repo z kodem źródłowym** (`esp32-radio`), nie w tym repo:

1. Podnieś wersję w `include/config.h` → `FIRMWARE_VERSION`
2. Zbuduj i skopiuj pliki (buduje OBA warianty):
   ```powershell
   .\build-full-bin.ps1
   ```
   Skrypt sam ustawia `"version"`/`"build_date"` w obu manifestach z `config.h`.
3. W **tym repo** (`installer`) commit i push na `main`:
   - `bootloader.bin`, `partitions.bin`, `boot_app0.bin`, `littlefs.bin` (wspólne)
   - `firmware.bin`, `firmware-ili9488.bin` (osobne per wariant)
   - opcjonalnie `esp32-radio-FULL.bin`, `esp32-radio-FULL-ili9488.bin` (esptool-js, offset `0x0`)
   - `manifest.json`, `manifest-ili9488.json`, `index.html`
4. GitHub Actions opublikuje Pages — deploy **nie buduje** firmware, tylko weryfikuje obecność plików (obu wariantów).

## Pliki i offsety (manifesty)

| Plik | Offset (dec) | Offset (hex) |
|------|--------------|--------------|
| bootloader.bin | 0 | 0x0 |
| partitions.bin | 32768 | 0x8000 |
| boot_app0.bin | 57344 | 0xE000 |
| firmware.bin / firmware-ili9488.bin | 65536 | 0x10000 |
| littlefs.bin | 13238272 | 0xCA0000 |

Zgodne z `partitions.csv` i `build-full-bin.ps1` w repo głównym.

## Dwa sposoby wgrywania

| Metoda | Plik | Uwagi |
|--------|------|--------|
| **Strona instalatora** (ta) | 5× `.bin` z wybranego manifestu | Przełącznik wariantu + przycisk „Zainstaluj” na Pages |
| **esptool-js** | `esp32-radio-FULL.bin` / `esp32-radio-FULL-ili9488.bin` | Jeden plik, offset **0x0** |

## Zywe ogloszenie w panelu WWW radia (notice.json)

`notice.json` w tym repo jest pobierany przez KAZDE juz dzialajace radio przy
wejsciu w zakladke "Firmware" panelu WWW (patrz `checkForNotice()` w
`data/index.html` w repo glownym). Pozwala ostrzec/poinformowac wszystkich
uzytkownikow (np. "nie aktualizuj przez WiFi z wersji starszej niz X", "znany
problem ze stacja Y") **bez wydawania nowego firmware** - wystarczy edytowac
ten plik i zrobic push na `main`, Pages opublikuje zmiane automatycznie.

**Dziala tylko na urzadzeniach, ktore maja JUZ wgrane firmware z tą funkcją**
(od wersji, w ktorej zostala dodana) - nie da sie tym ostrzec urzadzen na
STARSZYM firmware, ktore jeszcze nie znaja tego mechanizmu.

Format:
```json
{
  "enabled": true,
  "severity": "warning",
  "title": "Tytul (opcjonalnie)",
  "text": "Tresc z **pogrubieniem** gdzie trzeba.",
  "color": ""
}
```
- `enabled` — `false` lub usuniecie pliku = ogloszenie ukryte (domyslny stan). Uwaga: musi to byc **wartosc logiczna bez cudzyslowu** (`false`/`true`), nie tekst `"false"`/`"true"` — inaczej panel po cichu potraktuje to jako wylaczone.
- `severity` — `"info"` (niebieski) / `"warning"` (pomaranczowy) / `"critical"` (czerwony) — ustawia kolor ramki, tytulu i tresci ORAZ delikatny odcien tla. Brak pola lub literowka (np. `"Warning"` z wielkiej litery) = domyslnie **warning** (pomaranczowy), zeby bledny wpis w tym pliku sam rzucal sie w oczy zamiast cicho wygladac jak zwykla informacja.
- `color` — opcjonalny wlasny kolor hex (np. `"#ff5252"`, dlugosc 3/4/6/8 cyfr), nadpisuje kolor ramki/tytulu/tresci z `severity`. **Tlo** (delikatny odcien) nadal pochodzi z `severity`, wiec wlasny `color` warto dobrac tak, zeby byl czytelny na tym tle (np. przy `severity:"critical"` i wlasnym niebieskim `color` tlo pozostanie czerwonawe).
- `title`/`text` — zwykly string (jeden jezyk) **albo** obiekt per-jezyk, np. `{"pl":"...", "en":"...", "fr":"...", "uk":"..."}` — panel dobierze wersje zgodna z aktualnie wybranym jezykiem strony, z fallbackiem na PL/EN.
- W tekscie dziala TYLKO `**pogrubienie**` i nowa linia (`\n`) — reszta (np. `<script>`) jest zawsze bezpiecznie escapowana, nie da sie wstrzyknac HTML/JS nawet przez zepsuty/zlosliwy plik.
- Panel odpytuje ten plik na nowo przy **kazdym** wejsciu w zakladke "Firmware" (nie tylko raz na cala sesje strony) — zmiana/wylaczenie ogloszenia jest widoczne juz przy nastepnym przelaczeniu zakladki, bez potrzeby odswiezania (F5) calej strony.

## Rozwiązywanie problemów

- **„Pobieranie firmware” / 404** — brak `.bin` na Pages → uruchom `build-full-bin.ps1` i push do tego repo.
- **Stary panel WWW po flashu** — nie przebudowano `littlefs.bin` (`pio run -t buildfs`) po zmianie `data/`.
- **Port zajęty** — zamknij monitor szeregowy, odśwież stronę, podłącz ponownie USB.
- **Brak COM** — kabel z danymi, port UART lub USB (oba wgrywają; logi tylko UART).
