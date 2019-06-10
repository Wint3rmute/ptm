# Kolos 1

### Podstawowe tryby pracy portu równoległego
Dla układu Intel 8255 z dwoma 8-bitowymi portami PA i PB

* **Tryb 0** – przeznaczony do realizacji bezwarunkowych operacji I/O.
    * Możliwość zaprogranowania każdego portu jako I lub O
    * Wyjścia z rejestrami zatrzaskowymi i bez tych rejestrów (co?)
* **Tryb 1** - przeznaczony do realizacji operacji IO z przerwaniem, przy jednym kierunku przesyłania danych. Potrzebne do tego celu sygnały sa wyprowadzane lub wprowadzane z wykorzystaniem konkretnej linii portu
* **Tryb 2** - przeznaczony do operacji IO z przerwaniem, tylko przez port A, przy dwóch kierunkach przesyłu danych


### Omów mechanizmy odświeżania DRAM
Odświeżanie DRAM jest konieczne przez wzgląd na sposób przechowywania danych - trzymane są w kondensatorach, które z czasem tracą ładunek. Odświeżanie jest dodatkową pracą dla procesora.

*Słowniczek:*
*RAS - row address strobe - adres wiersza komórki*
*CAS - column address strobe - adres kolumny komórki*

Mamy 3 tryby odświeżania:
* **Tylko sygnałem RAS** - tylko adres wiersza zostaje zatwierdzony po sygnale RAS. Nie ma potrzeby używania CAS ale jest potrzebny zewnętrzny licznik do iteracji po wierszu
* **Ukryte** - użycie obu linii CAS i RAS, najpierw następuje synchronizacja wiersza, potem kolumny
* **CAS przed RAS** - gdy brak sygnału na CAS przed RAS, DRAM ignoruje adresy i używa wewnętrznego licznika, aby odświeżyć

### Omów organizacje pamięci cache i mechanizmy zastępowania danych w pamięci cache
*Słowniczek:*
*Cache - kurewsko szybka i kurewsko droga pamięććć*
*Hit or Memes ratio - Skuteczność Cache - im częściej szukana w RAMie komórka znajduje się już w cache'u, tym lepiej* TODO

###### Organizacje cache'u:
* **Direct Mapping Cache** - dla ustawionego rozmiaru 'skoku' (dalej S), czyli odległości relatywnej w pamięci głównej, dana linia i w pamięci cache, może odwoływać się do adresu i, s+i, 2s+i w RAMie.
Czyli trzymamy w cache'u zawartość komórki oraz licznik i, który mówi, jak duży jest skok
* **Fully Associative Cache** - najłatwiej - każda linia pamięci cache może odwoływać się do dowolnej komórki pamięci, trochę jak zwykły wskaźnik.
* **Set Associative Cache** - Pamięć dzielimy na zbiory - sety. Mieszanka DMC oraz FAC - numer setu determinuje nam zakres głównej pamięci, natomiast każda komórka cache w secie, może wskazywać na dowolne miejsce w zakresie tego setu

###### Mechanizmy zastępowania danych
* Z uwzględnieniem informacji:
    * **MIN** - co najdłużej nie będzie potrzebne. Korzystając z low-levelowej magii czarodziejów assemblera, czytamy kod do przodu i ustalamy, jakie dane nie będą potrzebne w najbliższym czasie. Niestety różne rozgałęzienia (ify) przebiegu programu mogą wyprowadzić w pole takie mechanizmy (ciekawostka - makra `likely` oraz `unlikely` w C)
    * **LRU** - co najdłużej potrzebne nie było (skoro długo nie było potrzebne, dalej pewnie też nie będzie). Może być słabe w sytuacji, gdy mamy w cache 10 komórek, a pracujemy na 11stu, więc ciągle będziemy wprowadzać i usuwać komórki z cache'u
* Bez uwzględniania informacji:
    * **FIFO** - pierwsze wlata pierwsze wylata *głosem Pana Mazurkiewicza*. Kiepskie, kiedy dużo operujemy na jednej zmiennej, a wymieniamy resztę (np jakaś iteracja). Wtedy będziemy 'wymiatali' często używaną zmienną tak często, jak te używane tylko raz.
    * **Random** - tanie, nie wymaga żadnego układu zarządzania cache, morda nosacza

TODO JEST WIĘCEJ O ZAPISIE DO PAMIĘCI GŁÓWNEJ GDY WYLATUJE Z CACHE


### Omów zasadę przetwarzania potokowego
Przetwarzanie potokowe - *pipelining*.
Krótki wstęp - przetworzenie każdego rozkazu dzieli się na kilka etapów, realizowanych przez kilka różnych układów. To trochę jak przekazywanie worka z piachem w szeregu ludzi. Można czekać, aż jeden worek przejdzie przez cały łańcuszek i rozkaz się skończy, albo można przekazywać worek za workiem, żeby każdy miał ciągle coś do roboty.

todo obrazek


# Kolos 2

### Omów działania, jakie podejmuje procesor po przyjęciu zgłoszenia przerwania
* Identyfikuje linie urządenia zgłaszającego przerwanie
* Zamiętuje stan procesora poprzez przeniesienie rejestrów PSW na stos 
* Wykonywanie procedury obsługi znajduącego się pod konkretnym adresem( zeleżnym od urządzenia które zgłosiło przerwanie)
### Omów cykl zapisu i odczytu DRAMu
* Cykl odczytu - Pierwszą fazą żądania odczytu z pamięci stanowi pobranie adresu wiersza, w którym znajduje się komórka i zatwierdzeniu tego adresu, po ustabilizowaniu się stanu szyny adresowej sygnałem RAS (Row Address Strobe). Następnie na szynę adresową pamięci podawany jest adres kolumny zawierającej żądaną komórkę, który zatwierdzany jest sygnałem CAS (Column Address Strobe). Odstęp czasu pomiędzy sygnałami CAS i RAS wynika z konstrukcji pamięci i musi zapewnić czas nie tylko na ustabilizowanie się stanu szyny adresowej, lecz także na wysterowanie wiersza do odczytu.

* Cykl zapisu - pobranie adresu pod który będziemy zapisywać, synchronizacja RAS i CAS, wyzwolenie sygału WR zapisującego dane

### Omów zasady transmisji asynchronicznej i synchronicznej relizowanej portem szeregowym
* transmisja asynchroniczna -  częstotliwość zegara k razy większa od częstotliwości nadawania, długość znaku to 5-8 bitów, możliwa jest kontrola poprawności transmisji za pomocą bitu parzystości. Synchronizacja na podstawie przerwy w nadawaniu.
* transmisja synchroniczna - częstotliwość k razy większa od nadawnia, ilość znaków synchronizacji to 1 lub 2. Synchronizacja może być zewnętrzna lub wewnętrzna.
## Omów architekturę SIMD
Architektury systoliczne, to architektury specjalizowane do implementacji operacji macierzowych, przetwarzania
sygnałów i obrazów w czasie rzeczywistym. Globalnie synchronizowane procesory elementarne są połączone w
regularną siatkę. Każdy procesor jest połączony tylko z najbliższymi sąsiadami. Struktura wewnętrzna procesora,
w zależności od postawionego zadania, możę być bardzo prosta lub dążyć do stopnia ukomplikowania współczesnych
mikroprocesorów. Wyniki przetwarzania są uzyskiwane stopniowo.
1.18.1 Podstawowe struktury
* tablica z częściowym rozpowszechnianiem danych – komunikacja PE punkt-punkt oraz magistrala,
komunikacja zewnętrzna przez magistralę, rozbudowane układy sterujące, stopień wykorzystania >50%
• tablica heksagonalna – komunikacja między procesorami typu punkt-punkt, komunikacja zewnętrzna tylko
za pomocą procesorów brzegowych, prosta konstrukcja, duża liczba PE, niski stopień ich wykorzystania
(<50%)
* tablica przetwarzania potokowego – komunikacja jak w heksagonalnej, prosta konstrukcja, liczba PE
mniejsza niż w tablicy heksagonalnej, stopień ich wykorzystania – co najmniej 50%
• tablica typu wavefront – asynchroniczna komunikacja punkt-punkt, na zewnątrz tylko brzegowe, dobra
skalowalność, łatwe przeprogramowywanie, dobre parametry FTC, stopień wykorzystania PE to 50%
* tablica z rozpowszechnianiem danych – komunikacja tylko przez magistrale, rozbudowane układy sterujące, łatwa implementacja algorytmów, wysoki stopień wykorzystania PE
1.18.2 Parametry jakościowe
* Czas wykonania obliczeń – czas od rozpoczęcia pierwszej operacji, aż do ukończenia ostatniej
* Okres przetwarzania potokowego – odstęp między dwoma sukcesywnymi wynikami obliczeń procesora
* Okres bloku – odstęp między inicjacjami dwóch sukcesywnych bloków
* Wskaźnik użycia procesora
* Rozmiar tablicy – tablica nie może być nieograniczona, czasem musi być ona mniejsza od rzeczywistego
rozmiaru problemu, czasem możemy też użyć tablicy jednowymiarowej do zasymulowania tablicy 2D
 Pamięć lokalna – używana do rozszerzenia fizycznej tablicy do znacznie większej tablicy wirtualnej
10
1.18.3 Transputery
Transputer to mikroprocesor w jednym układzie scalonym, zaprojektowany specjalnie do obliczeń równoległych
(szybka komunikacja i łatwość połączenia z innymi transputerami). Wraz z nim został opracowany język programowania równoległego OCCAM.
W skład transputera wchodzi procesor typu RISC, wewnętrzna pamięć RAM oraz łącze pamięci zewnętrznej,
która umożliwia adresowanie w przestrzeni 4 GB. Do komunikacji z innymi transputerami wykorzystywane są
cztery kanały DMA

# Kolos 3
### Omów przetwarzanie potokowe
###### Było wyżej

### Specjalne tryby DRAM
* odczyt-modyfikacja-zapis – adres kolumny nie musi zostać zestrobowany, można natomiast użyć adresu
z impulsów /CAS niskiego potencjału i następnie dokonać zapisu w przeciągu kilku nanosekund.
* tryb stronicowy – wiersz DRAM pozostaje otwarty przez utrzymywanie niskiego potencjału /RAS w trakcie
trwania odczytów bądź zapisów odosobnionym impulsem CAS.
* tryb nakładkowy – cztery impulsy CAS mają dostęp do czterech sekwencyjnych lokacji w wierszu.

### Charakterystyka koprocesora

### Omów czym jest SIMD



# Kolos 4
### DMA: rola, tryby pracy
**DMA - Direct Memory Access** - służy do tego, żeby procesor nie musiał zajmować się prostymi głupotami, w rodzaju przesyłania danych z jednego urządzenia do drugiego.

Przykładowo, gdy chcemy z pamięci wysłać coś przezmodem, procesor musiałby pobrać te dane z pamięci i wysłać do modemu. DMA robi to za niego, dzięki czemu procesor może zająć się w tym momencie czymś innym.

DMA nie jest od myślenia, jest od prostych zadań. Procesor wydaje rozkaz, DMA zasuwa.

###### Tryby pracy
* **Blokowy** - DMA dostaje rozkaz i zajmuje się pchaniem danych. Po prostu. Procesor przez ten czas siedzi cicho, bo nie ma dostępu do magistrali danych. Może obliczać sobie jakieś rzeczy we własnym zakresie, ale najpewniej nie będzie miał dostępu do urządzeń zewnętrznych, więc ma ograniczone pole do popisu. Za to układ DMA bez ograniczeń rozwija pełne prędkości
* **Wykradanie taktów** - Procesor daje DMA dostęp do magistrali danych, kiedy chce. DMA musi czekać, dopóki procesor da mu zielonego światła
* **Zgodnie z zapotrzebowaniem** - Ustalany jest rozmiar "bloków", na jakie DMA podzieli dane do przesłania. Jeśli po przesłaniu pojednynczego bloku procesor nie upomina się o dostęp do magistrali, przesyłany będzie kolejny blok i tak dalej

### Odczyt ROM
Pierwszą fazą jest ustawienie adresu zczytanego w strukturze pamięci (linie adresowe). Uaktywnienie linii MEMREQ na  podstawie  adresu,  a  także  linii  CE  (logicznego  wyboru  danego  układu  pamiętającego) – stanem niskim. Następnie uaktywnia się linię odczytu i pobiera dane.


### Zapis SRAM
### Adresowanie współbieżne i izolowane
### Tryby pracy układu czasowo-licznikowego

# Kolos 5 (to jakaś grupa śmierci)
### Omów przetwarzanie potokowe
###### Było wyżej
### Omów specjalne tryby pracy DRAM
###### Było wyżej
### Charakterystyka koprocesora MIMD

# Kolos 6
### Odczyt ROM i zapis RAM
### tryby czasowo licznikowe
### DMA
### Współadresowanie i adresowanie izolowane 
###### Było wyżej
