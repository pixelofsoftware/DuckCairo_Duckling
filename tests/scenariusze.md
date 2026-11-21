# Scenariusze sprawdzenia

Próby z 2026-11-21 na macOS 27.2 (Apple Silicon), `duckc 0.3-alpha (83882805a 2026-11-20)`, Cairo 1.18.4 (Homebrew), wariant natywny, po `python3 build.py --target all --output build`.

| Scenariusz | Dane lub czynności | Oczekiwany wynik | Komenda lub procedura | Otrzymany wynik |
| --- | --- | --- | --- | --- |
| Znane piksele | scena 10x10: białe tło, czerwony kwadrat (2,2)-(6,6), czarna linia grubości 2 na y = 8 | (3,3) = `0xFFFF0000`; (1,1), (6,3) białe; (5,5) czerwony; wiersze 7 i 8 czarne, 6 i 9 białe | `run.py --target tests` | zgodny - OK |
| Układ pikseli | ta sama scena | krok 40 B; w pamięci B, G, R, A (little-endian) | testy | zgodny - OK |
| Przemnożenie przez alpha | niebieski z alpha 0,5 na przezroczystym tle | surowo A = 128, B = 128; `kolorPiksela` = (0, 0, 255, 128) | testy | zgodny - OK |
| PNG: zapis i odczyt | scena zapisana do `test-scena.png` i wczytana | 10x10, RGB24 (alpha 255), znane piksele; sygnatura PNG | testy, przykład | zgodny - OK |
| Ta sama scena w SVG i PDF | `test-scena.svg`, `test-scena.pdf`; kaczka w przykładzie | po zamknięciu: kompletny SVG (`viewBox 0 0 10 10`, czerwone wypełnienie), PDF `%PDF-` … `%%EOF` | testy, przykład | zgodny - OK |
| Błędne wymiary | obraz 0x10, -5x5, 40000x10, 32000x32000; SVG 0 pt; PDF nieskończony i 20000 pt | `BLEDNE_WYMIARY` bez wywołania Cairo | testy, przykład | zgodny - OK |
| Statusy Cairo | SVG i PNG w nieistniejącym katalogu; brakujący PNG | 11 `error while writing to output stream`; 18 `file not found` | testy, generator | zgodny - OK |
| Zamknięte obiekty | operacje po `zamknijKontekst` i `zamknijPowierzchnie`, podwójne zamknięcie | `false` / `ZAMKNIETY` | testy, przykład | zgodny - OK |
| Kontekst po zakończeniu powierzchni | rysowanie kontekstem po zamknięciu SVG | 12 `the target surface has been finished`, trwały status | testy | zgodny - OK |
| Zwalnianie | testy (w tym 300 cykli obraz-PNG-PDF) pod `leaks --atExit` | 0 wycieków | `cd build && leaks --atExit -- ./tests-native/package_llvm.exe` | `0 leaks for 0 total leaked bytes` - OK |
| Kontrola ujemna | kopia bez `cairo_surface_destroy` w `zamknijPowierzchnie` | `leaks` wykrywa wycieki | jak wyżej, na zmienionej kopii | `609 leaks for 272832 total leaked bytes` - sprawdzenie działa |
| Praktyczne zastosowanie | generator na `dane/sprzedaz.csv` | `wykres.png` 640x400 z kolorem pierwszego słupka (242,201,76), kompletne `wykres.svg` i `wykres.pdf`; wykres czytelny (obejrzany) | `printf 'dane/sprzedaz.csv\nwykres\nSprzedaż kaczek gumowych 2026\n' \| python3 run.py --build build --workdir .` | zgodny, kod 0 - OK |
| Złe dane i błędy zapisu | `dane/zle.csv`; `brak.csv`; nazwa w nieistniejącym katalogu | kod 2 (`wiersz 2: niepoprawna wartosc 'dużo'`); kod 3; kod 5 ze statusem dla każdego pliku | generator | zgodny - OK |
| Testy | 28 sprawdzeń | `Bledy: 0`, kod 0 | `python3 run.py --build build --target tests` | `Bledy: 0` - OK |
| Odtworzenie w nowym katalogu | świeży klon repozytorium | budowa, przykład, generator i testy zgodne z README | `git clone`, komendy z README | macOS: OK; Linux x86_64 (Ubuntu 24.04, Cairo 1.18.0): nie sprawdzono |
