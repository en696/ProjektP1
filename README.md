

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

Celem projektu jest pokazanie jednego z sposobów tworzenia (Projektowania i wdrażania) nowoczesnej infrastruktury i środowiska pod aplikacje webowa.
Projekt ukazuje  wdrożenie dwóch aplikacji webowych które zostały zkonteryzowana w dokerze i zostały osadzone w google cloud a są zarzadzania za  pomocą kubernetesa.
Wykorzystam  moduły  odpowiedzialnych za wysoka dostępność, oraz omówię  zasady działania sieci oraz poruszę kwestie skalowalności i łatwości aktualizacji aplikacji webowej bez przerw w  jej działaniu.
Do kierowania użytkowników do odpowiednej aplikacji użyje ingressa.


#### 2. Opis teoretyczny
####	2.1 Infrastruktura fizyczna jako klucz do wysokiej dostępności i niezawodności

Utworzenie wysoko dostępnego środowiska w dzisiejszych czasach jest bardzo ważne jednak większość firm nie stać na to aby to zrobić lokalnie ponieważ do wysokiej dostępności i niezawodności potrzebna nam jest wysoko dostępna infrastruktura która jest droga w wdrożeniu oraz utrzymaniu. Wymaga znacznie więcej uwagi administratorów oraz jest wyższa szansa na awarie niż utworzenia tego środowiska pod aplikacje w chmurze oraz daje nam możliwość bardzo łatwego skalowania całej infrastruktury.Dlatego wybrałem chmure googla

#### 	2.2 Kubernetes jako narzędzie do łatwego zarzadzania kontenerami Dockera

Docker jest platformą bazującą na metodzie wirtualizacji LXC na poziomie systemu operacyjnego (tzw. "konteneryzacji"). Pozwala ona na odizolowanie aplikacji od systemu operacyjnego, a także przydział konkretnych zasobów – CPU, RAM czy HDD.

Kubernetes to oprogramowanie do automatyzacji instalacji, skalowania i zarządzania aplikacji skonteneryzowanych. Dla ułatwienia
zarządzania aplikacjami Kubernetes grupuje kontenery, na których pracują instancje komponentów aplikacji w jednostki logiczne zwane Podami.
Kubernetes bazuje na piętnastu latach doświadczeń Google w przetwarzaniu masowych obciążeń na dziesiątkach tysięcy komputerów (projekty Borg, Omega). Od 2014 roku Kubernetes został oddany społeczności na licencji open source. Od tego czasu liczba deweloperów znacznie się powiększyła.

Pod w Kubernetesie
Najmniejsza jednostka w kubernetes. To ona odpowiedzialna jest za deklarowanie kontenerów odpalanych na tym samym host/node. Ogólnie kiedy ktoś mówi o Pod to ma namyśli przynajmniej jeden działający kontener

Node w Kubernetesie
Maszyną fizyczną lub wirtualną na której będą instalowane Pody. do utworzenia klastra  potrzebne  są co najmniej 3 maszyny 2 node i jeden master który będzie odpowiedzialny za zarzadzanie pozostałymi nodami

####	2.3 Services czyli moduł odpowiedzialny za sieć w Kubernetes

Komponent jest  odpowiedzialny za komunikację miedzy podami jak i również za wystawienie podow  na świat.
Komunikacja pomiędzy kontenerami w Podzie odbywa się w jadrze linuxa bez udziału sieci tzn  jeśli w jednym podzie mamy 2 lub więcej kontenerów wszystkie kontenery będą miały wspólny adres ip.
Service może występować w 3 trybach ClusterIP, LoadBalancer, NodePort. Aby zrozumieć jak działa LoadBalancer wpierw trzeba zrozumieć jak działa ClusterIP i NodePort ponieważ loadbalanser z nich korzysta.

Tryb sieci ClusterIP służy wyłącznie do komunikacji wewnątrz klastra tzn tylko  miedzy podami i nodami

Tryb sieci  NodePort to najbardziej prymitywny sposób na uzyskanie zewnętrznego ruchu bezpośrednio do Twojej usługi. NodePort, jak sama nazwa wskazuje, otwiera określony port we wszystkich węzłach (VM), a każdy ruch przesyłany do tego portu jest przekazywany do usługi.

Najlepszą metoda do utworzenia wysoko dostępnej usługi jest Service w trybie Load Balancer, aby load balancer działał potrzebuje wykorzystać loadbalanser dostawcy chmury prywatnej lub publicznej w moim przypadku jest to loadbalanser chmury googla.
tryb sieci NodePort i ClusterIP zostanie utworzony samoczynnie po utworzeniu load balancera .

![Diagram](https://github.com/en696/ProjektP1/blob/master/Rysunek211.jpg)


Jak widzimy na obrazku komunikacja w clusterIP odbywa się za pomocą wirtualnego routera który pozwala komunikować się wszystkim podom w klastrze. Z poda  ngnix-hellow 1 możemy uzyskać dostęp do Pod ngnix-hellow 2 pomimo ze znajdu się na innym nodzie i w innej adresaci  , eth0 jest połączony cbr0 za pomocą mostu a interfejsu Veth0 jest do niego przyłączony.
Komunikacja pomiędzy podami w tym samym nodzie odbywa się za pomocą bridga
Kubernetes pody są nie stałe często są ubijane , replikowane , skalowane i zamieniane na inne dlatego nie wolno się przywiązywać do adresaci podów ponieważ kubernetes sam nadaje adresy ip podom z takiej puli jak mu wydzielimy u mnie jest to 10.52.0.0/14. Używam chmury Gogola do klastra kubernetes a wiec do adresaci nodow tez nie nalezy się przywiązywać ponieważ one tez podlegają skalowaniu . Kubernetes używa sieci nakładkowej jest ona zaznaczona na obrazku jako overlay w moim przypadku jest to flannel ale to można zmienić istnieje wiele innych rozwiązań do kubernetesa.  Kubernetes z modułem falannel sam za nas przypisze addresy ip, bedzie pilnował aby żaden pod nie dostał tego samego adresu ip  i stworzy mosty miedzy interfejsami oraz utworzy routing

Flannel uruchamia małego, pojedynczego agenta binarnego wywoływanego flanneldna każdym hoście i odpowiada za przydzielanie dzierżawy podsieci każdemu hostowi, wstępnie skonfigurowanej przestrzeni adresowej. Flannel używa API Kubernetes lub etcd bezpośrednio do przechowywania konfiguracji sieci, przydzielonych podsieci i wszelkich danych pomocniczych (takich jak publiczny adres IP hosta). Pakiety są przekazywane za pomocą jednego z kilku mechanizmów backendu, w tym VXLAN.

#### 2.4 ingress
W przeciwieństwie do wszystkich powyższych obiektów, Ingress w rzeczywistości nie jest typem service. Zamiast tego znajduje się przed wieloma usługami i działa jako "inteligentny router" lub punkt początkowy w klastrze.
Domyślny kontroler ingres ngnix uruchomi moduł równoważenia obciążenia HTTP (S) . Umożliwi to wykonanie routingu opartego na ścieżkach i subdomenach do usług zaplecza. Na przykład można wysłać wszystko na adres edomin.pl/hellow-aps-1 do usługi hellow-aps-1 lub pod eldorotos/hellow-aps-2 ścieżka do usługi hellow-aps-2
![Diagram](https://github.com/en696/ProjektP1/blob/master/Rysunek11.jpg)

Obrazek przedstawia sposób działania ingressa użytkownik łączy się z nginx-ingress-controller która jest pod adresem 35.228.100.152. Następnie wybiera z która aplikacja chce się połączyć poprzez podanie nazwy aplikacji po / jeśli użytkownik wpisze nieprawidłowo nazwę aplikacji lub taka nie istnieje dla podanej domeny to użytkownik zostanie przekierowany do nginx-ingress-default-backend. W moim przykładzie jest tylko jedna domena edomin.pl lecz ingres może wykorzystywać wiele domen jednocześnie. Później użytkownik zostaje przekierowany do usługi ClusterIP która już łączy się bezpośrednio  z podami w których jest aplikacja. Ingress kontroler rozrzuca ruch przemiennie na pody
Aplikacja w podzie jest dostępna na porcie 80 oraz 443.


####	2.5 Kubernetes replicaset i auto scaling

Replicaset jest obiektem kubernetesa który zapewnia nam  skalowalność podow.
Pody możemy skalować ręcznie lub ustawić Auto scaling.
Przy tworzeniu pliku konfiguracyjnego  obiektu replicaset należy przypisać im etykiety po których kubernetes będzie rozpoznawał pody które maja zostać zreplikowane oraz będzie piłował aby zawsze była odpalona przynajmniej minimalna liczba podow które zadeklarujemy.
Obiekt jest odpowiedzialny za utrzymanie ciągle działających podow jeśli jakiś pod ulegnie awarii kubernetes restartuje tego poda , lub kiedy utracimy komunikacja z nodem zostanie utracona poprzez awarie noda kubernetes automatycznie utworzy  pody na innym nodzie

Auto scaling w kubernetesie zapewnia nam duża elastyczność i odciąża prace admina ponieważ mamy możliwość wybrania ile maksymalnie i minimalnie  chcemy mieć odpalonych podów. Podczas dużego obciążenia naszej aplikacji, kubernetes na bieżąco monitoruje zużycie poda i jak osiągniemy na nim podana przez nas w procentach utylizacje procesora lub pamięci ram kubernetes  będzie skalował pody do osiągniecia maximum które to my mu wyznaczymy a gdy obciążenie się zmniejszy, Kubernetes będzie stopniowo zmniejszał liczbę podów w clustrze do minimum   

####	2.6 Kubernetes deployment , rolling back, rolling update,

Obiekt deployment jest obiektem nadrzędnym nad replicaset tzn w obiekcie deployment możemy napisać plik konfiguracyjny replicaset i autoslacer. Obiekt jest odpowiedzialny za utrzymanie ciągle działających podow jeśli jakiś pod ulegnie awarii kubernetes restartuje tego poda.
Jeśli chcemy uzyskać jak najwyższa niezawodność naszej aplikacja powinna być umieszczona w obiekcie deployment. Ponieważ deployment jest odpowiedzialny za rolling update który natomiast daje nam możliwość aktualizacji aplikacji bez konieczności jej zatrzymywania.
Dzieje się to w taki sposób iż mamy powiedzmy odpalone 4 pody, to każdy pod po kolei jest zastępowany nowa wersja aplikacji dzięki czemu użytkownik zalogowany do naszej aplikacji może  w ogóle się nie zorientować ze właśnie ta aplikacja została zaktualizowana.
roling back natomiast daje nam możliwość wycofania tego procesu jeśli okaże się ze nasza nowa wersja aplikacji mam poważne błędy które uniemożliwiają korzystania z naszej aplikacji.
Pliki konfiguracyjny deploymentu tak jak innych obiektów  jest zapisywana w formacie  yaml lub JSON

#### 2.7 HELM

Helm pomaga w zarządzaniu aplikacjami Kubernetes - Helm Charts pomaga definiować, instalować i aktualizować nawet najbardziej złożoną aplikację Kubernetes.
Dzieki Helmowi mozna stworzyć jeden plik który nazywa sie Charts w którym zdefinujemy wszystkie obiekty kubernetesa takie jak deployment service i config mapy

#### 2.8  ConfigMaps

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


#### 3.3 aplikacja webowa

Aplikacja ma za zadania pokazać działanie Ingressa który udostępnia nam aplikacje w sieci publicznej pod domena edomin.pl oraz działa jak load balancer rozrzuca ruch na różne nody i pody.
Aplikacje pokazują nazwę hostname poda i jego adres ip
Aplikacja posiada opcje auto-refresh
Pliki konfiguracyjne obiektu deployment obu aplikacji  zostały dodane do githuba.
Jest to hellow-app-1.yaml i hellow-app-2.yaml.
Na obu dwóch aplikacjach została uruchomiona replikacja na minimum 3 pody
I zostało dodane autoscaling uzależnione od utylizacji procesora jeśli jest równe bądź większe od 80% aplikacje automatycznie się zareplikują do max 8 podow
Obiekt ConfigMaps jest odpowiedzialny za przechowywanie certyfikatów SSL dla obydwóch aplikacji
Na aplikacjach dzięki swojej liczebności można przeprowadzać roling update.

#### 3.4 Ingressa

Za pomocą helma został utworzone dwa obiekty nginx-ingress-default-backend oraz nginx-ingress-controller.
nginx-ingress-default-backend obsługiwać wszystkie ścieżki URL i hosty, których kontroler Nginx nie rozumie tj. Wszystkie żądania, które nie są odwzorowane za pomocą Ingressu.
nginx-ingress-controller działa jako usługa load balancer na addresie 35.228.100.152 i na dwuch portach 443 oraz 80 rozrzuca ruch HTTP oraz HTTPS.
Została dodana adnotacja która dodaje dodatkowa autoryzacje trzeba podać login i hasło przed dostępem do aplikacji hellow-app-1,
został utworzony obiekt ingres hellow-app-ingress za pomocą   pliku hellow-app-ingress.yaml obiekt określa reguły ścieżki oraz na który port zostanie przekazanie zadanie po nazwie domeny/subdomeny.

#### 3.5 debugowanie problemów


<<<<<<< HEAD
Aby sprawdzić ile nodow tworzy cluster i czy wszystkie odpowiadaja nalezy wdyac polecenie kubectl get node (W moim przypadku powiny być 3 nody).

![Diagram](https://github.com/en696/ProjektP1/blob/master/2.jpg)

Wszystkie moje pody odpalone sa w default namespace a  wieć zeby sprawdzić czy sa uruchomione nalezy wydać polecenie  kubectl get pod. Powiny być odpalone 3 pody pierwszej aplikacji oraz 3 pody drugiej aplikacji oraz 2 pody odpowiedzialne za backed oraz 2 ingress controler

![Diagram](https://github.com/en696/ProjektP1/blob/master/1.jpg)

Sprawdzanie odpalonych usuług sieciowych tzw services wykonujemy za pomaca kubectl get service powiny działac 4 usługi
2 Usługi typu clusterIP od aplikacji, jeden jako loudbalanser oraz jeden jako Ruch przychodzący

![Diagram](https://github.com/en696/ProjektP1/blob/master/3.jpg)

Do kazdego kontenera mozemy sie zalogowac za pomoca shell google cloud używajac polecenia kubectl exec -ti nazwa poda uzyskana za pomoca kubectl get node  sh

![Diagram](https://github.com/en696/ProjektP1/blob/master/5.jpg)



#### 4 Wnioski

W dzisiejszych czasach coraz większy udział w aplikacjach maja aplikacje webowe , każdemu twory takiej aplikacji zależy na tym aby była ciągle aktywna i stabilnie działała. Jest to bardzo trudne zadanie zapewnić bardzo wysoka dostępność. Przykładowy projekt który przedstawiłem jest wstanie rozwiać znaczna większość problemów przed którymi stoją administratorzy aplikacji, Rozwiązanie które przedstawiłem rozwiązuje nam takie problemy jak potrzeba zatrzymania aplikacji podczas aktualizacji aplikacji. czy auto skalowanie ilości aplikacji podczas dużego obciążenia aplikacji co daje nam szybka odpowiedz na to że chwilowo jest znacznie większy ruchu. dockeryzacja aplikacji jest przydatna jeśli mamy zamiar odpalić duża liczbę aplikacji ponieważ zyskujemy mniejsze utylizacje komponentów porównując to np. z zwykłym serwer czy nawet z serwer zwizualizowanym. docker i kubernetes   rozwiązuje nam także problem z backupami i przywracaniem z backupów ponieważ każdy pod który ulegnie awarii jest automatycznie restartowany a jeśli po kilku razach nie zacznie działać to zostaje osierocony a następnie zastąpiony nowy sprawnym podem . Jeśli ulegnie awarii node to pody automatycznie wstaną na innych nodach w tym klastrze oraz  mamy do dyspozycji role back możemy łatwo przywrócić aplikacje do stanu przed źle działająca nowa wersja aplikacji . Mój przykład przedstawia również sposób tworzenia load balancer który jest połączony z ingress kontrolerem zapewnia nam to wysoką wydajność oraz mamy możliwość przypisanie do jednego adresu publicznego dla wielu domen i subdomen. Rozwiązanie chmurowe może okazać się droższe na dłuższą metę  od rozwiązania lokalnego jednak gdy nasza aplikacja szybko rośnie i potrzebujemy dużej elastyczności lub jeśli zależy nam na jak najwyższej wydajności aplikacji oraz niezawodności wtedy rozwiązanie które przedstawiłem jest doskonałe.


#### 5 Lektrua
https://blog.laputa.io/kubernetes-flannel-networking-6a1cb1f8ec7c
https://medium.com/@anilkreddyr/kubernetes-with-flannel-understanding-the-networking-part-2-78b53e5364c7
https://itnext.io/managing-ingress-controllers-on-kubernetes-part-2-36a64439e70a
https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/
https://github.com/kubernetes/ingress-nginx
https://kubernetes.io/docs/concepts/services-networking/service/
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
https://helm.sh/
https://medium.com/@tao_66792/how-does-the-kubernetes-networking-work-part-3-910ae2f8dc08
