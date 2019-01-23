# ProjektP1
Czym jest Kubernetes ?
Kubernetes to oprogramowanie do automatyzacji instalacji, skalowania i zarządzania aplikacji skonteneryzowanych. Dla ułatwienia zarządzania aplikacjami Kubernetes grupuje kontenery, na których pracują instancje komponentów aplikacji w jednostki logiczne zwane Podami.

Kubernetes bazuje na piętnastu latach doświadczeń Google w przetwarzaniu masowych obciążeń na dziesiątkach tysięcy komputerów (projekty Borg, Omega). Od 2014 roku Kubernetes został oddany społeczności na licencji open source. Od tego czasu liczba deweloperów znacznie się powiększyła.

Czym jest Pod ?
Najmniejsza jednostka w kubernetes. To ona odpowiedzialna jest za deklarowanie kontenerów odpalanych na tym samym host/node. Ogólnie kiedy ktoś mówi o Pod to ma namyśli przynajmniej jeden działający kontener

Czym jest Node ??

Maszyną fizyczną lub wirtualną na której będą instalowane Pody. do utorzenia klastra potrzebne nam są conajmniej 3 maszyny 2 node i jeden master który bedzie odpowiedzialny za kontrolowanie pozostałymi nodami  

Jak działa sieć w Kubernetesie ?

Komponent odpowiedzialny za sieć w k8s nazywa się service  
odpowiedzialny  jest on za kominikacje miedzy podami jak i rowniez za wystawienie kontenerow na świat.Komunikacja pomiedzy kontenerami w Podzie odbywa się w jadrze linuxa bez udziału sieci tzn  jeśli w jednym podzie mamy 2 lub wieicej kontenerów wszystkie kontenery bedą miały wspolny adres ip .Service może wystepować w 3 stanach ClusterIP, LoadBalancer,NodePort.
Serwis posiada swój własny adres IP oraz nazwę DNS, a Kubernetes zapewnia mechanizmy równoważenia obciążenia.

Jak działa loadbalanser w k8s
