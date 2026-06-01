# ESP32 Radio — instalator webowy (GitHub Pages)

Strona wgrywa firmware przez **ESP Web Tools** (Chrome/Edge, Web Serial).  
Flash jest **kasowany automatycznie** przy instalacji — nie zmieniaj tego w `manifest.json`.

## Wymagania sprzętowe użytkownika

- Płytka **ESP32-S3** z **16 MB** flash (np. N16R8)
- Przeglądarka **Chrome** lub **Edge** na komputerze (nie telefon)

## Publikacja nowej wersji (checklist)

Wykonuj w **repo z kodem źródłowym** (`esp32-radio`), nie w tym repo:

1. Podnieś wersję w `include/config.h` → `FIRMWARE_VERSION`
2. Zbuduj i skopiuj pliki:
   ```powershell
   .\build-full-bin.ps1
   ```
   Skrypt sam ustawia `"version"` w `installer/manifest.json` z `config.h`.
3. W **tym repo** (`installer`) commit i push na `main`:
   - `bootloader.bin`, `partitions.bin`, `boot_app0.bin`, `firmware.bin`, `littlefs.bin`
   - opcjonalnie `esp32-radio-FULL.bin` (esptool-js, offset `0x0`)
   - `manifest.json`, `index.html`
4. GitHub Actions opublikuje Pages — deploy **nie buduje** firmware, tylko weryfikuje obecność plików.

## Pliki i offsety (`manifest.json`)

| Plik | Offset (dec) | Offset (hex) |
|------|--------------|--------------|
| bootloader.bin | 0 | 0x0 |
| partitions.bin | 32768 | 0x8000 |
| boot_app0.bin | 57344 | 0xE000 |
| firmware.bin | 65536 | 0x10000 |
| littlefs.bin | 13172736 | 0xC90000 |

Zgodne z `partitions.csv` i `build-full-bin.ps1` w repo głównym.

## Dwa sposoby wgrywania

| Metoda | Plik | Uwagi |
|--------|------|--------|
| **Strona instalatora** (ta) | 5× `.bin` z manifestu | Przycisk „Zainstaluj” na Pages |
| **esptool-js** | `esp32-radio-FULL.bin` | Jeden plik, offset **0x0** |

## Rozwiązywanie problemów

- **„Pobieranie firmware” / 404** — brak `.bin` na Pages → uruchom `build-full-bin.ps1` i push do tego repo.
- **Stary panel WWW po flashu** — nie przebudowano `littlefs.bin` (`pio run -t buildfs`) po zmianie `data/`.
- **Port zajęty** — zamknij monitor szeregowy, odśwież stronę, podłącz ponownie USB.
- **Brak COM** — kabel z danymi, port UART lub USB (oba wgrywają; logi tylko UART).
