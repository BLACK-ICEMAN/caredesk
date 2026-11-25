# CareDesk – Build & Packaging Guide

## Project structure
```
CareDesk/
├── models.h        structs, limits, extern global arrays
├── validation.h/.c text, date, age, weekday helpers (no GTK)
├── storage.h/.c    defines global arrays; save/load .dat files
├── seed.h/.c       demo (sample) data loader
├── logic.h/.c      ALL rules: register, book, admit, bill, discount, report (no GTK)
├── ui.h/.c         GTK 3 window, tabs, dialogs, CSS
├── main.c          gtk_init -> load -> window -> gtk_main -> save
├── Makefile
└── tests/test_logic.c   console tests of logic.c (26 checks)
```
**Login:** first run user is `admin` / `admin123` (change it in the Account tab). Accounts are saved in `users.dat`.

Dependency direction: `ui -> logic -> validation`, and `logic/ui -> storage -> models`. Only `ui.c` and `main.c` know GTK.

## Demo data for your teacher
On the first login the program asks **"Load sample (demo) data?"** - click **Yes**. It creates 5 doctors, 10 patients, 16 appointments, 11 medical records, 4 admissions (2 still admitted) and 5 bills (paid and unpaid, one with a discount), plus a Staff login `nurse / nurse123`. You can also use **Account tab -> Load Demo Data** (Admin only, empty system only).
To start again from zero: close the program, delete all `.dat` files except `users.dat`, reopen.
Tip: load the demo data a few minutes before the presentation, because the "today" and "this month" figures are based on the date you load it.

## A. Build on Windows (MSYS2) – recommended
1. Install MSYS2 from https://www.msys2.org and open **"MSYS2 MINGW64"** (not the purple/UCRT one).
2. Install tools:
   ```
   pacman -Syu
   pacman -S mingw-w64-x86_64-gcc mingw-w64-x86_64-gtk3 mingw-w64-x86_64-pkg-config make
   ```
3. Build and run:
   ```
   cd /c/path/to/CareDesk
   make
   ./caredesk.exe
   ```
   (Windows-only extra: add `-mwindows` to `LIBS` in the Makefile to hide the black console window.)

## B. Build on Linux (Ubuntu/Debian)
```
sudo apt install build-essential libgtk-3-dev pkg-config
make && ./caredesk
```

## C. Run the console tests (optional, good for your report)
```
gcc -o test_logic tests/test_logic.c logic.c validation.c storage.c seed.c
./test_logic
```
(Run it in an empty folder – it creates no files, but the app's `.dat` files would be read if present.)

## D. Make the .exe run on any PC (no compiler, no MSYS2)
Do this in the **MINGW64** shell after `make`.

1. Create a folder: `mkdir -p dist/bin && cp caredesk.exe dist/bin/`
2. Copy every DLL the exe needs:
   ```
   ldd caredesk.exe | grep -i mingw64 | awk '{print $3}' | xargs -I{} cp {} dist/bin/
   ```
   (`ldd` lists the DLLs; this copies the ones from `/mingw64/bin`.)
3. Copy GTK's runtime resources (icons, schemas, image loaders, theme):
   ```
   mkdir -p dist/share dist/lib
   cp -r /mingw64/share/glib-2.0   dist/share/
   cp -r /mingw64/share/icons      dist/share/
   cp -r /mingw64/share/themes     dist/share/ 2>/dev/null
   cp -r /mingw64/lib/gdk-pixbuf-2.0 dist/lib/
   ```
4. Test on the same machine: open `dist/bin/caredesk.exe` by double-click.
   Real test: copy `dist` to a PC (or a clean Windows VM) that has no MSYS2. If a DLL error appears, run `ldd` again on that missing name and copy it too.
5. Hand-in options:
   - **Zip**: zip the `dist` folder; the user double-clicks `bin\caredesk.exe`.
   - **Installer** (optional): install Inno Setup, create a script that copies `dist\*` to `{app}` and adds a Start-menu shortcut to `{app}\bin\caredesk.exe`.

Data files (`patients.dat`, `bills.dat`, ...) are created in the folder the program is started from. Keep the exe's folder writable (not `C:\Program Files` without admin rights – or put a shortcut with "Start in" set to a user folder).

## E. Common problems
| Problem | Fix |
|---|---|
| `pkg-config: gtk+-3.0 not found` | You are in the wrong MSYS2 shell, or gtk3 not installed (step A2). |
| Exe closes instantly outside MSYS2 | A DLL is missing – repeat step D2/D3. |
| Squares instead of icons | Copy `share/icons` (step D3). |
| Rows not coloured | Click a row to deselect; selected rows use the theme colour. |
