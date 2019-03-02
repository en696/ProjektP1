

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
Projekt ukazuje  wdoroznie aplikacji webowej któraz została zkonteryzowana w dokerze i została osadzona google cloud a jest zarzadzania za  pomoca kubernetesa. Omowie moduły  odpowiedzalnych za wysoka dostepnosc i  zasad działania sieci oraz omowienie kwesti skalowalnosci i łatwosci aktualizacji aplikacji webowej bez przerw w  jej działaniu oraz jak działa ingres.

#### 2. Opis teoretyczny
####	2.1 Infrastruktura fizyczna jako klucz do wysokiej dostepnosci i niezawodnosci

Utworzenie wysoko dostępnego środowiska w dzisiejszych czasach jest bardzo ważne jednak wiekszość firm nie stać na to aby to zrobić lokalnie ponieważ do wysokiej dostepności i niezawodnosci potrzebna nam jest wysoko dostepna infrastruktura która jest droaga w wdrozeniu oraz utrzymaniu. Wymaga znacznie wiecej uwagi administratorów oraz jest wyzsza szansza na awarie niz utworzenia tego srodowiska pod aplikacje w chmurze i często znacznie tansza oraz daje nam mozliwosc bardzo łatwego skalowania całes infrastruktury.

#### 	2.2 Kubernetes jako narzędzie do łatwego zarzadzania kontenerami Dockera

Docker jest platformą bazującą na metodzie wirtualizacji LXC na poziomie systemu operacyjnego (tzw. "konteneryzacji"). Pozwala ona na odizolowanie aplikacji od systemu operacyjnego, a także przydział konkretnych zasobów – CPU, RAM czy HDD.

Kubernetes to oprogramowanie do automatyzacji instalacji, skalowania i zarządzania aplikacji skonteneryzowanych. Dla ułatwienia
zarządzania aplikacjami Kubernetes grupuje kontenery, na których pracują instancje komponentów aplikacji w jednostki logiczne zwane Podami.
Kubernetes bazuje na piętnastu latach doświadczeń Google w przetwarzaniu masowych obciążeń na dziesiątkach tysięcy komputerów (projekty Borg, Omega). Od 2014 roku Kubernetes został oddany społeczności na licencji open source. Od tego czasu liczba deweloperów znacznie się powiększyła.

 Pod w Kubernetesie
Najmniejsza jednostka w kubernetes. To ona odpowiedzialna jest za deklarowanie kontenerów odpalanych na tym samym host/node. Ogólnie kiedy ktoś mówi o Pod to ma namyśli przynajmniej jeden działający kontener

Node w Kubernetesie
Maszyną fizyczną lub wirtualną na której będą instalowane Pody. do utworzenia klastra aby utworzyć klaster potrzebne  są co najmniej 3 maszyny 2 node i jeden master który będzie odpowiedzialny za zarzadzanie pozostałymi nodami  


####	2.2 Services czyli moduł odpowiedzialny za sieć w Kubernetes

Pierwszym komponetem którego omówie jest Service.

Komponent jest  odpowiedzialny za komunikację miedzy podami jak i równiez za wystawienie podow  na świat.
Komunikacja pomiędzy kontenerami w Podzie odbywa się w jadrze linuxa bez udziału sieci tzn  jeśli w jednym podzie mamy 2 lub wieicej kontenerów wszystkie kontenery bedą miały wspolny adres ip.
Service może wystepować w 3 trybach ClusterIP, LoadBalancer,NodePort. Aby zrozumieć jak działa LoadBalancer wpierw trzeba zrozumieć jak działa clusterIP i NodePort poniewaz loadbalanser z nich kozysta.

Tryb sieci ClusterIP słuzy wyłacznie do komunikacji wewnatrz klastra tzn tylko  miedzy podami i nodami

Tryb sieci  NodePort to najbardziej prymitywny sposób na uzyskanie zewnętrznego ruchu bezpośrednio do Twojej usługi. NodePort, jak sama nazwa wskazuje, otwiera określony port we wszystkich węzłach (VM), a każdy ruch przesyłany do tego portu jest przekazywany do usługi.

Najlepszą metoda do utworzenia wysoko dostępnej usługi jest Service w trybie LoadBalancer, aby loadbalanser działał potrzebuje wykorzystać loadbalanser dostawcy chmury prywatnj lub publicznej w moim przypadku jest to loadbalanser chmury googla. Jesli wybierzemy typ loadbalanser
tryb sieci NodePort i ClusterIP zostanie utworzony samoczynie.

![Diagram](https://github.com/en696/ProjektP1/blob/master/Rysunek211.jpg)


Jak widzimy na obrazku komunikacja w clusterIP odbywa sie za pomoca wirtualnego routera który pozwala komunikować sie wszystkim podom w klustrze tzn z podu  ngnix-hellow 1 możemy uzyskac dostep do Pod ngnix-hellow 2 pomimo ze znajdu sie na innym nodzie i w innej adresaci  , eth0 jest połaczony cbr0 za pomoca mostu a interface Veth0 jest do niego przyłaczony.
Komunikacja pomiedzy podami w wym samym nodzie odbywa się za pomoca bridga
Kubernetes pody sa nie stałe czesto sa ubijane , replikowane , skalowane i zamianiane na inne dlatego nie wolno się przywiazywać do adresaci podów ponieważ kubernetes sam nadaje adresy ip podom z takiej puli jak mu wydzielimy u mnie jest to 10.52.0.0/14. Urzywam chmury googla do klastra kubernetes a wiec do adresaci nodow tez nie nalezy się przywiazywać ponieważ one tez podlegaja skalowaniu . Kubernetes używa sieci nakładkowej jest ona zaznaczona na obrazku jako overlay w moim przypadku jest to flannel ale to mozna zmienic istnieje wiele innych roziwazan do kubernetesa.  Kubernetes z modułem falannel sam za nas przypisze addresy ip, bedzie pilnował aby zaden pod nie dostał tego samego addresu ip  i stworzy mosty miedzy interfajsami oraz utworzy routing

Flannel uruchamia małego, pojedynczego agenta binarnego wywoływanego flanneldna każdym hoście i odpowiada za przydzielanie dzierżawy podsieci każdemu hostowi, wstępnie skonfigurowanej przestrzeni adresowej. Flannel używa API Kubernetes lub etcd bezpośrednio do przechowywania konfiguracji sieci, przydzielonych podsieci i wszelkich danych pomocniczych (takich jak publiczny adres IP hosta). Pakiety są przekazywane za pomocą jednego z kilku mechanizmów backendu, w tym VXLAN.


#### 2.3 ingress
W przeciwieństwie do wszystkich powyższych obiektow, Ingress w rzeczywistości nie jest typem service. Zamiast tego znajduje się przed wieloma usługami i działa jako "inteligentny router" lub punkt początkowy w klastrze.
Domyślny kontroler ingres ngnix uruchomi moduł równoważenia obciążenia HTTP (S) . Umożliwi to wykonanie routingu opartego na ścieżkach i subdomenach do usług zaplecza. Na przykład można wysłać wszystko na adres edomin.pl/hellow-aps-1 do usługi hellow-aps-1 lub pod ścieżką eldorotos/hellow-aps-2 ścieżka do usługi hellow-aps-2
![Diagram](https://github.com/en696/ProjektP1/blob/master/Rysunek11.jpg)

Obrazek przedstawia sposób działania ingresa użtkownik łaczy się z nginx-ingress-controller która jest pod addresem 35.228.100.152. Nstepnie wybiera z która aplikacja chce sie połaczyć poprzez podanie nazwy aplikacji po / jesli uzytkownik wpisze nieprawidłowonazwe aplikacji lub taka nie istnieje dla podanej domeny to użytkownik zostanie przekierowany do nginx-ingress-default-backend. W moim przykładzie jest tylko jedna domena edomin.pl lecz ingres moze wykorzystywać wiele domen jednoczesnie. Pozniej użytkownik zostaje przekierowany do suługi cluster ip ktora juz łaczy sie bezposrednio  z podami w ktorych jest aplikacja.Ingress kontroler rozrzuca ruch przemienie na pody
Aplikacja w podzie jest dostepna na porcie 80 oraz 443.


####	2.4 Kubernetes replicaset i auto scaling

Replicaset jest obiektem kubernetesa który zapewnia nam  skalowalnosc podow.
Pody możemy skalować recznie lub ustawić Auto scaling.
Przy tworzeniu pliku konfiguracyjnego  obiektu replicaset nalezy przypisać im etykiety po których kubernetes bedzie rozpoznawał pody które moaja zostać zreplikowane oraz bedzie pilował aby zawsze była odpalona przynajmniej minimalna liczba podow które zadeklarujemy.
Obiekt jest odpowiedzialny za utrzymanie cigle działajacyh podow jesli jakis pod ulegnie awari kubernetes restartuje tego poda , lub komunikacja z node zostanie ustracona poprzez awarie noda kubernetes automatycznie utworzy  pody na innym nodzie

Auto scaling w kubernetesie zapewnia nam duza elastycznasc i odciaza prace admina ponieważ mamy mozliwość wybrania ile maksymalnie i minimalnie  chcemy miec odpalonych podów. Podczas duzego obciazenia naszej aplikacji, kubernetes na biezaco monitoruje zuzycie poda i jak osiagniemy na nim podana przez nas w procentach utylizacje procesora lub pamieci ram kubernetes  bedzie skalował pody do osiagniecia maximum które to my mu wyznaczymy a gdy obciazenie sie zmniejszy, Kubernetes bedzie stopniowo zminiejszał liczbe podów w clustrze do minimum   

####	2.3 Kubernetes deployment , rolling back, rolling update,

Obiekt deplyment jest obiektem nadrzadnym nad replicaset tzn w obiekcie deployment mozemy napisać plik konfiguracyjny replicaset i autoslacer.Obiekt jest odpowiedzialny za utrzymanie cigle działajacyh podow jesli jakis pod ulegnie awari kubernetes restartuje tego poda.Obiektem
Jeśli chemy uzyskać jak najwyzsza niezawodnosc naszej aplikacja powina być umieszczona w obiekcie deployment. Ponieważ deployment jest odpowiedzialny za rolling update który natomiast daje nam mozliwosc updatowania plikacji bez koniecznosci jej zatrzymywania.
Dzieje sie to w taki sposob iz mamy powiedzmy odpalone 4 pody, to kazdy pod po kolei jest zastepowany nowa wersia aplikacji dzieki czemu uzytkownik zalogowany do naszej aplikacji moze wogule sie nie zorientowac ze własnie ta aplikacja została zaktualizowana.
roling back natomiast daje nam mozliwosc cofniencia tego procesu jesli okaze sie ze nasza nowa wersja aplikacji mam powazne bledy ktore uniemozliwiaja korzystania z naszej aplikacji.
Pliki konfiguracyjny deploymentu tak jak innych obiektów  jest zapisywana w formacie  yaml lub JSON

#### 2.5 HELM

Helm pomaga w zarządzaniu aplikacjami Kubernetes - Helm Charts pomaga definiować, instalować i aktualizować nawet najbardziej złożoną aplikację Kubernetes.
Dzieki Helmowi mozna stworzyć jeden plik który nazywa sie Charts w którym zdefinujemy wszystkie obiekty kubernetesa takie jak deployment service i config mapy

#### 2.7  ConfigMaps

Pliki ConfigMaps wiążą pliki konfiguracyjne, argumenty wiersza poleceń, zmienne środowiskowe, numery portów i inne artefakty konfiguracji z kontenerami i komponentami systemu Pods w czasie wykonywania. ConfigMaps umożliwia oddzielenie konfiguracji od poda i komponentów, co pomaga utrzymać przenośność , ułatwia konfigurowanie ich konfiguracji i zarządzanie nimi
ConfigMaps nie zawsze jest dobrym pomysłem ponieważ przechowuje niezaszyfrowane informacje o konfiguracji.

#### 3. Część praktyczna

#### 3.1 Infrastruktura

W Google cloud zostało utwoczony cluster kubernetesa.
Klaster kubernetesa składa się z 3 nodów, są 3 maszyny wirtualne a każda o parametrach
HDD 100 GB
CPU Intel(R) Xeon 1x2GZh
RAM 3.75 GB
OS jest to googlowski OS opart o jadra linuxa version 4.14.65
master nodem jest api google cloud

#### 3.2 Infrastruktura sieci




#### 3.3 aplikacja webowa

Aplikacja ma za zadania pokazać działanie Ingressa ktory udostepnia nam aplikacje w sieci publicznej pod domena edomin.pl oraz działa jak loudbalanser rozrzuca ruch na rozne nody i pody.
Pokazuje nazwe hostname poda i jego adress ip
Aplikacja posiada opcje auto-refresh
Pliki kofiguracyjne obiektu deployment obu aplikacji  zostały dodane do githuba.
Jest to hellow-app-1.yaml i hellow-app-2.yaml.
Na obu dwóch aplikacjach została uruchomiona replikacja na minimum 3 pody
I zostało dodane autoscaling uzaleznione od utylizacji procesora jesli jest rowne badz wieksze od 80% aplikacje automatycznie sie zreplikuja do max 8 podów
Obiekt ConfigMaps jest odpowiedzialny za przechowywanie certyfikatów SSL dla obydwuch aplikacji
Na aplikacjach dzieki swojej liczbnosci mozna przeprowadzac Roling update.

#### 3.4 Ingressa

Za pomocą helma został utworzone dwa obiekty nginx-ingress-default-backend oraz nginx-ingress-controller.
nginx-ingress-default-backend obsługiwać wszystkie ścieżki URL i hosty, których kontroler Nginx nie rozumie tj. Wszystkie żądania, które nie są odwzorowane za pomocą Ingressu.
nginx-ingress-controller działa jako usługa loudbalanser na addresie 35.228.100.152 i na dwuch portach 443 oraz 80 rozrzuca ruch HTTP oraz HTTPS.
Została dodana adnotacja która dodaje authentykacje przed dostepem do aplikacji hellow-app-1
został utworzony obiekt ingres hellow-app-ingress za pomoca   pliku hellow-app-ingress.yaml obiekt określa reguły ścieżki oraz na który port zostanie przekazanie ządanie po nazwie domeny/subdomeny


#### 4 Wnioski
W dzisiejszych czasie budowanie wysoko dostepnego srodowiska pod aplikacje webowa podstawa
W dzisiejszych coraz wiekszy udział w aplikacjach maja aplikacje webowe , kazdemu twory takiej aplikacji zalezy na tym aby była cigle aktywna . Jest to bardzo trudne zadanie zepewnic bardzo wysoka dostepnosc.Przykładowy projekt który przedstawiłem jest wstanie roziwać znaczna wiekszosc problemów które maja tworcy aplikacji, Rozwiazanie ktore przedstawiłem rozwiazuje nam takie problemy jak potrzeba zatrzymania aplikacji podczas aktualizacji aplikacji czy autoskalowanie ilosci aplikacji podczas duzego obciazenia aplikacji co daje nam szybka odpowiedz na to że chwilowo jest znacznie wiekszy ruchu.dockeryzacja aplikacji jest przydatna jesli mamy zamiar odpalic duza liczbe aplikacji poniewaz zyskujemy mniejsze utylizacje komponentow porownujac to np z zwykłym serverm czy nawet z serverm zwirualizowanym.dockeryzacja i kubernetes  aplikacji rozwiazuje nam takze pbroblem z backupy i przywracaniem z backupow ponieważ kazdy pod który ulegnie awari jest automatycznie restartowany a jesli ulegnie awari node to pody automatycznie wstana na innch nodach oraz  mamy do dyspozycji role back mozemy łatwo przywrocic aplikacje do stanu przed zle działajaca nowa wersja aplikacji. Rozwiazanie chmurowe moze okazac sie drosze od rozwiazania lokalnego jednak gdy nasza aplikacja szybko rosnie i potrzebujemy duzej elastycznosci lub jesli zalezy nam na jak najwyzszej wydajnosci aplikacji oraz niezawodnosci wtedy rozwiazanie ktore przedstawiłem jest doskonałe 


####5 Lektrua
