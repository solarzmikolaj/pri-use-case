# Przetwarzanie Danych z Wykorzystaniem Paradygmatu Funkcyjnego (Pandas/NumPy)
## Przewodnik i Notatki na Obronę Projektu (big-data-exercise3)

Ten dokument opisuje, w jaki sposób paradygmat **programowania funkcyjnego (Functional Programming - FP)** został zaimplementowany w analizie danych z wykorzystaniem bibliotek **Pandas** i **NumPy** w projekcie `big-data-exercise3` ([apartment_analysis.py](file:///c:/Users/Amelka%20i%20Zuzia/Desktop/BigData/big-data-exercise3/App/apartment_analysis.py)).

---

## 1. Dlaczego Pandas i NumPy to Paradygmat Funkcyjny?

Tradycyjne języki programowania i starsze style analizy opierają się na programowaniu imperatywnym (pętle `for`, modyfikacje obiektów w miejscu). W analizie danych (Big Data) takie podejście jest nieefektywne i podatne na błędy. Narzędzia takie jak **Pandas** i **NumPy** zostały zaprojektowane w oparciu o idee funkcyjne:
1. **Niezmienność (Immutability):** Operacje nie powinny modyfikować oryginalnych danych, lecz zwracać nowe kopie/wyniki.
2. **Wektoryzacja (Vectorization):** Operacje są deklaratywnymi przekształceniami całych struktur danych bez jawnego użycia pętli `for`.
3. **Potoki przetwarzania (Pipelines / Method Chaining):** Przekazywanie wyniku jednej operacji jako wejścia do kolejnej w formie płynnego łańcucha.

---

## 2. Szczegółowe Przykłady Funkcyjne z Kodu Projektu

### A. Niezmienność danych (Immutability) i brak efektów ubocznych (Side Effects)
W programowaniu funkcyjnym dążymy do tego, aby funkcje były **czyste** (pure functions) – czyli nie zmieniały stanu obiektów przekazanych jako argumenty.

#### Przykład z kodu ([data_processing.py:L122-L133](file:///c:/Users/Amelka%20i%20Zuzia/Desktop/BigData/big-data-exercise3/App/data_processing.py#L122-L133)):
```python
def clean_data(df: pd.DataFrame) -> Tuple[pd.DataFrame, Dict[str, Any]]:
    cleaned_df = df.copy()
    # ... modyfikacje na kopii ...
    return cleaned_df, metadata
```
* **Wyjaśnienie:** Funkcja `clean_data` nie modyfikuje oryginalnej ramki danych `df`. Pierwszą rzeczą, którą robi, jest utworzenie głębokiej kopii za pomocą `df.copy()`. Wszystkie przekształcenia (filtrowanie, winsoryzacja, logarytmowanie) są wykonywane na kopii, która jest następnie zwracana. Dzięki temu oryginalne dane wejściowe pozostają nienaruszone, co zapobiega trudnym do wykrycia błędom ubocznym.

---

### B. Wektoryzacja (Vectorized Operations) zamiast Pętli
Pętle `for` w Pythonie są powolne. Wektoryzacja polega na przeniesieniu pętli do niskopoziomowego kodu C (pod spodem NumPy) poprzez stosowanie operacji na całych tablicach/seriach jednocześnie. Jest to deklaratywne podejście funkcyjne (mówimy *co* zrobić z serią, a nie *jak* iterować po elementach).

#### Przykład 1: Przekształcenie matematyczne serii ([data_processing.py:L150](file:///c:/Users/Amelka%20i%20Zuzia/Desktop/BigData/big-data-exercise3/App/data_processing.py#L150))
```python
cleaned_df["cena_pln_log"] = np.log1p(cleaned_df["cena_pln"])
```
* *Jak to działa?* Funkcja `np.log1p` przyjmuje całą serię cen i zwraca nową serię zawierającą logarytm naturalny z wartości powiększonej o 1. Nie ma tu pętli przechodzącej po każdym wierszu z osobna.

#### Przykład 2: Filtrowanie warunkowe ([data_processing.py:L137](file:///c:/Users/Amelka%20i%20Zuzia/Desktop/BigData/big-data-exercise3/App/data_processing.py#L137))
```python
cleaned_df = cleaned_df[(cleaned_df["rok_budowy"] >= 1900) & (cleaned_df["rok_budowy"] <= 2026)]
```
* *Jak to działa?* Zamiast pętli sprawdzającej każdy wiersz, tworzymy maski logiczne (wektory typu boolean) za pomocą operatorów bitowych `&`. Pandas wybiera tylko te wiersze, dla których maska ma wartość `True`.

---

### C. Potoki Przetwarzania Danych (Method Chaining)
Styl funkcyjny zachęca do łączenia wywołań metod w jeden ciągły potok przetwarzania danych.

#### Przykład z kodu ([apartment_analysis.py:L177](file:///c:/Users/Amelka%20i%20Zuzia/Desktop/BigData/big-data-exercise3/App/apartment_analysis.py#L177)):
```python
median_per_m2 = df_analysis.groupby("dzielnica")["cena_pln_per_m2"].median().sort_values(ascending=False)
```
* **Kroki potoku:**
  1. `df_analysis.groupby("dzielnica")` – Grupuje dane według dzielnic (podział).
  2. `["cena_pln_per_m2"]` – Wybiera kolumnę ceny za metr kwadratowy z każdej grupy.
  3. `.median()` – Aplikuje funkcję redukującą (mediana) dla każdej grupy.
  4. `.sort_values(ascending=False)` – Sortuje uzyskane mediany malejąco.
* **Wyjaśnienie:** Cały proces agregacji i analizy odbywa się w jednej linii, bez zapisywania stanów pośrednich w zmiennych tymczasowych.

---

### D. Funkcje jako Narzędzia Matematyczne (Pure Functions)
Wszystkie funkcje w pliku `data_processing.py` działają jak czyste funkcje matematyczne – pobierają parametry i zwracają wynik, np.:
* `calculate_price_stats(df)` -> Zwraca słownik z miarami statystycznymi rozkładu.
* `get_iqr_stats(df, column)` -> Wylicza Q1, Q3, rozstęp ćwiartkowy (IQR), granice oraz zwraca wyodrębnioną ramkę z wartościami odstającymi.

---

## 3. Pytania Rekrutacyjne / Egzaminacyjne (Q&A) – Gotowe Odpowiedzi dla Wykładowcy

### Pytanie 1: W jaki sposób Twój kod do analizy danych wykorzystuje programowanie funkcyjne?
> **Odpowiedź:** Wykorzystuje go poprzez biblioteki Pandas i NumPy, które są zaprojektowane w paradygmacie funkcyjnym. Zamiast pętli `for` używamy **wektoryzacji** (np. przy filtrowaniu błędnych lat budowy czy transformacji logarytmicznej cen). Ponadto, w kodzie stosujemy zasady **niezmienności danych** – funkcja `clean_data` nie modyfikuje wejściowej ramki danych w miejscu, lecz tworzy kopię `df.copy()`, przekształca ją i zwraca nowy obiekt. Stosujemy też **method chaining** (łączenie metod) do czytelnego tworzenia potoków agregacji danych.

### Pytanie 2: Co to jest wektoryzacja i dlaczego jest lepsza od tradycyjnej pętli `for` w Pythonie?
> **Odpowiedź:** Wektoryzacja to technika polegająca na wykonywaniu operacji na całych tablicach danych jednocześnie, zamiast przetwarzania pojedynczych elementów jeden po drugim. 
> Jest znacznie lepsza z dwóch powodów:
> 1. **Wydajność:** Pętle w czystym Pythonie są wolne z powodu dynamicznego typowania i narzutu interpretera. Wektoryzacja w NumPy/Pandas deleguje te pętle do zoptymalizowanego kodu w języku C.
> 2. **Styl deklaratywny:** Kod staje się prostszy i bardziej przejrzysty, ponieważ opisuje matematyczne operacje na zbiorze (np. `cena / metraz`), a nie mechanikę indeksowania i iterowania.

### Pytanie 3: Po co zastosowano transformację logarytmiczną (`np.log1p`) na cenach mieszkań i jak to działa?
> **Odpowiedź:** Ceny mieszkań mają silny rozkład prawoskośny (skewness ok. 2.0+), ponieważ istnieje niewiele ekstremalnie drogich nieruchomości, które tworzą długi ogon po prawej stronie. Taka skosnosc negatywnie wpływa na wiele algorytmów statystycznych i uczenia maszynowego. 
> Transformacja `np.log1p(x)` (czyli $\ln(x+1)$) ściąga duże wartości do mniejszego zakresu, spłaszczając ten długi ogon i przybliżając rozkład cen do rozkładu normalnego (redukując skośność do poziomu bliskiego 0). Użycie log1p zamiast zwykłego log zapobiega błędom matematycznym, gdyby w danych pojawiła się wartość zero.

### Pytanie 4: Czym różni się standardowy Z-score od Modified Z-score w wykrywaniu wartości odstających (outlierów)?
> **Odpowiedź:** 
> - Standardowy **Z-score** bazuje na średniej arytmetycznej i odchyleniu standardowym ($Z = \frac{x - \mu}{\sigma}$). Zarówno średnia, jak i odchylenie standardowe są skrajnie wrażliwe na wartości odstające (outliery je "przyciągają", przez co same granice detekcji się przesuwają i wykrywamy mniej outlierów).
> - **Modified Z-score** jest odporną (robust) metodą alternatywną. Używa mediany oraz **odchylenia bezwzględnego od mediany (MAD)** zamiast średniej i odchylenia standardowego. Ponieważ mediana i MAD są niewrażliwe na skrajne wartości, Modified Z-score znacznie lepiej i stabilniej identyfikuje outliery w silnie zaburzonych zbiorach danych.
