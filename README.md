

#Przyklad wysoko dostępnego i skalowalnego środowiska dla aplikacji webowych z wykorzystaniem  kubernetesa


Zbigniew Domin


===============================================================


# Spis Treści

#### 1. Wstęp    
#### 2. Opis teoretyczny
#### 3. Opis praktyczny
#### 4. Wnioski końcowe
#### 5. Literatura

#### 1. Wstęp
Celem projektu jest pokazanie jednego z sposobów tworzenia (Projetkowania i wdrazania) nowoczesnej infrastruktury i srodowiska pod aplikacje webowa.
Celem jest ukazanie modow odpowiedzalnych za wysoka dostepnosc i omowienie zasad działania sieci oraz omowienie kwesti skalowalnosci i łatwosci aktualizacji aplikacji webowej bez przerw w dostepnosci jej
Utworzenie wysoko dostępnego środowiska w dzisiejszych czasach jest bardzo ważne  ale żeby zrozumieć jak je stworzyć w kubernetesie trzeba poznać podstawowe pojęcia i zadac sobie pytanie czym jest kubernetes.
Kubernetes to oprogramowanie do automatyzacji instalacji, skalowania i zarządzania aplikacji skonteneryzowanych. Dla ułatwienia zarządzania aplikacjami Kubernetes grupuje kontenery, na których pracują instancje komponentów aplikacji w jednostki logiczne zwane Podami.
Kubernetes bazuje na piętnastu latach doświadczeń Google w przetwarzaniu masowych obciążeń na dziesiątkach tysięcy komputerów (projekty Borg, Omega). Od 2014 roku Kubernetes został oddany społeczności na licencji open source. Od tego czasu liczba deweloperów znacznie się powiększyła.

Czym jest Pod ?
Najmniejsza jednostka w kubernetes. To ona odpowiedzialna jest za deklarowanie kontenerów odpalanych na tym samym host/node. Ogólnie kiedy ktoś mówi o Pod to ma namyśli przynajmniej jeden działający kontener

Czym jest Node ??

Maszyną fizyczną lub wirtualną na której będą instalowane Pody. do utworzenia klastra aby utworzyć klaster potrzebne  są co najmniej 3 maszyny 2 node i jeden master który będzie odpowiedzialny za zarzadzanie pozostałymi nodami  

Kiedy juz rozumiemy podstawowe pojecia mozemy przejsc do komponetów kubernetesa które zapewnia nam wysoka dostapnosc skalowalnosc i self healing
#Jedna z najwazniejszych rzeczy aby otrzymać wysokodestepna aplikacje jest zapewnienie bardzo szybkiego łacza i utworzenie loadbalansera który bedzie mogł rozrzucać ruch na wszystkie pody w aplikacji. Zanim utworzę klaster

Pierwszym komponetem którego omówie jest Service.

Komponent jest odpowiedzialny za sieć w k8s
jest on odpowiedzialny za komunikację miedzy podami jak i równiez za wystawienie podow na swiat na świat.
Komunikacja pomiędzy kontenerami w Podzie odbywa się w jadrze linuxa bez udziału sieci tzn  jeśli w jednym podzie mamy 2 lub wieicej kontenerów wszystkie kontenery bedą miały wspolny adres ip.
Service może wystepować w 3 trybach ClusterIP, LoadBalancer,NodePort. Aby zrozumieć jak działa LoadBalancer wpierw trzeba zrozumieć jak działa clusterIP i NodePort
![Diagram](https://github.com/en696/ProjektP1/blob/master/Rysunek21.jpg)
$$
Jak widzimy na obrazku komunikacja w clusterIP odbywa sie za pomoca routera który pozwala komunikować sie wszystkim podom w klustrze tzn z podu  ngnix-hellow 1 możemy uzyskac dostep do Pod ngnix-hellow 2 pomimo ze znajdu sie na innym nodzie  , eth0 jest połaczony cbr0 za pomoca mostu a interface Veth0 jest do niego przyłaczony.Kubernetes pody sa nie stałe czesto sa ubijane , replikowane , skalowane i zamianiane na inne dlatego nie wolno się przywiazywać do adresaci podów ponieważ kubernetes sam nadaje adresy ip podom z takiej puli jak mu wydzielimy u mnie jest to 10.52.0.0/14. Urzywam chmury googla do klastra kubernetes a wiec do adresaci nodow tez nie nalezy się przywiazywać ponieważ one tez podlegaja skalowaniu . Kubernetes Jest badrdzo przyjemny w urzytkowaniu ponieważ sam za nas przypisze addresy ip i stworzy mosty miedzy interfajsami oraz utworzy routing  


Najlepszą metoda do utworzenia wysoko dostępnej usługi jest Service w trybie LoadBalancer aby loadbalanser działał potrzebuje wykorzystać loadbalanser dostawcy chmury prywatnj lub publicznej w moim przypadku jest to loadbalanser chmury googla. Jesli wybierzemy typ loadbalanser
NodePort i ClusterIP zostanie utworzony samoczynie,


#Dzieki metodzie NodePort mozemy wystawić poda na zewnatrz jednak ta metoda nie jest najlepszym loadbalanserm ponieważ zajmuje jeden port na wszystkich wezłach i taka aplikacja bedzie miała przypisany wysoki port , problemem jest rowniez to ze nie mozemy uzyc nazwy dns tylko musi to #byc adres ip.
Klaster Kubernetesa może zostać postawiony na maszynach z np CentOS ale duzo lepszym rozwiazaniem jest wykorzystanie do tego chmury pywatnej np Openstacka i modułu Magnum, lub roziwazań chmur publicznych najlepszym wyborem bedzie google cloud.

Obrazek ilustruje jak działa load balanser w k8s
