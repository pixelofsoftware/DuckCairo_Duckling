# DuckCairo: wrapper Cairo dla Ducklinga

![Napisane w Duckling](https://img.shields.io/badge/napisane%20w-Duckling-f2c94c)

Zespół: Pixel of Software (Marcin Kowalczyk). Temat: P02 DuckBridge, biblioteka Cairo (poziom 3). Skład: 1 osoba.

Moduł `cairo` (`app/cairo.dk`) rysuje poza ekranem: na obrazie w pamięci (piksele, zapis PNG) i na powierzchniach SVG oraz PDF. Ma publiczne typy `Powierzchnia`, `Kontekst`, `Status` i `Piksele`, sprawdza wymiary przed wywołaniem biblioteki i opisuje każdy błąd statusem Cairo. Użycie zamkniętego obiektu jest odrzucane. Wrapper nie potrzebuje adaptera C, okna, GPU ani funkcji zwrotnych.

Całe minimum karty Cairo działa i jest sprawdzone testami. Do tego są dwa przykłady (podstawowy i generator wykresów słupkowych z CSV do PNG, SVG i PDF) oraz rozszerzenia:
* okręgi i dowolne ścieżki;
* napisy UTF-8;
* odczyt PNG;
* odwracanie przemnożenia przez alpha przy odczycie pikseli.

## Uruchomienie

Wymagane są:
* Python 3;
* Duckling (`duckc` w `PATH`, instalacja według tutoriala);
* Cairo z obsługą PNG, SVG i PDF.

| System | Instalacja Cairo |
| --- | --- |
| Ubuntu | `sudo apt install libcairo2` |
| macOS | `brew install cairo` |

Z katalogu projektu:

```bash
python3 build.py --target all --output build      # aplikacja, przykład i testy; kopiuje dane/ do build/
python3 run.py --build build --target tests       # testy: PASS/FAIL, kod = liczba błędów
python3 run.py --build build --target przyklad    # przykład: przyklad.png, .svg, .pdf w katalogu budowy
printf 'dane/sprzedaz.csv\nwykres\nSprzedaż kaczek gumowych 2026\n' | python3 run.py --build build --workdir .
```

Generator wykresów czyta trzy wiersze ze standardowego wejścia:
1. plik CSV, w którym każdy wiersz to `etykieta;wartość` (wiersze z `#` są pomijane, przecinek dziesiętny jest dozwolony);
2. nazwę plików wynikowych bez rozszerzenia;
3. tytuł.

Rysuje tę samą scenę 640 x 400 trzy razy i zapisuje `wykres.png`, `wykres.svg` oraz `wykres.pdf`. Scena ma tytuł, siatkę, osie, kolorowe słupki i podpisy z polskimi literami. Potem sprawdza wyniki: wczytuje PNG i porównuje kolor pierwszego słupka, a w SVG i PDF sprawdza początek i koniec pliku. Kody zakończenia:

| Kod | Znaczenie |
| --- | --- |
| 0 | sukces |
| 2 | złe dane albo parametry |
| 3 | błąd odczytu CSV |
| 5 | błąd zapisu albo niezgodność |

`build.py` szuka biblioteki w tej kolejności:
1. ścieżka z opcji `--cairo /sciezka/libcairo.so.2`;
2. `pkg-config cairo`;
3. typowe miejsca: `/usr/lib/x86_64-linux-gnu/libcairo.so.2` oraz `/opt/homebrew/lib/libcairo.2.dylib`;
4. `ctypes.util.find_library`.

`build.py` i `run.py` powstały z szablonu konkursu: doszła obsługa biblioteki, cel `przyklad` i kopiowanie `dane/` do katalogu budowy.

## Funkcje i demonstracja

| Funkcja z minimum karty | Sposób uruchomienia | Oczekiwany rezultat |
| --- | --- | --- |
| Powierzchnia obrazu i kontekst; kolor, prostokąt, wypełnienie, linie, grubość | przykład | `Obraz 200x120: rysowanie true`; tułów `(242,201,76,255)`, woda `(70,130,180,255)` |
| Zapis PNG i ta sama scena na SVG i PDF z poprawnym zakończeniem | przykład; generator | `przyklad.svg: no error has occurred; przyklad.pdf: no error has occurred`; generator: `kompletny SVG - true`, `kompletny PDF - true` |
| Typy publiczne, kontrola wymiarów, opisy statusów, odrzucanie zamkniętych obiektów | przykład, testy | `Obraz 0x50: niepoprawne wymiary powierzchni`; `SVG w nieistniejacym katalogu: kod 11: error while writing to output stream`; `Wypelnienie zamknietym kontekstem: obiekt zostal juz zamkniety` |
| Własność kontekstu i powierzchni; układ pikseli (krok, kolejność kolorów i bajtów, przemnożenie) | sekcje poniżej, testy | czerwony piksel surowo `4294901760` (`0xFFFF0000`), w pamięci bajty B, G, R, A; półprzezroczysty niebieski zapisany jako alpha 128, niebieski 128 |
| Test znanych pikseli, odczyt zapisanych wyników, błąd wymiarów | `run.py --target tests` | 28 sprawdzeń, `Bledy: 0` |
| Praktyczne zastosowanie | generator na `dane/sprzedaz.csv` | `8 slupkow`, trzy pliki, sprawdzenie zgodne, kod 0 |

**Demonstracja:**
1. Uruchom przykład i otwórz `build/przyklad.png`, `.svg` oraz `.pdf`. To ta sama kaczka narysowana trzema metodami.
2. Uruchom generator na `dane/sprzedaz.csv` i otwórz `wykres.png`, `wykres.svg` oraz `wykres.pdf`.
3. Spróbuj `dane/zle.csv`: dostaniesz `wiersz 2: niepoprawna wartosc 'dużo'` i kod 2.
4. Podaj nazwę wynikową w nieistniejącym katalogu, np. `brak_katalogu/wykres`. Dla każdego pliku zobaczysz status Cairo `error while writing to output stream` i kod 5.

## Typy i API modułu `cairo`

| Funkcja | Zwraca | Błąd |
| --- | --- | --- |
| `obraz(szer, wys) -> Powierzchnia` | obraz ARGB32 w pikselach, przezroczysty | `status.kod = BLEDNE_WYMIARY`, gdy bok spoza 1..32767 albo bufor ponad 256 MiB (bez wywołania Cairo) |
| `svg(sciezka, szer, wys)`, `pdf(sciezka, szer, wys) -> Powierzchnia` | powierzchnia wektorowa w punktach, plik kompletny po zamknięciu | `BLEDNE_WYMIARY` (≤ 0, NaN, nieskończoność, > 14400 pt), status Cairo (np. 11 `WRITE_ERROR`) |
| `wczytajPng(sciezka) -> Powierzchnia` | obraz z pliku PNG | status Cairo (np. 18 `FILE_NOT_FOUND`) |
| `kontekst(&p) -> Kontekst` | kontekst rysowania | `ZAMKNIETY` dla zamkniętej powierzchni |
| `kolor(&k, r, g, b, a)` (0..1), `kolor255`, `gruboscLinii`, `prostokat`, `przesun`, `liniaDo`, `linia`, `okrag`, `zamknijSciezke` | `bool` | `false` dla zamkniętego kontekstu albo złych argumentów (kolor poza 0..1, grubość ≤ 0, NaN) |
| `wypelnij`, `wypelnijZachowaj`, `obrysuj`, `wyczysc`, `tekst` | `Status` | status Cairo po operacji albo `ZAMKNIETY` / `BLEDNE_PARAMETRY` |
| `zapiszPng(&p, sciezka) -> Status` | zapis obrazu do PNG | `ZLY_RODZAJ` dla SVG/PDF, status Cairo |
| `pobierzPiksele(&p) -> Piksele` | kopia pikseli z krokiem wiersza | `ZLY_RODZAJ`, `ZAMKNIETY` |
| `surowy(&px, x, y) -> i64`, `kolorPiksela(&px, x, y) -> Kolor` | `0xAARRGGBB` przemnożone przez alpha; składowe 0..255 po odwróceniu przemnożenia | -1 / `(0,0,0,0)` poza obrazem |
| `stan(&k)`, `zamknijKontekst(&k)`, `zamknijPowierzchnie(&p)`, `opisStatusu(kod)`, `wersjaBiblioteki()` | stan, zamykanie, opisy | `ZAMKNIETY` / `false` przy ponownym zamknięciu |

`Status` ma pola `ok`, `kod` i `opis`:
* kody od 0 wzwyż to `cairo_status_t`, a opis pochodzi z `cairo_status_to_string`, np. 11 `error while writing to output stream` albo 12 `the target surface has been finished`;
* kody ujemne nadaje wrapper: `ZAMKNIETY`, `BLEDNE_WYMIARY`, `BLEDNE_PARAMETRY`, `ZLY_RODZAJ`.

Błąd zapisany w kontekście Cairo jest trwały: kolejne operacje na tym kontekście kończą się tym samym statusem, który zwraca też `stan(&k)`.

## Własność: kto zwalnia kontekst i powierzchnię

Według zasad Cairo każde `*_create` daje obiekt z jedną referencją, którą trzeba oddać przez `*_destroy`. Wrapper trzyma dokładnie jedną referencję w każdym obiekcie:

| Obiekt | Tworzy | Zwalnia |
| --- | --- | --- |
| `Powierzchnia` | `obraz`, `svg`, `pdf`, `wczytajPng` | `zamknijPowierzchnie`: `cairo_surface_finish` (dla SVG/PDF zapisuje i zamyka plik), potem `cairo_surface_destroy` |
| `Kontekst` | `kontekst(&p)` przez `cairo_create`, które **dodaje własną referencję do powierzchni** | `zamknijKontekst`: `cairo_destroy` |
| obiekt błędu Cairo (nieudane `*_create`) | Cairo | wrapper od razu przez `*_destroy`; użytkownik dostaje tylko `Status` |

Kolejność zamykania jest dowolna, bo kontekst utrzymuje swoją powierzchnię przy życiu. Plik SVG/PDF jest jednak kompletny dopiero po `zamknijPowierzchnie`, a rysowanie kontekstem po zakończeniu powierzchni daje status 12 `SURFACE_FINISHED`. Po zamknięciu wrapper odrzuca operacje bez wywoływania biblioteki: `false` albo `ZAMKNIETY`, a ponowne zamknięcie zwraca `false` lub `ZAMKNIETY`. Obiektów nie należy kopiować przez `copy`, bo kopia wskazywałaby ten sam obiekt Cairo. Program testów pod `leaks --atExit` na macOS zgłasza 0 wycieków. Kontrola ujemna, czyli wersja bez `cairo_surface_destroy`, daje 609 wycieków.

## Układ pikseli

`pobierzPiksele` wywołuje `cairo_surface_flush` i kopiuje dane obrazu do `List[u8]`:

| Cecha | Ustalenie |
| --- | --- |
| Format | `CAIRO_FORMAT_ARGB32`; obraz z nieprzezroczystego PNG ma `CAIRO_FORMAT_RGB24` (ten sam układ, górny bajt nieużywany, `przezroczystosc = false`, odczyt daje alpha 255) |
| Wiersze | od góry; wiersz zajmuje `krok` bajtów (stride z `cairo_image_surface_get_stride`) i może być dłuższy niż `szerokosc * 4`; dane mają `krok * wysokosc` bajtów |
| Piksel | 32-bitowa liczba `0xAARRGGBB` w **natywnej kolejności bajtów** procesora; na x86_64 i arm64 (little-endian) w pamięci leżą kolejno **B, G, R, A** |
| Przezroczystość | kolor jest **przemnożony** przez alpha (premultiplied): R, G, B ≤ A; niebieski z alpha 0,5 to bajty B = 128, A = 128 |
| Piksel (x, y) | zaczyna się na bajcie `y * krok + x * 4` |

`surowy(&px, x, y)` składa bajty w liczbę według rzeczywistej kolejności bajtów procesora (sprawdzanej w czasie działania). `kolorPiksela` dodatkowo odwraca przemnożenie i zwraca składowe 0..255.

## Środowisko i zależności

| Element | Wartość |
| --- | --- |
| Kompilator | `duckc 0.3-alpha (83882805a 2026-11-20)`, `commit-hash: 83882805a` |
| Uruchomienie | natywnie |
| System operacyjny | macOS 27.2, Apple Silicon (arm64) |
| Zależności | Cairo 1.18.4 (Homebrew, `/opt/homebrew/lib/libcairo.2.dylib`, z PNG, SVG, PDF; czcionki przez FreeType/fontconfig), Python 3 |

Wersja odniesienia z karty to Cairo 1.18.0. Użyte funkcje i wartości statusów są w 1.18.x takie same. Cairo 1.18 zapisuje wymiary SVG bez jednostki (`width="10"`), a starsze wersje z jednostką `pt`. Testy sprawdzają `viewBox`, więc działają w obu przypadkach. Na Linuksie x86_64 (Ubuntu 24.04, `libcairo.so.2`) projektu nie sprawdzałem.

## Struktura i działanie

| Plik | Odpowiedzialność |
| --- | --- |
| `app/cairo.dk` | wrapper: deklaracje C, typy, wymiary, powierzchnie, kontekst, rysowanie, statusy, piksele |
| `app/app.dk` | praktyczne zastosowanie: generator wykresów słupkowych z CSV do PNG, SVG i PDF |
| `app/pliki.dk` | odczyt pliku przez `fopen`/`fread` (sprawdzanie zapisanych wyników) |
| `przyklady/podstawowy.dk` | przykład podstawowy (kaczka na trzech powierzchniach) |
| `tests/testy.dk` | testy pikseli, plików, błędów i zwalniania |
| `dane/` | `sprzedaz.csv` (8 miesięcy), `zle.csv` (niepoprawna wartość) |

**Deklaracje C** są ręczne, według `cairo.h`, `cairo-svg.h` i `cairo-pdf.h`. Wskaźniki `cairo_surface_t*` i `cairo_t*` są przechowywane jako `u64`, a współrzędne i kolory przekazywane jako `f64` (`double`). Adapter C nie jest potrzebny, bo nie czytamy pól struktur Cairo. Wariant DVM nie jest obsługiwany. Napisy używają prostego API tekstu Cairo (`cairo_select_font_face("sans-serif")`, `cairo_show_text`), więc kształt liter zależy od czcionek systemu. Testy nie sprawdzają pikseli tekstu.

**Ograniczenia:**
* jedna strona PDF;
* brak gradientów, obrazów jako źródła i przekształceń (`cairo_scale`, `cairo_rotate`);
* brak zaawansowanego składu tekstu (Pango).

## Sprawdzenie projektu

```bash
python3 run.py --build build --target tests      # 28 sprawdzeń
python3 run.py --build build --target przyklad   # kod 0 = wszystkie oczekiwania przykładu spełnione
```

Testy sprawdzają:
* znane piksele sceny 10x10: kolor kwadratu, krawędzie wyrównane do pikseli, linia grubości 2 na dwóch wierszach;
* krok wiersza, kolejność bajtów w pamięci i przemnożenie przez alpha;
* zapis PNG i jego ponowny odczyt (RGB24) ze znanymi pikselami oraz sygnaturę pliku;
* tę samą scenę w SVG (kompletny plik, czerwone wypełnienie) i PDF (`%PDF-` … `%%EOF`);
* błędne wymiary obrazu i stron SVG/PDF: 0, ujemne, ponad 32767, ponad limit pamięci, nieskończoność, ponad 14400 pt;
* statusy Cairo: zapis do nieistniejącego katalogu, brak pliku PNG, rysowanie po zakończeniu powierzchni;
* złe kolory i grubość;
* zamknięte obiekty i podwójne zamknięcie;
* piksele i PNG wyłącznie dla obrazu;
* 300 cykli obraz–PNG–PDF.

Scenariusze są w [tests/scenariusze.md](tests/scenariusze.md).

## Autorstwo i materiały

Kod i dokumentacja: Pixel of Software (Marcin Kowalczyk), z pomocą asystenta AI. Szczegóły są w [ZRODLA.md](ZRODLA.md).

Licencja własnego kodu: MIT (plik `LICENSE`). Cairo można stosować na warunkach LGPL-2.1 albo MPL-1.1. Biblioteka nie jest dołączona do repozytorium; wrapper łączy się z nią dynamicznie, więc warunki LGPL dla łączenia dynamicznego są zachowane.
