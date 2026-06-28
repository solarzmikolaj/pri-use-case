# Programowanie Funkcyjne w Projekcie Biblioteki (Wersja Hybrydowa OOP/FP)
## Przewodnik i Notatki na Obronę Projektu

Ten dokument stanowi kompleksowe podsumowanie i wyjaśnienie aspektów **programowania funkcyjnego (Functional Programming - FP)** zastosowanych w kodzie [library.py](file:///c:/Users/Amelka%20i%20Zuzia/Desktop/BigData/big-data-library-functional/library.py). Został przygotowany w taki sposób, aby ułatwić powtórzenie materiału i precyzyjne odpowiadanie na pytania promotora lub recenzenta podczas obrony pracy.

---

## 1. Wprowadzenie: Paradygmat Funkcyjny w Pythonie

Python jest językiem **wieloparadygmatowym** (multi-paradigm). Oznacza to, że pozwala na łączenie różnych stylów programowania:
- **Obiektowego (OOP)** – reprezentowanego przez klasy `Book`, `User`, `Reader`, `Librarian`, `Library`.
- **Funkcyjnego (FP)** – reprezentowanego przez unikanie stanów ubocznych (tam, gdzie to możliwe), stosowanie funkcji czystych, przekazywanie funkcji jako parametrów, funkcje anonimowe (lambda) oraz przetwarzanie strumieniowe danych przy użyciu `filter`, `map`, `sorted` i `max`.

Zastosowanie elementów funkcyjnych w tym projekcie miało na celu **skrócenie kodu**, **zwiększenie jego czytelności (styl deklaratywny)** oraz **zredukowanie liczby tymczasowych zmiennych i pętli `for`**.

---

## 2. Kluczowe Koncepcje Funkcyjne Użyte w Kodzie

### A. Funkcje jako Obiekty Pierwszej Klasy (First-Class Citizens)
W programowaniu funkcyjnym funkcje są traktowane jak każdy inny typ danych (np. liczby czy napisy). Można je:
1. Zapisać w zmiennej.
2. Przekazać jako argument do innej funkcji.
3. Zwrócić z funkcji.

#### Przykład z kodu ([library.py:L235-L241](file:///c:/Users/Amelka%20i%20Zuzia/Desktop/BigData/big-data-library-functional/library.py#L235-L241)):
Zdefiniowaliśmy funkcję `wyswietl_katalog_funkcyjnie`, która jako parametry przyjmuje inne funkcje (`predicate_func` oraz `sort_key_func`):
```python
def wyswietl_katalog_funkcyjnie(books: list, predicate_func, sort_key_func=None):
    filtered_books = list(filter(predicate_func, books))
    
    if sort_key_func:
        display_list = sorted(filtered_books, key=sort_key_func)
    # ...
```
* **Wyjaśnienie:** Funkcja nie wie z góry, według jakich kryteriów będzie filtrować ani sortować książki. Logikę filtrowania (`predicate_func`) i sortowania (`sort_key_func`) wstrzykujemy z zewnątrz w postaci funkcji.

---

### B. Wyrażenia Lambda (Funkcje Anonimowe)
Wyrażenia `lambda` to skrócona składnia definiowania prostych, nienazwanych (anonimowych) funkcji w jednej linii. Używa się ich głównie jako argumentów do funkcji wyższego rzędu.

#### Przykład 1: Dynamiczny predykat filtrowania ([library.py:L260](file:///c:/Users/Amelka%20i%20Zuzia/Desktop/BigData/big-data-library-functional/library.py#L260))
W menu wyszukiwania tworzymy dynamiczną lambdę na podstawie wpisu użytkownika:
```python
predykat = lambda b: (phrase in b.title.lower() or phrase in b.author.lower()) and (not tylko_dostepne or b.available_copies > 0)
```
* **Co robi?** Przyjmuje obiekt książki `b` i zwraca wartość logiczną (`True`/`False`), określając, czy książka pasuje do wpisanej frazy i czy jest dostępna.

#### Przykład 2: Dynamiczne klucze sortowania ([library.py:L270-L274](file:///c:/Users/Amelka%20i%20Zuzia/Desktop/BigData/big-data-library-functional/library.py#L270-L274))
```python
if wybor_sort == "1":
    sort_key = lambda b: b.title.lower()
elif wybor_sort == "2":
    sort_key = lambda b: b.author.lower()
elif wybor_sort == "3":
    sort_key = lambda b: b.available_copies
```
* **Co robi?** W zależności od wyboru użytkownika tworzy funkcję wyciągającą z obiektu książki odpowiednie pole (tytuł, autor, kopie) przekształcone tak, by sortowanie działało prawidłowo (np. bez rozróżniania wielkości liter).

---

### C. Funkcje Wyższego Rzędu (Higher-Order Functions - HOF)
Funkcja wyższego rzędu to funkcja, która **przyjmuje inną funkcję jako argument** lub **zwraca funkcję**. W kodzie intensywnie korzystamy zarówno z wbudowanych HOF w Pythonie (`filter`, `map`, `sorted`, `max`), jak i z własnej (`wyswietl_katalog_funkcyjnie`).

#### 1. Funkcja `filter(funkcja_warunku, kolekcja)`
Wybiera z kolekcji elementy spełniające zadany warunek (dla których funkcja warunku zwraca `True`). Zwraca leniwy iterator.

* **Przykład z kodu ([library.py:L125](file:///c:/Users/Amelka%20i%20Zuzia/Desktop/BigData/big-data-library-functional/library.py#L125)):**
  ```python
  matched = list(filter(lambda b: b.title.strip().lower() == search_title, self.__books.values()))
  ```
  *Jak to działa?* Filtruje wszystkie książki z biblioteki i pozostawia tylko te, których tytuł po usunięciu spacji i zamianie na małe litery jest identyczny z szukanym.

* **Przykład z kodu ([library.py:L227](file:///c:/Users/Amelka%20i%20Zuzia/Desktop/BigData/big-data-library-functional/library.py#L227)):**
  ```python
  readers = filter(lambda u: isinstance(u, Reader), self.__users.values())
  ```
  *Jak to działa?* Wybiera z bazy użytkowników tylko te obiekty, które są instancją klasy `Reader` (czytelnikami), odrzucając bibliotekarzy.

#### 2. Funkcja `map(funkcja_transformacji, kolekcja)`
Przekształca każdy element kolekcji za pomocą podanej funkcji transformacji. Zwraca leniwy iterator.

* **Przykład z kodu ([library.py:L228](file:///c:/Users/Amelka%20i%20Zuzia/Desktop/BigData/big-data-library-functional/library.py#L228)):**
  ```python
  return sum(map(lambda r: len(r.borrowed_books), readers))
  ```
  *Jak to działa?* Dla każdego czytelnika (`r`) pobiera liczbę jego aktualnych wypożyczeń (`len(r.borrowed_books)`), co daje strumień liczb. Następnie funkcja `sum()` sumuje te liczby, zwracając łączną liczbę wypożyczeń w systemie. Zastępuje to tradycyjną pętlę i zmienną akumulującą.

#### 3. Funkcja `max(kolekcja, key=funkcja_kryterium)`
Wyszukuje element maksymalny w kolekcji na podstawie wartości zwracanej przez funkcję kryterium.

* **Przykład z kodu ([library.py:L224](file:///c:/Users/Amelka%20i%20Zuzia/Desktop/BigData/big-data-library-functional/library.py#L224)):**
  ```python
  return max(all_books, key=lambda b: b.total_copies - b.available_copies)
  ```
  *Jak to działa?* Dla każdej książki liczy różnicę między liczbą wszystkich egzemplarzy a dostępnymi (czyli wyznacza liczbę aktualnie wypożyczonych sztuk) i na tej podstawie wskazuje najpopularniejszą książkę.

#### 4. Funkcja `sorted(kolekcja, key=funkcja_kryterium, reverse=True/False)`
Sortuje kolekcję na podstawie wartości zwracanej przez funkcję kryterium. Zwraca nową listę.

* **Przykład z kodu ([library.py:L232](file:///c:/Users/Amelka%20i%20Zuzia/Desktop/BigData/big-data-library-functional/library.py#L232)):**
  ```python
  return sorted(readers, key=lambda r: len(r.borrowed_books), reverse=True)
  ```
  *Jak to działa?* Sortuje czytelników malejąco (`reverse=True`) według liczby ich aktywnych wypożyczeń, tworząc ranking.

---

### D. Wyrażenia Listowe (List Comprehensions)
Zapewniają zwięzłą składnię do tworzenia list na podstawie istniejących kolekcji. Reprezentują one **deklaratywne podejście** (opisujemy *co* chcemy osiągnąć, a nie krok po kroku *jak* ma to zrobić pętla).

#### Przykład z kodu ([library.py:L209-L210](file:///c:/Users/Amelka%20i%20Zuzia/Desktop/BigData/big-data-library-functional/library.py#L209-L210)):
```python
readers = [u for u in self.__users.values() if isinstance(u, Reader)]
return [(r.login, b) for r in readers for b in r.borrowed_books]
```
* **Pierwsza linia:** Tworzy listę obiektów czytelników (filtrowanie za pomocą list comprehension).
* **Druga linia (Zagnieżdżone wyrażenie listowe):** Tworzy listę krotek `(login, ksiazka)` spłaszczając relację jeden-do-wielu (jeden czytelnik ma wiele książek). Odpowiednik tradycyjnych zagnieżdżonych pętli `for`:
  ```python
  result = []
  for r in readers:
      for b in r.borrowed_books:
          result.append((r.login, b))
  return result
  ```
  Zastosowanie wyrażenia listowego pozwala zrobić to samo czytelniej w jednej linii.

---

## 3. Pytania Rekrutacyjne / Egzaminacyjne (Q&A) – Gotowe Odpowiedzi dla Wykładowcy

### Pytanie 1: Dlaczego w projekcie połączono podejście obiektowe (OOP) i funkcyjne (FP)?
> **Odpowiedź:** Połączyliśmy oba paradygmaty, ponieważ idealnie się dopełniają. Obiektowość służy do **modelowania struktur danych i stanu** (np. książka, użytkownik czy baza danych biblioteki mają swoje właściwości, atrybuty prywatne i zachowania). Z kolei paradygmat funkcyjny doskonale nadaje się do **przetwarzania, filtrowania, transformacji i analizy tych danych** (np. raporty, wyszukiwanie, statystyki). Dzięki temu unikamy pisania skomplikowanych pętli i zmiennych pomocniczych, a kod staje się bardziej deklaratywny i czytelny.

### Pytanie 2: Co to jest funkcja lambda i kiedy warto ją stosować?
> **Odpowiedź:** Funkcja lambda to krótka, anonimowa (nienazwana) funkcja zdefiniowana w jednej linii kodu za pomocą słowa kluczowego `lambda`. Może przyjmować dowolną liczbę argumentów, ale może zawierać tylko jedno wyrażenie, którego wynik jest automatycznie zwracany. Warto ją stosować jako jednorazową, prostą funkcję pomocniczą przekazywaną jako argument do innych funkcji (np. jako klucz sortowania w `sorted()` lub warunek w `filter()`), bez konieczności definiowania jej za pomocą `def` i zaśmiecania przestrzeni nazw.

### Pytanie 3: Jak działa funkcja wyższego rzędu (Higher-Order Function)? Podaj przykłady ze swojego kodu.
> **Odpowiedź:** Funkcja wyższego rzędu to taka funkcja, która przyjmuje przynajmniej jedną inną funkcję jako argument lub zwraca funkcję jako swój wynik.
> W moim projekcie użyłem wbudowanych funkcji wyższego rzędu:
> - `filter()`, która przyjmuje funkcję sprawdzającą warunek (predykat), np. `filter(lambda u: isinstance(u, Reader), ...)`
> - `map()`, która przyjmuje funkcję transformującą dane, np. `map(lambda r: len(r.borrowed_books), ...)`
> - `sorted()`, która przyjmuje funkcję wyznaczającą klucz sortowania w argumencie `key`.
> Ponadto napisałem własną funkcję wyższego rzędu: `wyswietl_katalog_funkcyjnie`, która przyjmuje jako argumenty `predicate_func` (funkcję filtrującą katalog) oraz `sort_key_func` (funkcję określającą według czego sortować).

### Pytanie 4: Czym różni się funkcja `filter()` od wyrażenia listowego (list comprehension) w Pythonie i co jest lepsze?
> **Odpowiedź:** Pod kątem funkcjonalnym dają podobne rezultaty, ale różnią się implementacją. 
> `filter()` w Pythonie 3 zwraca **generator (leniwy iterator)**, co oznacza, że elementy są przetwarzane dopiero w momencie, gdy są potrzebne (oszczędność pamięci). Wyrażenie listowe od razu tworzy i ładuje całą listę do pamięci (ewentualnie można zastosować generator expression w nawiasach okrągłych). 
> Pod kątem czytelności w społeczności Pythona częściej preferuje się wyrażenia listowe, jednak `filter` w połączeniu z `lambda` jest bardzo czytelnym i klasycznym podejściem funkcyjnym. W projekcie używamy obu tych form świadomie, dopasowując je do potrzeb (np. `filter` do strumieniowania i redukcji, a list comprehension do generowania i spłaszczania list wyświetlanych).

### Pytanie 5: Czym różni się styl imperatywny od deklaratywnego w kontekście przetwarzania list?
> **Odpowiedź:** 
> - W stylu **imperatywnym** opisujemy krok po kroku instrukcje sterujące przepływem (np. tworzymy pustą listę, piszemy pętlę `for`, dodajemy warunek `if`, a następnie metodą `.append()` modyfikujemy listę). Mówimy komputerowi dokładnie *jak* ma to zrobić.
> - W stylu **deklaratywnym** opisujemy *co* chcemy osiągnąć (np. przy użyciu `filter` i `map` lub wyrażeń listowych deklarujemy zbiór wynikowy na podstawie kryteriów filtrowania i transformacji). Kod staje się bardziej matematycznym opisem relacji danych, a szczegóły sterowania pętlami są ukryte wewnątrz wbudowanych mechanizmów języka.
