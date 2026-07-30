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
| littlefs.bin | 13172736 | 0xC90000 |

Zgodne z `partitions.csv` i `build-full-bin.ps1` w repo głównym.

## Dwa sposoby wgrywania

| Metoda | Plik | Uwagi |
|--------|------|--------|
| **Strona instalatora** (ta) | 5× `.bin` z wybranego manifestu | Przełącznik wariantu + przycisk „Zainstaluj” na Pages |
| **esptool-js** | `esp32-radio-FULL.bin` / `esp32-radio-FULL-ili9488.bin` | Jeden plik, offset **0x0** |

## Rozwiązywanie problemów

- **„Pobieranie firmware” / 404** — brak `.bin` na Pages → uruchom `build-full-bin.ps1` i push do tego repo.
- **Stary panel WWW po flashu** — nie przebudowano `littlefs.bin` (`pio run -t buildfs`) po zmianie `data/`.
- **Port zajęty** — zamknij monitor szeregowy, odśwież stronę, podłącz ponownie USB.
- **Brak COM** — kabel z danymi, port UART lub USB (oba wgrywają; logi tylko UART).
