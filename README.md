# Chemical Ontology Classification

Hackathon solution for hierarchical multi-label classification of molecules into 500 chemical ontology classes (ChEBI-style DAG).

## Problem

Given a molecule (as SMILES), predict which of 500 ontology classes it belongs to — a **multi-label hierarchical classification** problem on a DAG class graph.

- **Train:** 33,668 molecules, 500 classes
- **Test:** 11,223 molecules
- **Output:** binary predictions (0/1) for all 500 classes per molecule

## Scoring

- **Primary metric:** Macro-averaged F1 (mean F1 across all 500 classes)
- **Tiebreaker** (if F1 difference < 1%): graph consistency — penalizes predicting a child class without its parent
- **Public leaderboard:** 50% of test molecules
- **Final leaderboard:** 100% of test molecules

<img width="1161" height="635" alt="image" src="https://github.com/user-attachments/assets/8deae600-33fd-447e-9163-3ae92f0301e2" />


# Rozwiązanie

Finalne rozwiązanie tego zadania znajduje się w notatniku `v3.ipynb`.
Poniżej przedstawiono opis zastosowanego podejścia oraz wykorzystanych technik.



## Wyodrębnione cechy (Features Extracted)

Model wykorzystuje dwa komplementarne typy reprezentacji cząsteczek:
- sekwencyjną (SMILES),
- strukturalno-fizykochemiczną.

### 1. Tokeny SMILES (reprezentacja sekwencyjna)
- SMILES traktowany jako ciąg tekstowy.
- Tokenizacja za pomocą BPE (Byte Pair Encoding), słownik = 500.
- BPE umożliwia grupowanie często występujących fragmentów jako pojedynczych tokenów.
- Model uczy się rozpoznawać podstruktury (np. pierścienie, grupy funkcyjne), a nie pojedyncze znaki.
- Ograniczenie słownika zapobiega nadmiernej fragmentacji i wspiera modelowanie zależności przez Transformer.

### 2. Odciski palców ECFP (lokalna topologia)
- 2048-wymiarowe wektory opisujące lokalne otoczenie atomów.
- Reprezentują strukturę cząsteczki poprzez analizę sąsiedztwa atomów w określonym promieniu.
- Redukcja wymiarowości (PCA) do 256:
  - zmniejsza sparsowość,
  - ogranicza przeuczenie,
  - zachowuje najważniejsze informacje strukturalne.

### 3. Klucze MACCS (predefiniowane wzorce)
- 166-bitowe deskryptory obecności konkretnych podstruktur.
- Działają jako zestaw z góry zdefiniowanych reguł chemicznych.
- Dostarczają jednoznacznych informacji o obecności kluczowych grup funkcyjnych.
- Wspierają klasyfikację związków na podstawie znanych cech chemicznych.

### 4. Deskryptory fizykochemiczne i topologiczne
Zestaw globalnych cech opisujących cząsteczkę:
- masa cząsteczkowa i LogP (hydrofobowość),
- liczba donorów i akceptorów wiązań wodorowych,
- TPSA (polarna powierzchnia),
- frakcja sp3 (złożoność przestrzenna),
- cechy pierścieni (liczba, aromatyczność, heteroatomy, struktury mostkowe).

Rola:
- dostarczają informacji o właściwościach makroskopowych,
- istotne dla przewidywania zachowania biologicznego (np. przenikalność, biodostępność).

---

## Architektura modelu: OmniFusionTransformer

### 1. Gałąź SMILES (Transformer)
- Wejście: tokeny SMILES po BPE.
- Embedding + kodowanie pozycyjne.
- Encoder Transformer:
  - 3 warstwy,
  - 4 głowy uwagi,
  - wymiar ukryty = 128.
- Global Average Pooling z maskowaniem paddingu.
- Wynik: wektor reprezentujący całą sekwencję.

### 2. Gałąź cech gęstych
- Wejście: ECFP (po PCA) + MACCS + deskryptory RDKit.
- Normalizacja cech.
- Mechanizm uwagi dla cech (feature attention).
- Warstwy gęste uczące nieliniowych zależności między cechami.

### 3. Mechanizm fuzji (Gated Fusion)
- Połączenie (konkatenacja) obu reprezentacji.
- Trenowalna bramka:
  - kontroluje udział informacji z każdej gałęzi,
  - filtruje i waży sygnały.

### 4. Głowa klasyfikacyjna
- Warstwa liniowa generująca logity dla 500 klas.
- Problem niezbalansowanych danych rozwiązany przez:
  - BinaryFocalLossWithLogits,
  - większe karanie błędów dla rzadkich klas.