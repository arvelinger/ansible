# Ansible jako skuteczne narzędzie w codziennej pracy

<p align="center">
 <img src="logo.png?raw=true" alt="Ansible" width="50%" height="50%" />
</p>

## Spis treści
* [Czym jest Ansible](#-czym-jest-ansible)
* [Wymagania do działania](#-wymagania-do-działania)
* [Startujemy](#-startujemy)
* [Przygotowanie](#-przygotowanie)
* [Ansible](#-ansible)
* [Instalacja Ansible](#instalacja-ansible)
* [Przygotowania do lenis.. automatyzacji!](#przygotowania-do-lenis-automatyzacji)
* [Dodajemy hosty](#dodajemy-hosty)
* [Testujemy!](#-testujemy)
* [ssh.yaml](#1%EF%B8%8F⃣-sshyaml)
* [newuser.yaml](#2%EF%B8%8F⃣-newuseryaml)
* [apt.yaml](#3%EF%B8%8F⃣-aptyaml)
* [newuser.yaml](#4%EF%B8%8F⃣-fail2banyaml)
* [fail2ban.yaml](#4%EF%B8%8F⃣-fail2banyaml)
* [Ważne](#%EF%B8%8F-ważne)
* [Wkład i wsparcie](#-wkład-i-wsparcie)

## 🤔 Czym jest Ansible?

Ansible jest narzędziem służącym do zarządzania stanem infrastruktury. Pozwala zautomatyzować administrację konfiguracją serwerów, dostarczanie aplikacji oraz wiele innych zadań stawianych IT. Jest jednym z najpopularniejszych przedstawicieli narzędzi do realizacji założenia paradygmatu IaC (Infrastructure as Code), który pozwala uzyskać powtarzalne konfiguracje maszyn. Inne podobne narzędzia to m.in. Chef czy Puppet, jednak to Ansible jest według mnie najprostszy, aby zapoznać się z tego typu narzędziami. Ansible działa na zasadzie bezagentowej, tzn. nie potrzebuje instalacji żadnego dodatkowego oprogramowania na zarządzanej maszynie. Narzędzie do łączenia z zarządzanymi serwerami używa protokołu SSH. Praca z Ansible pozwala zarówno na wykonywanie komend ad-hoc, jak i na przetwarzanie wcześniej zdefiniowanych w języku YAML plików konfiguracyjnych, tzw. playbooków.

(źródło: [nofluffjobs](https://nofluffjobs.com/blog/ansible-wprowadzenie-wdrozenie-i-scenariusze-uzytkowania/#Narzedzie_Ansible))

## 📖 Wymagania do działania

Aby playbook poprawnie wykonał swoje założone taski potrzebujesz maszyny z linuxem albo WSL. 
- [x] Linux - ja preferuję ubuntu/debiana
- [x] Wygenerowane klucze SSH
- [x] Wykonane chociażby jedno połączenie po SSH pomiędzy maszynami
- [x] Zainstalowany Ansible na maszynie 

## 🚀 Startujemy

### 🍴 Przygotowanie

Zacznij od wygenerowania kluczy SSH.
W konsoli wpisujemy:

```console
$ ssh-keygen -t rsa -b 4096
``` 

Gdzie:
* `-t` oznacza `type` polecenie to generuje parę kluczy RSA. RSA są domyślnym rodzajem kluczy.
* `-b` oznacza `bites' domyślny klucz ma 3072 bity, my dla większego bezpieczeństwa użyjemy klucza o ilości 4096 bitów.

Gdy zostaniesz zapytany o miejsce zapisu klucza, możesz śmiało wcisnąć `Enter` lub wybrać inną, bardziej dogodną dla Ciebie lokalizację.
Po pojawieniu się wpisu `Enter passphrase` również możesz wcisnąć `Enter`. Jeśli wolisz podać hasło aby zaszyfrować klucz to nic nie stoi na przeszkodzie.

* Klucz prywatny (czyli Twoja identyfikacja) zostanie zapisany w pliku `.ssh/id_rsa` w Twoim katalogu domowym.
* Klucz publiczny zostanie zapisany w pliku `.ssh/id_rsa.pub`, również dostępnym z Twojego katalogu domowego.

Następnie należy wysłać swój klucz na maszynę docelową. Robimy to w następujący sposób:

```bash
ssh-copy-id username@adres-serwera
```

Potwierdź zapisanie klucza serwera na swojej maszynie a następnie podaj hasło użytkownika na którego się logujesz

Sprawdź czy wszystko poszło gładko:

```bash
ssh username@adres-serwera
```

Jeśli zalogowało Cię bez konieczności podawania hasła to wszystko jest gotowe.

### 🤖 Ansible
***
#### Instalacja Ansible

Moim nawykiem jest odświeżanie repozytorium zanim zacznę dodawać nowe pakiety.

```bash
sudo apt update
```

Następnie dodajemy oprogramowanie które jest niezbędne dla poprawnego działania Ansible:

```bash
sudo apt install software-properties-common
```

Następnie dodajemy repozytorium Ansible które pozwoli nam posiadać pakiet w najnowszej możliwej wersji:

```bash
sudo add-apt-repository --yes --update ppa:ansible/ansible
```

Nie pozostaje nam nic innego jak zainstalować w końcu Ansible:

```bash
sudo apt install ansible -y
```

I gotowe! 💎

#### Przygotowania do lenis.. automatyzacji!

Pobieramy sobie tego gita na swoją maszynę

```bash
git clone https://git.bitsec.tk/arvelinger/ansible.git
```

Powinien pobrać się folder `ansible`, w nim powinny być dwa kolejne:
* `inventory` to w nim będziemy zapisywać adresy urządzeń na których będziemy wykonywać nasze playbooki
* `playbooks` tutaj są przygotowane przeze mnie playbooki.

#### Dodajemy hosty

Zaczniemy od dodania kilku maszyn testowych. Pozwoli to na zmniejszenie impaktu w infrastrukturze gdyby coś poszło nie tak.

Przechodzimy do folderu Ansible:

```bash
cd ansible
```

Otwieramy edytor tekstowy. Moim ulubionym jest nano:

```bash
nano inventory/hosts
```

Przykładowy plik z adresami maszyn powinien wyglądać tak:

```ini
[ubuntu]
192.168.1.100
192.168.1.101
192.168.1.102
```

### 💣 Testujemy!
***
#### 1️⃣ `ssh.yaml`

Zacznijmy od najprostszego playbooka `ssh.yaml`. Używam go aby wysyłać klucze do logowania poprzez SSH na maszynę docelową a także na umożliwienie mi wykonywania poleceń `sudo` bez użycia hasła.

Otwieramy playbook w ulubionym edytorze. W moim wypadku będzie to nano:

```bash
nano playbooks/ssh.yaml
```

Odszukujemy następujące linijki:

```ini
  vars:
    username: [wstaw tu swojego usera bez nawiasow]
    userpass: [wstaw tu swojeg haslo bez nawiasow]
```

Dla przykładu:

```ini
  vars:
    username: arvelinger
    userpass: M0st.Pow3rfull.Pas$w0rd.3veR
```

Zapisujemy i zamykamy plik poprzez kombinację przycisków `CTRL+X`, zatwierdzamy zmiany poprzez `y`, jeszcze tylko `Enter` i możemy zaczynać.

Z maszyny na której mamy zainstalowanego Ansible uruchamiamy playbooka następującą komendą:

```bash
ansible-playbook ./playbooks/ssh.yaml --user arvelinger -k -K ./inventory/hosts
```

* `-k` oznacza, że Ansible zapyta nas o hasło do połączenia się z maszynami w pliku `hosts`
* `-K` oznacza, że Ansible zapyta nas o hasło dla użytkownika `sudo` celem wprowadzenia niezbędnych zmian.

I już! Tak, to wszystko! Właśnie dodałeś klucze `RSA` do logowania się poprzez `SSH` oraz zmieniłeś plik `visudo` aby ułatwić sobie pracę i zwiększyć bezpieczeństwo. 

#### 2️⃣ `newuser.yaml`

Ten playbook wykorzystuję aby dodać swojego usera na świeżo skonfigurowanych serwerach gdzie mam do swojej dyspozycji konto `root` albo konto `ubuntu` (co na przykład ma miejsce w przypadku świeżych instalacji na chmurze Oracle). Ze względu na to, że korzystam z WSL pod Windows 11 klucz mam zapisany na swojej roboczej maszynie. I z tej wirtualnej maszyny kopiuję to dalej. Czasami jest też tak, że system mam skonfigurowany ale potrzebuję dodać nowego użytkownika wraz z hasłem, dodać go do grupy `sudo`, `docker` czy zwyczajnie umożliwić mu `sudo` bez konieczności podawania hasła.

Na początek musimy odnaleźć naszego playbooka i go zmodyfikować. Jeśli pobrałeś to repozytorium do swojego katalogu `HOME` to musimy się dostać najpierw do tego katalogu:

```bash
cd ansible
```

Otwieramy playbook'a w edytorze tekstowym:

```bash
nano playbook/newuser.yaml
```

Szukamy następujących wartości:

```ini
  vars:
    username: [wstaw tu swojego usera bez nawiasow]
    userpass: [wstaw tu swojeg haslo bez nawiasow]
```

I modyfikujemy je zgodnie z tym co jest tam napisane. Dla przykładu:

```ini
  vars:
    username: BestUserEver
    userpass: M0st.Pow3rfull.Pas$w0rd.3veR
```

W przeciwieństwie do playbook'a `ssh.yaml` tutaj podajemy login i hasło nowego użytkownika.
Pozostaje nam jeszcze uruchomienie tego playbook'a. Co też czynimy następującą komendą:

```bash
ansible-playbook ./playbooks/newuser.yaml --user [wstaw tutaj login istniejącego konta] -i ./inventory/hosts
```

Jeśli nie masz wyłączonego żądania hasła po wprowadzeniu komendy `sudo` oraz logowania przy użyciu klucza `RSA` musisz dodać dwa atrybuty `-k` oraz `-K`. Atrybuty te sprawią, że ansible zapyta Cię o Twoje hasła, a następnie wykona komendy wykorzystując to co podałeś.

Dla przykładu:

```bash
ansible-playbook ./playbooks/newuser.yaml --user [wstaw tutaj login istniejącego konta] -k -K -i ./inventory/hosts
```

Zanim ansible rozpocznie wykonywanie playbooka zobaczysz coś takiego:

```bash
SSH password: 
BECOME password[defaults to SSH password]:
```

Dzięki temu możesz wyłaczyć użytkowników których już nie potrzebujesz i zamienić na sobie tylko znane hasła i klucze.
Bezpieczeństwo przede wszystkim!

#### 3️⃣ `apt.yaml`

Przed nami mój ulubiony, a przede wszystkim najczęściej używany playbook w pracy codziennej.
Nie jest nowością ani tajemnicą, że utrzymanie systemu operacyjnego serwera czy własnej maszyny ma krytyczny wpływ na poziom bezpieczeństwa naszej maszyny, naszych danych, a także danych naszych pozostałych użytkowników. W dzisiejszych czasach producenci oprogramowania starają się przeciwdziałać podatnościom w swoich aplikacjach. Grzechem byłoby więc nie skorzystać i utrzymywać nasz system w kiepskim stanie. Tym bardziej, że aktualizacje są przecież darmowe.

Jeśli masz już przygotowany plik `hosts`, logujesz się do maszyn na których chcesz wykonać aktualizację przy użyciu kluczy `RSA` i nie musisz już podawać hasła do `sudo` to wystarczy jedna komenda. Tak! jedna komenda i `ENTER` dla potwierdzenia tego co chcesz zrobić.
Zanim jednak przejdziemy do komendy chciałbym wyjaśnić jakie czynności ten playbook tak naprawdę wykonuje.

Taski jakie kolejno wykonuje ansible na tym playbook'u:
1. Sprawdzanie miejsca na dysku
2. Ilość pozostałego miejsca na dysku
3. Sprawdzanie nieużywanych pakietów
4. Lista pakietów do usunięcia
5. Usuwanie zbędnych pakietów
6. Odświeżenie repozytorium
7. Sprawdzenie pakietów do aktualizacji
8. Lista pakietów do aktualizacji
9. Wyrażenie zgody na eksterminację całej ludzkości (wymagane zatwierdzenie `ENTER` lub przerwanie poprzez kombinację klawiszy `CTRL+C`)
10. Ponowne odświeżenie repozytorium oraz aktualizacja wszystkich pakietów
11. Sprawdzenie pozostałego miejsca na dysku
12. Pozostałe miejsce na dysku (powtórzenie tych czynności ma na celu umożliwienie weryfikacji czy nie potrzebujemy zwiększyć przestrzeni dyskowej na danej maszynie)
13. Sprawdzanie czy był aktualizowany kernel
14. Wykonanie restartu jeśli kernel był aktualizowany (ansible nie dokona restartu maszyn hurtowo, czeka aż każda z maszyn się uruchomi i dopiero przejdzie do restartu następnej maszyny)

Jak widzisz playbook jest nakierowany na świadomą automatyzację.

Nie zwlekajmy i przejdźmy od słów do czynów!

Komenda aby uruchomić tego playbook'a jest bardzo prosta.

```bash
ansible-playbook ./test/playbooks/apt.yaml --user [username] -i ./test/inventory/hosts
```

Oczywiście komenda zadziała jeśli logowałeś się już na ten serwer chociaż raz. Masz dodany klucz `RSA` i włączone logowanie przy jego użyciu. A także wyłączoną konieczność podawania hasła przy `sudo`. Jeśli nie to musisz odpowiednio zmodyfikować komendę:

```bash
ansible-playbook ./test/playbooks/apt.yaml --user [username] -k -K -i ./test/inventory/hosts
```

Nie zapomnij o potwierdzeniu punktu 9!

```bash
TASK [Zniszczenie calej ludzkosci] ******************************************************************************************************************************************************************************
[Zniszczenie calej ludzkosci]
Potwierdz prosze ze na pewno chcesz usunac cala ludzkosc! Sprawdz wczesniejsze dane zanim potwierdzisz enterem! Enter aby kontynuowac masowa zaglade. Ctrl+c aby przerwać usuwanie ludzkosci.:
```

I to tak naprawdę tyle!

A co jeśli na serwerze masz pakiety których nie chcesz aktualizować?

To nic trudnego! Przede wszystkim wykorzystajmy to co daje nam dostawca systemu. Wyłącz/zablokuj pakiet za pomocą `apt-mark` z opcją wstrzymania/oddania.
Przede wszystkim zajrzyjmy na [manpages](http://manpages.ubuntu.com/manpages/trusty/pl/man8/apt-mark.8.html)

A tam widzimy bardzo ważną dla nas informację:

```bash
       hold
           hold jest używane do wstrzymania pakietu, co zabroni automatycznego instalowania,
           aktualizowania lub usuwania pakietu. Polecenie jest nakładką na dpkg --set-selections,
           stan pakietu jest zarządzany przez dpkg(1), a opcja --file nie wpływa na działanie
           tego polecenia.

       unhold
           unhold jest używane do usunięcia stanu wstrzymania pakietu ustawionego poprzednio i
           pozwolenia na wykonywanie wszystkich akcji na tym pakiecie.
```

A zatem użycie polecenia `apt-mark hold` wraz z nazwą pakietu zapewni nam brak aktualizacji, usuwania czy ponownej instalacji tego pakietu. 
Przekonajmy się na własnej skórze.

Na początek oczywiście aktualizacja repozytorium:

```bash
sudo apt update
```

Pojawia nam się przykładowa informacja:

```bash
43 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

Sprawdźmy zatem jakie pakiety będą aktualizowane. A także wybierzmy taki którego nie będziemy chcieli aktualizować:

```bash
sudo apt list --upgradable
```

Pojawiła się lista. Ja z tej listy wybieram pakiet losowo aby tylko pokazać zasadę działania

Zatem podajemy polecenie:

```bash
sudo apt-mark hold xz-utils
```

W odpowiedzi dostajemy informację:

```bash
xz-utils set on hold.
```

Po podaniu komendy `sudo apt upgrade -y` dostaniemy informację o zatrzymanych pakietach.

```bash
Następujące pakiety zostały zatrzymane:
  xz-utils
```

Pakiet ten nie zostanie zaktualizowany w trakcie całego procesu.

Wniosek jest prosty. Możemy zautomatyzować wiele ale to po stronie administratora leży obowiązek odpowiedniego przygotowania systemu. Można jeszcze aktualizować bezwzględnie wszystko ale to się prędzej czy później skończy katastrofą 

#### 4️⃣ `fail2ban.yaml`

Czym jest fail2ban? 🤔

Fail2Ban jest aplikacją, która powinna być zainstalowana w zasadzie na każdej maszynie działającej pod kontrolą Unix / Linux / BSD, wystawioną na świat (zewnętrzne IP). Głównym zadaniem Fail2Ban jest blokada podejrzanych, zakończonych niepowodzeniem prób logowania się do usług świadczonych przez nasz serwer. Podsumowując zabezpiecza przed atakami typu brute force.

Działanie programu Fail2Ban opiera się na analizie logów i wychwytywaniu nieautoryzowanych prób logowania się do różnego rodzaju usług (np. ssh, smtp, pop3, imap...), po czym blokuje adres IP atakującego za pomocą reguł iptables lub dodając wpis do pliku /etc/hosts.deny.

Fail2Ban dostępny jest na wielu dystrybucjach jako gotowy pakiet więc instalacja jest banalnie prosta. No chyba, że masz 30 hostów na których musisz to zainstalować. Wtedy również jest to proste ale czasochłonne.

Zautomatyzujmy to! 😃

Tak naprawdę jeśli wykonane zostały wszystkie wcześniejsze kroki to zostaje tylko podanie komendy dla ansible:

```bash
ansible-playbook ./playbooks/fail2ban.yaml --user [username] -i ./inventory/hosts
```

I już! 

Let's automate the whole world!

### 🤷🏻‍♂️ Ważne
***
To są moje szablony i konfiguracje, których używam w różnych projektach i scenariuszach wdrożeniowych. Opierają się na narzędziach automatyzacji i wdrożeniowych, takich jak: [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu), [Docker](https://docs.docker.com/engine/install/ubuntu/), i wiele innych.

💲 Stworzyłem je jako darmowe repozytorium, które możesz rozwijać i zmieniać zgodnie z Twoimi konkretnymi przypadkami użycia.

⚠️ Należy pamiętać, że produkty mogą się zmieniać w czasie. Robię co w mojej mocy, aby być na bieżąco z najnowszymi zmianami i wydaniami, ale proszę zrozumieć, że nie zawsze tak będzie.

### 👾 Wkład i wsparcie
***
🤝 Jeśli chcesz wnieść swój wkład w ten projekt, utwórz pull request dla niezbędnych zmian.



