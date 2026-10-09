Konfiguracja i budowanie projektów .NET przez CLI

Przewodnik przedstawia kompletną ścieżkę tworzenia od zera wieloprojektowej solucji w języku C# (aplikacja konsolowa, biblioteka klas oraz testy jednostkowe) z poziomu wiersza poleceń, wykorzystując narzędzie dotnet oraz silnik MSBuild.
1. Inicjalizacja głównej solucji

Solucja (plik .sln) działa jako nadrzędny kontener organizujący projekty i zarządzający kolejnością ich budowania (odpowiednik Makefile spinającego cały projekt).
Bash

# Tworzy pusty plik solucji w obecnym katalogu
dotnet new sln -n NazwaTwojegoProjektu

2. Tworzenie projektów (Modułów)

Polecenie dotnet new generuje szkielety projektów w podfolderach. Najlepiej wywoływać je z poziomu głównego katalogu (tam, gdzie leży plik .sln).
Bash

# Tworzy bibliotekę klas (logika aplikacji, nieposiadająca funkcji Main)
dotnet new classlib -n NazwaTwojegoProjektu.Lib

# Tworzy punkt wejścia - aplikację konsolową (UI)
dotnet new console -n NazwaTwojegoProjektu.App

# Tworzy projekt do testów jednostkowych (w tym przypadku z użyciem frameworka MSTest)
dotnet new mstest -n NazwaTwojegoProjektu.Tests

3. Rejestracja projektów w solucji

Same wygenerowane foldery z kodem nie są automatycznie zarządzane przez system budowania. Należy je jawnie podpiąć do utworzonej wcześniej solucji.
Bash

dotnet sln add NazwaTwojegoProjektu.Lib
dotnet sln add NazwaTwojegoProjektu.App
dotnet sln add NazwaTwojegoProjektu.Tests

4. Konfiguracja zależności (Referencje)

Aby zapobiec problemom z zależnościami cyklicznymi, należy ustalić ścisłą, jednokierunkową hierarchię. Aplikacja konsolowa oraz projekt testowy muszą odwoływać się do biblioteki, aby mieć dostęp do jej klas. Biblioteka musi pozostać całkowicie niezależna.
Bash

# Aplikacja konsolowa otrzymuje dostęp do klas biblioteki
dotnet add NazwaTwojegoProjektu.App reference NazwaTwojegoProjektu.Lib

# Projekt testowy otrzymuje dostęp do klas biblioteki w celu ich testowania
dotnet add NazwaTwojegoProjektu.Tests reference NazwaTwojegoProjektu.Lib

(Uwaga: w przypadku omyłkowego zapętlenia zależności, błędną referencję można usunąć zamieniając słowo add na remove).
5. Kompilacja i uruchamianie aplikacji

MSBuild automatycznie odczytuje drzewo zależności ustalonych w kroku 4. i pilnuje, aby biblioteka skompilowała się jako pierwsza, zanim aplikacja będzie jej potrzebować.
Bash

# Buduje całą solucję w domyślnym trybie deweloperskim (Debug - bez optymalizacji)
dotnet build

# Buduje aplikację w trybie produkcyjnym (Release - z agresywną optymalizacją kodu)
dotnet build -c Release

# Buduje i natychmiast uruchamia wyznaczony projekt z funkcją Main
dotnet run --project NazwaTwojegoProjektu.App

6. Uruchamianie testów i czyszczenie środowiska

Zarządzanie środowiskiem testowym oraz usuwanie problematycznych plików tymczasowych.
Bash

# Wykrywa wszystkie projekty testowe w solucji i automatycznie je wykonuje
dotnet test

# Usuwa wygenerowane pliki binarne (foldery bin/ oraz obj/).
# Niezbędne przy zablokowaniu plików przez system lub przy problemach z cache kompilatora.
dotnet clean
