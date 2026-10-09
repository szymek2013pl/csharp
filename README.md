# Konfiguracja i budowanie projektów .NET przez CLI

Przewodnik przedstawia kompletną ścieżkę tworzenia od zera wieloprojektowej solucji w języku **C#** z poziomu wiersza poleceń, wykorzystując narzędzie `dotnet` oraz system budowania **MSBuild**.

Przykładowa solucja będzie składać się z:

* **biblioteki klas** — logika aplikacji,
* **aplikacji konsolowej** — punkt wejścia programu,
* **projektu testów jednostkowych** — testy biblioteki z wykorzystaniem MSTest.

## Spis treści

1. [Wymagania](#1-wymagania)
2. [Inicjalizacja głównej solucji](#2-inicjalizacja-głównej-solucji)
3. [Tworzenie projektów](#3-tworzenie-projektów)
4. [Dodawanie projektów do solucji](#4-dodawanie-projektów-do-solucji)
5. [Konfiguracja zależności](#5-konfiguracja-zależności)
6. [Budowanie projektu](#6-budowanie-projektu)
7. [Uruchamianie aplikacji](#7-uruchamianie-aplikacji)
8. [Uruchamianie testów](#8-uruchamianie-testów)
9. [Czyszczenie projektu](#9-czyszczenie-projektu)
10. [Przykładowa struktura projektu](#10-przykładowa-struktura-projektu)

---

## 1. Wymagania

Przed rozpoczęciem upewnij się, że masz zainstalowane **.NET SDK**.

Weryfikacja instalacji:

```bash
dotnet --version
```

Możesz również sprawdzić szczegółowe informacje o środowisku:

```bash
dotnet --info
```

---

## 2. Inicjalizacja głównej solucji

Solucja (`.sln`) jest kontenerem organizującym wiele projektów .NET.

Utwórz pustą solucję w bieżącym katalogu:

```bash
dotnet new sln -n NazwaTwojegoProjektu
```

Po wykonaniu polecenia otrzymasz plik:

```text
NazwaTwojegoProjektu.sln
```

---

## 3. Tworzenie projektów

Projekty najlepiej tworzyć w podfolderach głównego katalogu solucji.

### Biblioteka klas

Biblioteka będzie zawierała właściwą logikę aplikacji.

```bash
dotnet new classlib -n NazwaTwojegoProjektu.Lib
```

### Aplikacja konsolowa

Aplikacja konsolowa będzie punktem wejścia programu i będzie zawierała metodę `Main`.

```bash
dotnet new console -n NazwaTwojegoProjektu.App
```

### Projekt testów jednostkowych

Do testów wykorzystamy framework **MSTest**:

```bash
dotnet new mstest -n NazwaTwojegoProjektu.Tests
```

Po wykonaniu wszystkich poleceń struktura katalogów powinna wyglądać podobnie do:

```text
NazwaTwojegoProjektu/
├── NazwaTwojegoProjektu.sln
├── NazwaTwojegoProjektu.Lib/
│   └── NazwaTwojegoProjektu.Lib.csproj
├── NazwaTwojegoProjektu.App/
│   └── NazwaTwojegoProjektu.App.csproj
└── NazwaTwojegoProjektu.Tests/
    └── NazwaTwojegoProjektu.Tests.csproj
```

---

## 4. Dodawanie projektów do solucji

Utworzenie projektów nie powoduje automatycznego dodania ich do solucji.

Dodaj je za pomocą `dotnet sln`:

```bash
dotnet sln add NazwaTwojegoProjektu.Lib
dotnet sln add NazwaTwojegoProjektu.App
dotnet sln add NazwaTwojegoProjektu.Tests
```

Możesz sprawdzić, jakie projekty znajdują się w solucji:

```bash
dotnet sln list
```

---

## 5. Konfiguracja zależności

Wieloprojektowa solucja powinna mieć jasno określoną hierarchię zależności.

W naszym przypadku będzie ona wyglądała następująco:

```text
                  ┌──────────────────┐
                  │       Lib        │
                  └────────┬─────────┘
                           │
                ┌──────────┴──────────┐
                │                     │
                ▼                     ▼
        ┌───────────────┐     ┌────────────────┐
        │      App      │     │     Tests      │
        └───────────────┘     └────────────────┘
```

Biblioteka pozostaje niezależna od pozostałych projektów.

### Referencja z aplikacji do biblioteki

```bash
dotnet add NazwaTwojegoProjektu.App reference NazwaTwojegoProjektu.Lib
```

### Referencja z testów do biblioteki

```bash
dotnet add NazwaTwojegoProjektu.Tests reference NazwaTwojegoProjektu.Lib
```

Dzięki temu:

* `App` może korzystać z klas znajdujących się w `Lib`,
* `Tests` może testować kod znajdujący się w `Lib`,
* `Lib` nie musi znać ani aplikacji, ani projektu testowego.

### Usuwanie referencji

Jeżeli przypadkowo dodasz nieprawidłową zależność, możesz ją usunąć:

```bash
dotnet remove NazwaTwojegoProjektu.App reference NazwaTwojegoProjektu.Lib
```

---

## 6. Budowanie projektu

### Build w trybie Debug

Domyślnie projekt jest budowany w konfiguracji `Debug`:

```bash
dotnet build
```

Polecenie zbuduje projekty znajdujące się w bieżącej solucji.

MSBuild analizuje zależności pomiędzy projektami i odpowiednio ustala kolejność ich budowania.

### Build w trybie Release

Do przygotowania wersji przeznaczonej do publikacji można wykorzystać konfigurację `Release`:

```bash
dotnet build -c Release
```

Konfiguracja `Release` jest przeznaczona do produkcyjnego builda i zazwyczaj wykorzystuje optymalizacje kompilatora.

---

## 7. Uruchamianie aplikacji

Aby uruchomić aplikację konsolową:

```bash
dotnet run --project NazwaTwojegoProjektu.App
```

Możesz również uruchomić ją w konfiguracji `Release`:

```bash
dotnet run --project NazwaTwojegoProjektu.App -c Release
```

---

## 8. Uruchamianie testów

Wszystkie testy znajdujące się w solucji można uruchomić za pomocą:

```bash
dotnet test
```

Polecenie:

1. znajdzie projekty testowe,
2. zbuduje wymagane projekty,
3. uruchomi testy,
4. wyświetli wynik w terminalu.

Możesz również uruchomić testy tylko dla konkretnego projektu:

```bash
dotnet test NazwaTwojegoProjektu.Tests
```

---

## 9. Czyszczenie projektu

Jeżeli wystąpią problemy z artefaktami wygenerowanymi podczas kompilacji, można wyczyścić projekt:

```bash
dotnet clean
```

Polecenie usuwa wygenerowane artefakty kompilacji, m.in. zawartość katalogów:

```text
bin/
obj/
```

Po wyczyszczeniu projektu można ponownie wykonać:

```bash
dotnet build
```

---

## 10. Przykładowa struktura projektu

Po wykonaniu wszystkich opisanych kroków projekt może wyglądać następująco:

```text
NazwaTwojegoProjektu/
│
├── NazwaTwojegoProjektu.sln
│
├── NazwaTwojegoProjektu.Lib/
│   ├── Class1.cs
│   └── NazwaTwojegoProjektu.Lib.csproj
│
├── NazwaTwojegoProjektu.App/
│   ├── Program.cs
│   └── NazwaTwojegoProjektu.App.csproj
│
└── NazwaTwojegoProjektu.Tests/
    ├── UnitTest1.cs
    └── NazwaTwojegoProjektu.Tests.csproj
```

### Zależności

```text
NazwaTwojegoProjektu.App
            │
            ▼
NazwaTwojegoProjektu.Lib
            ▲
            │
NazwaTwojegoProjektu.Tests
```

---

## Pełna sekwencja poleceń

Jeżeli chcesz utworzyć całą solucję od zera, możesz wykonać kolejno:

```bash
# Utworzenie solucji
dotnet new sln -n NazwaTwojegoProjektu

# Utworzenie projektów
dotnet new classlib -n NazwaTwojegoProjektu.Lib
dotnet new console -n NazwaTwojegoProjektu.App
dotnet new mstest -n NazwaTwojegoProjektu.Tests

# Dodanie projektów do solucji
dotnet sln add NazwaTwojegoProjektu.Lib
dotnet sln add NazwaTwojegoProjektu.App
dotnet sln add NazwaTwojegoProjektu.Tests

# Dodanie referencji
dotnet add NazwaTwojegoProjektu.App reference NazwaTwojegoProjektu.Lib
dotnet add NazwaTwojegoProjektu.Tests reference NazwaTwojegoProjektu.Lib

# Budowanie
dotnet build

# Uruchomienie aplikacji
dotnet run --project NazwaTwojegoProjektu.App

# Uruchomienie testów
dotnet test

# Czyszczenie
dotnet clean
```

## Podsumowanie

Podstawowy workflow pracy z wieloprojektową solucją .NET może wyglądać następująco:

```text
Utwórz solucję
      │
      ▼
Utwórz projekty
      │
      ▼
Dodaj projekty do solucji
      │
      ▼
Skonfiguruj referencje
      │
      ▼
dotnet build
      │
      ├──────────────► dotnet run
      │
      └──────────────► dotnet test
```

Dzięki `dotnet CLI` cały proces tworzenia, budowania, testowania i uruchamiania wieloprojektowej aplikacji .NET można wykonać bezpośrednio z poziomu terminala.
