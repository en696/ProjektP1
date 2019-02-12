# ProjektP1
Wysoko dostępne środowisko w kubernetesie

Utworzenie wysoko dostępnego środowiska w dzisiejszych czasach jest bardzo ważne  ale żeby zrozumieć jak je stworzyć w kubernetesie trzeba poznać podstawowe pojęcia i zadac sobie pytanie czym jest kubernetes.
Kubernetes to oprogramowanie do automatyzacji instalacji, skalowania i zarządzania aplikacji skonteneryzowanych. Dla ułatwienia zarządzania aplikacjami Kubernetes grupuje kontenery, na których pracują instancje komponentów aplikacji w jednostki logiczne zwane Podami.
Kubernetes bazuje na piętnastu latach doświadczeń Google w przetwarzaniu masowych obciążeń na dziesiątkach tysięcy komputerów (projekty Borg, Omega). Od 2014 roku Kubernetes został oddany społeczności na licencji open source. Od tego czasu liczba deweloperów znacznie się powiększyła.

Czym jest Pod ?
Najmniejsza jednostka w kubernetes. To ona odpowiedzialna jest za deklarowanie kontenerów odpalanych na tym samym host/node. Ogólnie kiedy ktoś mówi o Pod to ma namyśli przynajmniej jeden działający kontener

Czym jest Node ??

Maszyną fizyczną lub wirtualną na której będą instalowane Pody. do utworzenia klastra potrzebne nam są co najmniej 3 maszyny 2 node i jeden master który będzie odpowiedzialny za zarzadzanie pozostałymi nodami  

Jak działa sieć w Kubernetesie ?

Komponent odpowiedzialny za sieć w k8s nazywa się service  
odpowiedzialny  jest on za komunikację miedzy podami jak i równiez za wystawienie kontenerów na świat. Komunikacja pomiędzy kontenerami w Podzie odbywa się w jadrze linuxa bez udziału sieci tzn  jeśli w jednym podzie mamy 2 lub wieicej kontenerów wszystkie kontenery bedą miały wspolny adres ip . Service może wystepować w 3 trybach ClusterIP, LoadBalancer,NodePort.
Serwis posiada swój własny adres IP oraz nazwę DNS, a Kubernetes zapewnia mechanizmy równoważenia obciążenia.
Najlepszą metoda do utworzenia wysoko dostępnej usługi jest Service w trybie LoadBalancer.
Najproszcza metoda utworzenia loadbalansera w kubernetesie to wykorzystanie NodePort. Dzieki metodzie NodePort mozemy wystawić poda na zewnatrz jednak ta metoda nie jest najlepszym loadbalanserm ponieważ zajmuje jeden port na wszystkich wezłach i taka aplikacja bedzie miała przypisany wysoki port , problemem jest rowniez to ze nie mozemy uzyc nazwy dns tylko musi to byc adres ip.
