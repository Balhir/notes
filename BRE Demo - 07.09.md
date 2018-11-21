# BRE Demo - 07.09

## Topics to cover

1. DAS Auto
    1. Planner
        1. PBS
    2. Starter
    3. Worker
        1. PBS
        2. BRE
        3. CORE
2. EventStore migracja
3. Scoring DIY
4. TimeTravel

## DAS Auto

Flow danych zaczynajac od PBS. że majać info o credit checkach i purchasach mozemy wybierac ludzi do anonymizacji.
Mamy takie plannery
mamy takie stepy w tych plannerach
Dogadujemy sie z systemami czy mozna anonymizowac

struktura solution
solution napisana w .net core
akceptacyjki w net45 bo specflow

SingleThreadService

nssm

## Event Store old data migration

trwałonawet 8h

chodziło o skrypty podczas deployou


Obrazek ze scrubs

On the last demo we were talking about removing the EventStore solution from Purchase Behaviour System.
Since that time we were able to successful replace it with PostgreSQL database.

update przy duzej ilości danych sa chujowe. Bo nie zmieniają danych w kolumnie tylko wywalaja stary rekord i dodaja go jeszcze raz z nowymi wartosciami

This new moving piece which is PosgtreSQL db has his pros and cons.
    
Pierwsza iteracja rozwizania działała ok na Test ale na QA kazde odpytanie API kończyło się timeout'em.
Głównym problemem na QA była ilość danych oraz to, że inner join, który był wymagany w zapytaniu SQL był zbudowany na dwóch stringach.
W związku z tym baza nie dawała sobie rady pomimo usilnych prób zbudowania indeksów, które by pomogły zoptymalizować plan zapytania.

Kolejnym pomysłem był widok zmaterializowany, ale niestety jego odświeżenie trało tyle samo co wywołanie bezpośredniego zapytania i nie mieliśmy pewności czy od momentu odświeżenia danych do ich pobrania przez API dane nie zdeaktualizowały się.

Wtedy podeszliśmy do problemu inaczej - zbudowaliśmy triggery, które na każdorazowej akcji na tabelach z purchase czy (pre)credit check aktualizowały wpis
w tabeli ze statystykami per customer. Dzięki temu zbudowaliśmy w zasadzie samoodświeżający się widok zmaterializowany, które zawsze miał aktualne dane. Po założeniu kilku brakujących indeksów doszliśmy do momentu gdzie wydajność tego rozwiązania nas zadowalałą.

Lesson learnd: if you do not have auto refresed materialized view you can build your own.

Inną sprawą jest także to, że wyciągneliśmy z bazy CORE dokładniej to z jej repliki ODS stare dane dot. purchasów wszystkich klientów od początku świata i wsadziliśmy to do bazy postgres w PBS. Dane te są nam potrzebne do wyliczania takich metryk jak ilość purchasów w 24h czy 30 dniach, które później są wykorzystywane w drzewach credit decision.
Od jakiegoś czasu dostajemy te dane w formie eventów, które kiedyś lądowały w ES a obecnie są przechowywane w bazie PBS @ postgres.

Rozwiązanie obu tych problemów pozwoliło nam ostatecznie usunąć EventStore z naszej solucji, odinstalować jego instancje na serwerach i skasować już nie potrzebne bazy danych.

Mogło to mieć wpływ na wyniki ostatnich testów wydanościowych gdzie zauważona została poprawa waydajności.

## Scoring DIY

Obecnie w Finladii korzystamy z dwóch rodzajów scoringu: in-house oraz full. Scoring in-house jest to stary typ, który wykorzystuje naszą wewnętrzną bazę scoringową oraz zestaw reguł, które to wyliczają ten score dla każdego klienta.
Full scoring jest to wywołanie zapytania do zewnętrznego serwisu ADAPA, który to na podwstawie modelu napisane w PMML i dużej ilości parametrów wejściowych wylicza score dla klienta.

Niestety ADAPA jest:

1. droga
2. licencja nie jest nasza (Lowell) tylko Intrum

dlatego powstał pomysł zaimplementowania modelu PMML bezpośrednio w C#.

Model PMML przyjmuje na wejściu klilkanaście parametrów. Każdy z nich jest numerycznym odpowiednikiem klasy do jakiej wynik danej metryki został przypisany. To znaczy, że jeśli miałeś 1 reminder w ostatnich 30 dniach to mieścisz się w klasie 1 parametru X, jeśli miałeś 2 to klasa 2, jeśli więcej to klasa 3, itd dla wszyskich zmiennych.
Klasy te mapują się na wartości współczynników, które następnie podstawione do wzoru:

<<wzór>>

dają nam wyliczony score.

Ta prosta zasada dała się bardzo ładnie przenieść do C#. Klasyfikację parametrów na klasy mieliśmy już napisaną, gdyż do Adapy wysyłaliśmy jedynie klasy bez wartości. Pozostało jedynie wyciągnąć z modelu PMML współczynniki per klase per parametr i mieliśmy już gotowe wartości do podstawienia do wzoru.

Jest to dość prymitywna forma przeniesienie modelu PMML do C#, ale w naszym przypadku sprawdza się. Jej wstępne testy dowiodły, że produkuje poprawne wyniki w porównaniu do ADAPA (test wykonany na dancyh produkcyjnych).

Na horyzoncie mamy także zaimplementowanie w ten sposób scoringu dla Norwegii, ale jest to jeszcze w fazie planów.