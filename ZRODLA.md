# Autorstwo i źródła

## Wkład autorów

| Autor | Wykonane elementy | Lokalizacja w projekcie |
| --- | --- | --- |
| Marcin Kowalczyk (Pixel of Software) | wybór biblioteki, założenia, przegląd i akceptacja kodu, publikacja | całość |
| Asystent AI Claude (Anthropic) | wrapper, przykład, generator wykresów, testy, dane, skrypty budowania, dokumentacja | `app/`, `przyklady/`, `tests/`, `dane/`, `build.py`, `run.py`, `README.md` |

## Wykorzystane materiały

| Materiał | Autor i źródło | Wersja lub data | Licencja | Sposób wykorzystania |
| --- | --- | --- | --- | --- |
| Cairo | Carl Worth, Behdad Esfahbod i współtwórcy, https://cairographics.org | 1.18.4 (odniesienie z karty: 1.18.0) | LGPL-2.1 albo MPL-1.1 (do wyboru) | biblioteka łączona dynamicznie, nie jest dołączona; deklaracje funkcji i wartości statusów według `cairo.h`, `cairo-svg.h`, `cairo-pdf.h` i dokumentacji https://cairographics.org/manual/ |
| `build.py`, `run.py`, `LICENSE` | Fundacja Duckling, szablon projektu z materiałów na start DuckQuest | 2026 | MIT (plik `LICENSE`) | skrypty rozbudowane o dołączanie Cairo, cel `przyklad` i kopiowanie `dane/`; licencja z dopisanym autorem |
| `app/pliki.dk` | własny moduł z projektu DuckFolio (Pixel of Software) | 2026 | MIT | odczyt pliku przez `fopen`/`fread` |
| Duckling (kompilator i biblioteka standardowa) | ducktype-org/duckling | 0.3-alpha, commit 83882805a | licencja Ducklinga | zależność budowania |

Dane w `dane/` (fikcyjna sprzedaż) i rysunki (kaczka, wykres) przygotowano dla tego projektu.

## Narzędzia AI

Kod wrappera, przykładu, generatora wykresów i testów, dane, zmiany skryptów budowania oraz dokumentację przygotowano z pomocą asystenta AI Claude (Anthropic). Autor ustalił zakres i wybór biblioteki, przejrzał wyniki, sprawdził działanie (budowa, 28 testów, przykład, generator, `leaks`, obejrzenie wykresu) i odpowiada za oddaną wersję.

## Licencja własnego projektu

MIT, treść w pliku `LICENSE` (szablon Fundacji Duckling z dopisaną linią `Copyright (c) 2026 Pixel of Software (Marcin Kowalczyk)`). Licencja Cairo dotyczy samej biblioteki, instalowanej osobno.
