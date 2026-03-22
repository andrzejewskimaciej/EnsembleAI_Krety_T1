# Podsumowanie Hackatonu

> **Data hackatonu:**  14-15.03.2026 
> **Zespół:** *Krety* - Ludwik Madej, Maciej Andrzejewski, Karol Kacprzak, Piotr Kotłowski

---

## Co udało nam się zrobić?

Ostateczny wynik - miejsce 24 (na ~45 zespołów). Udało się przesłać rozwiązania do wszystkich zadań. Niestety, niektóre z nich nie były dla nas satysfakcjonujące - problemem był przede wszystkim ograniczony dostęp do mocy obliczeniowej, przez co zabrakło tejże mocy (czyli równoważnie czasu), aby przeprowadzić wszystkie zaplanowane obliczenia. Z drugiej strony daje to zatem pole do dalszych prac nad zadaniami, większość z rozwiązań niewątpliwie można znacząco ulepszyć.



(Dokładne opisy zadań znajdują się w poszczególnych repozytoriach w plikach README.md)

---

### Task 1 - Chemical Ontology Classification

🔗 **Repo:** [link do repozytorium](https://github.com/andrzejewskimaciej/EnsembleAI_Krety_T1)

**Cel zadania:** wielokrotna binarna klasyfikacja hierarchiczna (500 klas) związków chemicznych na postawie reprezentujących ich stringów (1 string per związek).

Udało się stworzyć model (sieć neuronowa) z macro-averaged F1 $\approx 0.85$.

Prawdopodobnie niskie pole do dalszych ulepszeń modelu.

---

### Task 2 - Craft the Context - Code Completion Context Strategy

🔗 **Repo:** [link do repozytorium](https://github.com/andrzejewskimaciej/EnsembleAI_Krety_T2)

**Cel zadania:** Stworzenie precyzyjnego rurociągu zbierania kontekstu (Context Collection Pipeline) dla środowisk Python, który wyszukuje najbardziej trafne fragmenty kodu w repozytorium do zasilenia modeli auto-complete.

**Co zrobione:**
* **Strategia Regex-Definition:** Szybkie wyszukiwanie oparte na wyrażeniach regularnych, dopasowujące słowa wokół kursora do definicji w innych plikach.
* **Analiza luki (FIM):** Wyodrębnienie kluczowego słownictwa z 40 linii wokół edytowanego miejsca (tzw. metoda Fill-in-the-Middle).
* **Priorytetyzacja (Tier System):** Pliki zawierające definicje wywoływanych funkcji dostają najwyższy priorytet. Pliki z ogólnym podobieństwem słownictwa są dodawane w drugiej kolejności.
* **Obrona przed przycinaniem (Left-Trim Defense):** Systemy oceniające ucinają długi kontekst od lewej strony (Mellum ma limit 8K tokenów). Skrypt celowo odwraca kolejność doklejanych plików, aby najważniejsze z nich lądowały na końcu i uniknęły ucięcia.
* **Formatowanie:** Fragmenty kodu połączono wymaganym tokenem `<|file_sep|>`.

**Co można ulepszyć:**
* **Zastosowanie AST:** Zastąpienie regexów parsowaniem drzewa składniowego (AST) dla bezbłędnego śledzenia importów. 
* **Algorytm BM25:** Wdrożenie zaawansowanego wyszukiwania leksykalnego (TF-IDF) do lepszego punktowania powiązań między plikami.
* **Struktura katalogów:** Dodanie punktów premiowych dla plików znajdujących się w tym samym folderze co edytowany plik.
* **"Self-Healing":** Ratowanie obciętych na samej górze importów i wstrzykiwanie ich z powrotem tuż nad lukę.

**Metryka:**
* Wynik **0.35** na ukrytych danych testowych.
* Użyto metryki **ChrF Score** , która mierzy precyzję i pełność na poziomie pojedynczych znaków, co świetnie wyłapuje różnice w składni.
* Ostateczny wynik zależy od średniej z trzech testowanych modeli (Mellum, Codestral, Qwen2.5-Coder).

### Task 3 - Heat Pump Grid Load Forecasting

🔗 **Repo:** [link do repozytorium](https://github.com/andrzejewskimaciej/EnsembleAI_Krety_T3)

**Cel zadania:** predykcja średniego miesięcznego zużycia prądu przez pompy ciepła na podstawie historycznych danych ich działania (szeregi czasowe, dane z czujników zbierane z częstotliwością 5 minut).  
Z powodu dużego rozmiaru ramki danych bardzo trudno było na niej operować na zwykłym laptopie. Zastosowano zatem znaczącą agregację danych - do danych dziennych (uwzględniającą pewne cechy feature engineeringu typowego dla szeregów czasowych), wytrenowano klasyczną sieć neuronową do przewidywania dziennego zużycia prądu z metryką MAE i brano średnią dla całego miesiąca.

Odziwo, tak niedokładne podejście i tak zaowocowało ostatecznie 8 miejscem (na 45).

Jest jednak bardzo dużo możliwości poprawy rozwiązania, począwszy od rzetelnego, statystycznego feature engineeringu uwzględniającego możliwości szeregów czasowych, a skończywszy na bardziej zaawansowanych modelach (LSTM, Transformer).

Metryka oceny rozwiązania: zaproponowana przez organizatorów - MAE dla średnich per (miesiąc, urządzenie)

---

### Task 4 - ECG Digitization Pipeline

[](https://github.com/andrzejewskimaciej/EnsembleAI_Krety_T4#ecg-digitization-pipeline)

🔗 **Repo:** [link do repozytorium](https://github.com/andrzejewskimaciej/EnsembleAI_Krety_T4)

**Cel zadania:** zrzutować ze zdjęć odczytów EKG (zaszumione obrazy - pogięte kartki, plamy, dodatkowe zapiski) wykresy kardioid do postaci wektorów 1D.

Przygotowano zaawansowany, algorytmiczny (bez użycia inteligencji obliczeniowej) preprocessing danych, który w znaczny sposób oczysza zdjęcia, aby były gotowe do łatwego zczytania.

Niestety, uruchomienie pretrenowanych modeli do tego celu przekroczyło zdolności laptopów. Jako rozwiązanie przesłano pewną (bardzo niedokładną) heurystykę znalezioną na podstawie danych treningowych.

Tutaj również pole do dalszego rozwoju jest spore - przede wszystkim ujednolicenie i dopracowania preprocessingu danych oraz uruchomienie pretrenowanych modeli (często badanych w artykułach naukowych), aby zbadać ich aplikowalność dla tych danych. W razie ich nieudolności, kolejnym krokiem byłoby stworzenie swoich modeli - np. wytrenowanie konwolucyjnej sieci neuronowej (jest 3000 zdjęć + potencjalna augmentacja, a więc powinno być to wykonalne). 

Metryka oceny rozwiązania: zaproponowana przez organizatorów: korelacja Pearsona (kształt sygnału) + signal-to-noise ratio (amplitutda sygnału) + korelacja krzyżowa (kalibracja czasowa, badanie przesunięcia)

---

## Harmonogram (wstępny) dalszych prac

| Data         | Temat / Cel spotkania                                   |
| ------------ | ------------------------------------------------------- |
| Pon, [23.03] | Ustalenie szcegółów dotyczących dalszej realizacji prac |
| Pon, [13.04] | Prezentacja kompleksowego rozwiązania do taska 3.       |
| Pon, [4.05]  | Prezentacja kompleksowego rozwiązania do taska 4.       |

---

## Projekt poboczny

**Podgrupa:** Ludwik Madej, Maciej Andrzejewski, Karol Kacprzak

**Opis:** Celem projektu jest dostarczenie kompleksowego rozwiązania umożliwiającego wyznaczenie kohezji oraz kąta tarcia wewnętrznego na podstawie danych pomiarowych pochodzących z eksperymentu ściskania trójosiowego z ciśnieniem okalającym materiału tarciowego.

**Aktualny status:** Finalny etap wyznaczenia szukanych parametrów. Następne będzie przygotowanie do publikacji pierwszego (z dwóch planowanych) artykułów naukowych opisujących przeprowadzony proces badawczy.
