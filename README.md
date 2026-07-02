# Zoo Opisy 🐾 — Generator opisów produktów zoologicznych

Webowa aplikacja, która **przepisuje** opisy produktów zoologicznych z eksportu CSV BaseLinker. Nowy opis powstaje w 100% na bazie starego (te same fakty), ale przerobiony wg promptu copywriterskiego (Zooplus/MaxiZoo). **Zdjęcia ze starego opisu są zachowane** i wplecione w nowy tekst. Nadpisywana jest tylko kolumna `opis` (kolumna C).

## Jak działa

1. Wgrywasz CSV BaseLinker (separator `,` lub `;`, UTF-8).
2. Dla każdego wiersza aplikacja czyta stary `opis` (kolumna C) i:
   - wyciąga **nazwę produktu** z `<h1>` (lub `<h2>`)
   - wyciąga **wszystkie zdjęcia** (`<img src>` z niepustym adresem)
   - wyciąga **tekst źródłowy** (fakty produktu)
3. Wysyła do AI (domyślnie DeepSeek) prompt copywriterski + stary tekst + liczbę zdjęć.
4. AI pisze nowy opis wg struktury (nagłówek, wstęp, korzyści ✅, bezpieczeństwo, wskazówki, FAQ) i wstawia znaczniki `[[IMG_1]]`…`[[IMG_N]]`.
5. Aplikacja zamienia znaczniki na **prawdziwe zdjęcia** ze starego opisu, owija tekst w sekcje platformy i **dopina na końcu każde zdjęcie, którego AI nie użyło** — dzięki temu żadne zdjęcie nie ginie.
6. Pobierasz nowy CSV z nadpisaną kolumną `opis` (reszta bez zmian).

## Gwarancja zachowania zdjęć

Zdjęcia NIE są przekazywane do AI — są wyciągane deterministycznie z kodu i wstawiane z powrotem 1:1 (te same URL-e). AI decyduje tylko *gdzie* je umieścić przez znaczniki; jeśli któreś pominie, trafia na koniec opisu. W logu widać `zdjec X/Y` i ile zostało dopiętych.

## Funkcje

- **BYOK** — klucz API wpisujesz w przeglądarce, nie opuszcza komputera
- **3 dostawcy** — DeepSeek (domyślny), Google Gemini, Anthropic Claude
- **Concurrency** — do 20 równoległych requestów (default 5)
- **Retry** — automatyczny przy 429/529/5xx z exponential backoff
- **Anulowanie** — przerwij w trakcie, pobierz częściowy wynik
- **UTF-8 BOM** — wynikowy CSV otwiera się poprawnie w Excelu
- **Autodetekcja separatora** — `,` lub `;`

## Format wejściowego CSV

- Separator: `,` lub `;` (autodetekcja)
- Kodowanie: UTF-8
- **Wymagana kolumna:** `opis` (kolumna C — stary opis HTML ze zdjęciami)
- Pozostałe kolumny (`produkt_ean`, `produkt_sku`, `zdjecie*`, `opis_dodatkowy_*` itd.) zachowywane bez zmian

## Klucz API DeepSeek

https://platform.deepseek.com/api_keys — wymaga kredytu na koncie (brak darmowego API). Model domyślny: `deepseek-v4-flash` (najtańszy, szybki).

## Użycie lokalne

Statyczna strona — bez build, bez npm.

```bash
npx serve .
# lub otwórz index.html w przeglądarce
```

## Deployment na Vercel

1. https://vercel.com/new → wybierz repo `zoo-opisy`
2. Vercel wykryje statyczny site → **Deploy**
3. URL: `https://zoo-opisy.vercel.app`

Brak zmiennych środowiskowych — klucz API wpisywany w przeglądarce.

## Edycja promptu

Cały prompt w [`prompt.js`](./prompt.js) jako stała `PROMPT_MASTER`. Tam też funkcje `parseOldDescription` (ekstrakcja nazwy/tekstu/zdjęć) i `buildPrompt`.

## Stack

- Vanilla JavaScript (bez frameworka)
- PapaParse 5.x (CSV) + natywny DOMParser (ekstrakcja zdjęć)
- DeepSeek / Gemini / Claude API (BYOK)
- Hosting: Vercel static
