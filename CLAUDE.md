# sicoin — NBP currency converter (Lab 1, Technologie Obiektowe)

University lab assignment. A desktop GUI that converts any amount from any source currency
to any target currency using NBP (National Bank of Poland) average rates (table A).
The lecturer cares most about **OOP and SOLID**; the code is kept for possible later review
and the app is only presented live on the author's laptop (no distribution needed).

Repo: https://github.com/ArziPL/sicoin

## Assignment requirements (from lab1To.pdf, not in repo)
- Decompose the problem; follow SOLID.
- Must use design patterns: **Strategy, State, Observer, Iterator**.
- Deliverables: **UML class diagram**, implementation, live presentation.
- User must be **unable to enter invalid data** (validation is a grading point).
- Ignore the PDF lines about a WGS-84 rectangle / fire stations: copy-paste from another lab.

## Data source
- URL (also in `path.txt`): https://static.nbp.pl/dane/kursy/xml/LastA.xml
- Encoding ISO-8859-2; Qt's `QXmlStreamReader` decodes it correctly (verified).
- Per `<pozycja>`: `nazwa_waluty`, `przelicznik`, `kod_waluty`, `kurs_sredni`.
  Header has `numer_tabeli`, `data_publikacji`. ~32 currencies.
- Gotchas: decimal **comma** (`3,9321`); **multiplier** (`przelicznik`, e.g. HUF ×100);
  **PLN is not in the table**, so add it with rate 1.
- Conversion via PLN: `amount * (rateFrom / multFrom) / (rateTo / multTo)`.

## Stack and build
- MSYS2 UCRT64 toolchain: GCC 16.2, Qt 6.12 (qt6-base), CMake, Ninja, gdb.
  Tools are on PATH via `C:\msys64\ucrt64\bin`; `pacman` runs from the "MSYS2 UCRT64" terminal.
- Qt modules used: Widgets (GUI), Network (`QNetworkAccessManager`), Core (`QXmlStreamReader`).
- Editor: VS Code + CMake Tools; `CMakePresets.json` has `debug` / `release`.
- Build dir is **outside OneDrive**: `%LOCALAPPDATA%/builds/sicoin/<preset>`.
- CLI: `cmake --preset debug` then `cmake --build --preset debug`. In VS Code: F5 builds and debugs.
- New `.cpp` files must be added to `qt_add_executable` in `CMakeLists.txt`.

## Working agreement
- **Simplicity above all**: as few files and as little code as possible. It's 1 of 6 small
  OOP labs; clear OOP/SOLID and the 4 patterns matter, elaborate structure is not expected.
- Go **slowly, step by step**; brainstorm and agree on a design before writing code.
- The user runs **git commands themselves**: give the commands, don't execute them.
- Prefer the most popular/standard tools and conventions; avoid convenience-only tweaks.
- Explain what each new file/class does; this is a learning project.

## Design (agreed)
```
src/
├─ main.cpp               wiring: NbpRateSource → ExchangeService → MainWindow
├─ model.h                CurrencyRate, RateTable, interfaces RateSource + RateObserver
├─ NbpRateSource.h/.cpp   download + parse NBP XML
├─ ExchangeService.h/.cpp holds table, refresh(), convert(), notifies observers
├─ AppState.h             Loading / Ready / Error
└─ MainWindow.h/.cpp      GUI
```
| Class | Role | Pattern / SOLID |
|---|---|---|
| `CurrencyRate` | struct: code, name, multiplier, rate | — |
| `RateTable` | date, number, rates; `begin()/end()` | **Iterator** (C++ idiom) |
| `RateSource` | interface `fetch() -> RateTable` | **Strategy**, DIP/OCP |
| `NbpRateSource` | only implementation (HTTP + XML) | Strategy (concrete) |
| `RateObserver` | interface `onRatesUpdated(table)` | **Observer** |
| `ExchangeService` | table, `refresh()`, `convert()`, notify | Observer subject |
| `AppState` + 3 subclasses | status text, whether form is enabled | **State** |
| `MainWindow` | form, swap, live result, rates table, refresh | Observer (concrete) |

- Qt types everywhere (`QString`, `QList`); no std/Qt mixing.
- Validation by construction: currency **combo boxes**, amount in **`QDoubleSpinBox`**
  (min/max, 2 decimals). Result updates live; no Convert button.
- If lecturer wants more: explicit iterator class (~15 lines) or a 2nd strategy (~10 lines).

## Roadmap / status
Each step: Claude writes code + explains, user runs F5, checks, and commits.
1. [x] Environment: toolchain, Git + GitHub, CMake/Qt skeleton (empty window opens on F5).
2. [x] Design agreed (see above).
3. [ ] UML class diagram, first version: Mermaid in `docs/uml.md`.
4. [ ] `model.h`: CurrencyRate, RateTable, RateSource, RateObserver (check: compiles).
5. [ ] `NbpRateSource` (check: temporary console print of the real table).
6. [ ] `ExchangeService` (check: console conversion; PLN and HUF ×100 correct).
7. [ ] `AppState` (Loading / Ready / Error).
8. [ ] `MainWindow`: full GUI; remove temporary prints.
9. [ ] Edge-case testing (no internet, PLN↔PLN, swap); final UML matches code.
10. [ ] Presentation cheat sheet: where each pattern / SOLID principle lives.

## Scope (agreed)
- In: converter form, **swap** button, **refresh** button, **rates table** view.
- Out: offline fallback / caching, anything else not listed.

## Decisions log
- C++20 + Qt 6 Widgets (not QML): pure C++, maps cleanly to the UML diagram.
- Qt installed from MSYS2 rather than the official installer: no account needed, matches GCC.
- Use Qt's own XML/network classes rather than adding pugixml/libcurl.
- Implement our own Observer interface for the domain logic (Qt signals/slots are already
  an observer, but the pattern must be visible); signals/slots only for GUI wiring.
