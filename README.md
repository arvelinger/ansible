# Playbooki do automatyzacji podstawowych zadań (Work In Progress)

<p align="center">
 <img src="logo.png?raw=true" alt="Ansible" width="50%" height="50%" />
</p>

## 🤔 Czym jest Ansible?

Ansible jest narzędziem służącym do zarządzania stanem infrastruktury. Pozwala zautomatyzować administrację konfiguracją serwerów, dostarczanie aplikacji oraz wiele innych zadań stawianych IT. Jest jednym z popularniejszych przedstawicieli narzędzi, realizujących założenia paradygmatu IaC (Infrastructure as Code), który pozwala uzyskać powtarzalne konfiguracje maszyn. Inne podobne narzędzia to m.in. Chef czy Puppet, jednak to Ansible jest według mnie najprostszy, aby zapoznać się z tego typu narzędziami. Ansible działa na zasadzie bezagentowej, tzn. nie potrzebuje instalacji żadnego dodatkowego oprogramowania na zarządzanej maszynie. Narzędzie do łączenia z zarządzanymi serwerami używa protokołu SSH. Praca z Ansible pozwala zarówno na wykonywanie komend ad-hoc, jak i na przetwarzanie wcześniej zdefiniowanych w języku YAML plików konfiguracyjnych, tzw. playbooków.

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

```bash
ssh-keygen -t rsa -b 4096
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











These are my templates and configurations I use in various projects and deployment scenarios. They are based on automation and deployment tools such as [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu), [Docker](https://docs.docker.com/engine/install/ubuntu/), and much more.

I created them as free resources to be extended by you according to your specific use cases. 

### Warning

⚠️ Please beware that products can change over time. I do my best to keep up with the latest changes and releases, but please understand that this won’t always be the case.

### Contribution and Support

🤝 If you’d like to contribute to this project, create a pull request for the necessary changes. 



