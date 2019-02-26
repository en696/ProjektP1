

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
Projekt ukazuje  wdoroznie aplikacji webowej któraz została zkonteryzowana w dokerze i została osadzona google cloud a jest zarzadzania za  pomoca kubernetesa. Omowie moduły  odpowiedzalnych za wysoka dostepnosc i  zasad działania sieci oraz omowienie kwesti skalowalnosci i łatwosci aktualizacji aplikacji webowej bez przerw w dostepnosci jej działaniu.

#### 2. Opis teoretyczny
### 	2.1 Infrastruktura fizyczna jako klucz do wysokiej dostepnosci i niezawodnosci

Utworzenie wysoko dostępnego środowiska w dzisiejszych czasach jest bardzo ważne jednak wiekszość firm nie stać na to aby to zrobić lokalnie ponieważ do wysokiej dostepności i niezawodnosci potrzebna nam jest wysoko dostepna infrastruktura która jest droaga w wdrozeniu oraz utrzymaniu. Wymaga znacznie wiecej uwagi administratorów oraz jest wyzsza szansza na awarie niz utworzenia tego srodowiska pod aplikacje w chmurze i często znacznie tansza oraz daje nam mozliwosc bardzo łatwego skalowania całes infrastruktury.

### 	2.2 Kubernetes jako narzędzie do łatwego zarzadzania kontenerami Dockera

Docker jest platformą bazującą na metodzie wirtualizacji LXC na poziomie systemu operacyjnego (tzw. "konteneryzacji"). Pozwala ona na odizolowanie aplikacji od systemu operacyjnego, a także przydział konkretnych zasobów – CPU, RAM czy HDD.

Kubernetes to oprogramowanie do automatyzacji instalacji, skalowania i zarządzania aplikacji skonteneryzowanych. Dla ułatwienia
zarządzania aplikacjami Kubernetes grupuje kontenery, na których pracują instancje komponentów aplikacji w jednostki logiczne zwane Podami.
Kubernetes bazuje na piętnastu latach doświadczeń Google w przetwarzaniu masowych obciążeń na dziesiątkach tysięcy komputerów (projekty Borg, Omega). Od 2014 roku Kubernetes został oddany społeczności na licencji open source. Od tego czasu liczba deweloperów znacznie się powiększyła.

 Pod w Kubernetesie
Najmniejsza jednostka w kubernetes. To ona odpowiedzialna jest za deklarowanie kontenerów odpalanych na tym samym host/node. Ogólnie kiedy ktoś mówi o Pod to ma namyśli przynajmniej jeden działający kontener

Node w Kubernetesie

Maszyną fizyczną lub wirtualną na której będą instalowane Pody. do utworzenia klastra aby utworzyć klaster potrzebne  są co najmniej 3 maszyny 2 node i jeden master który będzie odpowiedzialny za zarzadzanie pozostałymi nodami  

#Jedna z najwazniejszych rzeczy aby otrzymać wysokodestepna aplikacje jest zapewnienie bardzo szybkiego łacza i utworzenie loadbalansera który bedzie mogł rozrzucać ruch na wszystkie pody w aplikacji.

### 	2.2 Services czyli moduł odpowiedzialny za sieć w Kubernetes
Pierwszym komponetem którego omówie jest Service.

Komponent jest  odpowiedzialny za komunikację miedzy podami jak i równiez za wystawienie podow  na świat.
Komunikacja pomiędzy kontenerami w Podzie odbywa się w jadrze linuxa bez udziału sieci tzn  jeśli w jednym podzie mamy 2 lub wieicej kontenerów wszystkie kontenery bedą miały wspolny adres ip.
Service może wystepować w 3 trybach ClusterIP, LoadBalancer,NodePort. Aby zrozumieć jak działa LoadBalancer wpierw trzeba zrozumieć jak działa clusterIP i NodePort

![Diagram](https://github.com/en696/ProjektP1/blob/master/Rysunek211.jpg)

Tryb sieci CLusterIP słuzy wyłacznie do komunikacji wewnatrz klastra tzn tylko wyłacznie miedzy podami i nodami

Tryb sieci Node port słuzy

Jak widzimy na obrazku komunikacja w clusterIP odbywa sie za pomoca wirtualnego routera który pozwala komunikować sie wszystkim podom w klustrze tzn z podu  ngnix-hellow 1 możemy uzyskac dostep do Pod ngnix-hellow 2 pomimo ze znajdu sie na innym nodzie i w innej adresaci  , eth0 jest połaczony cbr0 za pomoca mostu a interface Veth0 jest do niego przyłaczony.
Komunikacja pomiedzy podami w wym samym nodzie odbywa się za pomoca bridga
Kubernetes pody sa nie stałe czesto sa ubijane , replikowane , skalowane i zamianiane na inne dlatego nie wolno się przywiazywać do adresaci podów ponieważ kubernetes sam nadaje adresy ip podom z takiej puli jak mu wydzielimy u mnie jest to 10.52.0.0/14. Urzywam chmury googla do klastra kubernetes a wiec do adresaci nodow tez nie nalezy się przywiazywać ponieważ one tez podlegaja skalowaniu . Kubernetes używa sieci nakładkowej jest ona zaznaczona na obrazku jako overlay w moim przypadku jest to flannel ale to mozna zmienic istnieje wiele innych roziwazan do kubernetesa.  Kubernetes z modułem falannel sam za nas przypisze addresy ip, bedzie pilnował aby zaden pod nie dostał tego samego addresu ip  i stworzy mosty miedzy interfajsami oraz utworzy routing

Flannel uruchamia małego, pojedynczego agenta binarnego wywoływanego flanneldna każdym hoście i odpowiada za przydzielanie dzierżawy podsieci każdemu hostowi, wstępnie skonfigurowanej przestrzeni adresowej. Flannel używa API Kubernetes lub etcd bezpośrednio do przechowywania konfiguracji sieci, przydzielonych podsieci i wszelkich danych pomocniczych (takich jak publiczny adres IP hosta). Pakiety są przekazywane za pomocą jednego z kilku mechanizmów backendu, w tym VXLAN.


Najlepszą metoda do utworzenia wysoko dostępnej usługi jest Service w trybie LoadBalancer aby loadbalanser działał potrzebuje wykorzystać loadbalanser dostawcy chmury prywatnj lub publicznej w moim przypadku jest to loadbalanser chmury googla. Jesli wybierzemy typ loadbalanser
tryb sieci NodePort i ClusterIP zostanie utworzony samoczynie,


#Dzieki metodzie NodePort mozemy wystawić poda na zewnatrz jednak ta metoda nie jest najlepszym loadbalanserm ponieważ zajmuje jeden port na wszystkich wezłach i taka aplikacja bedzie miała przypisany wysoki port , problemem jest rowniez to ze nie mozemy uzyc nazwy dns tylko musi to #byc adres ip.
Klaster Kubernetesa może zostać postawiony na maszynach z np CentOS ale duzo lepszym rozwiazaniem jest wykorzystanie do tego chmury pywatnej np Openstacka i modułu Magnum, lub roziwazań chmur publicznych najlepszym wyborem bedzie google cloud.

Obrazek ilustruje jak działa load balanser w k8s
