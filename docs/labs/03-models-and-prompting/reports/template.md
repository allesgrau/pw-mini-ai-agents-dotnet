# Raport — Laboratorium 03

**Zespół:** DuDi (Aleksandra Dmitruk, Barbara Gawlik, Jakub Gronkiewicz)
**Data:** 22.10.2026

## Środowisko

| Parametr | Wartość |
|----------|---------|
| Wersja Ollama | 0.40.8 |
| Sprzęt (CPU/GPU, RAM) | AMD Ryzen 7 7730U, 16GB RAM |
| Liczba dostępnych notatek | 170 |

## EXP-00 — Sposób oceny wyników

Jak oceniamy poprawność odpowiedzi modelu? Jakie metryki przyjmujemy i dlaczego?

| Metryka | Definicja |
|---------|-----------|
| Floor accuracy | Ile razy model poprawnie wybrał piętro? |
| Elevator accuracy | Czy model poprawnie identyfikował obecność windy? |
| Exact match | Czy model poprawnie określił obecność windy i piętro? |

## EXP-01 — Porównanie modeli

| Model | Floor accuracy | Elevator accuracy | Exact match | Średni czas [ms] |
|-------|----------------|-------------------|-------------|------------------|
| gemma3:4b | 0.8 | 0.6 | 0.5 | 7095 |
| qwen3:4b | 1.0 | 1.0 | 1.0 | 104168 |

**Wnioski:** W każdej kategorii zdecydowanie lepiej wypada model `qwen3:4b`, choć działa ok. 14,6x wolniej, niż `gemma3:4b`. 

## EXP-02 — Prompting

Model: gemma3:4b

| Wariant | Floor accuracy | Elevator accuracy | Exact match |
|---------|----------------|-------------------|-------------|
| zero-shot | 0.8 | 0.6 | 0.5 |
| one-shot | 1.0 | 0.9 | 0.9 |
| few-shot | 0.9 | 1.0 | 0.9 |

**Wnioski:** Dodanie choćby jednego przykładu do promptu wyraźnie poprawia wyniki `gemma3:4b`: exact match rośnie z 0.5 do 0.9. Najbardziej zyskuje rozpoznawanie windy, czyli właśnie ta część zadania, z którą model radził sobie najgorzej bez przykładów. Mały model z dobrze dobranym przykładem zbliża się więc do wyników `qwen3:4b` z EXP-01 i działa przy tym kilkanaście razy szybciej.

## EXP-03 — Temperature

Model: gemma3:4b

| Temperature | Exact match | Powtarzalne notatki |
|-------------|-------------|---------------------|
| 0.0 | 0.5 | (do uzupełnienia) |
| 0.5 | 0.5 | (do uzupełnienia) |
| 1.0 | 0.5 | (do uzupełnienia) |

**Wnioski:** Temperatura nie wpłynęła na poprawność: exact match wynosi 0.5 dla wszystkich trzech wartości. Błędy modelu wynikają więc z niezrozumienia notatek, a nie z losowości próbkowania. Podniesienie albo obniżenie temperatury tego nie naprawi, w przeciwieństwie do lepszego promptu.

## EXP-04 — Structured output — JSON i walidacja

Model: gemma3:4b

| Wariant | Poprawny JSON | Exact match |
|---------|---------------|-------------|
| prompt | 10 | 0.5 |
| json-schema | 10 | 0.5 |

**Wnioski:** Oba warianty zwróciły poprawny składniowo JSON we wszystkich 10 przypadkach, a mimo to exact match wynosi tylko 0.5. Poprawny JSON nie oznacza więc poprawnej odpowiedzi. Walidacja formatu chroni aplikację przed błędem parsowania, ale nie sprawdza, czy wartości w polach są prawdziwe.

## Najtrudniejsze notatki

Które notatki model najczęściej interpretował błędnie i dlaczego?

(do uzupełnienia)

## Rekomendacja

Który model i prompt wybieramy do projektu — i jakie mamy dowody, że to dobry wybór?

Do projektu wybieramy `gemma3:4b` z promptem one-shot, temperaturą `0.0` i odpowiedzią w trybie JSON Schema. Dowody:

* one-shot daje exact match `0.9` (EXP-02), blisko `qwen3:4b` (`1.0`);
* `gemma3:4b` odpowiada średnio w ok. `7 s`, a `qwen3:4b` w ok. `104 s` (EXP-01), co w interaktywnym agencie ma duże znaczenie;
* temperatura `0.0` daje najbardziej powtarzalne wyniki bez straty poprawności (EXP-03);
* JSON Schema gwarantuje parsowalny format (EXP-04).

Jeśli najważniejsza jest maksymalna poprawność, a czas odpowiedzi nie ma znaczenia, lepszym wyborem będzie `qwen3:4b`.