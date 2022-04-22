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

```bash
  vars:
    username: [wstaw tu swojego usera bez nawiasow]
    userpass: [wstaw tu swojeg haslo bez nawiasow]
```

Dla przykładu:

```bash
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


















These are my templates and configurations I use in various projects and deployment scenarios. They are based on automation and deployment tools such as [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu), [Docker](https://docs.docker.com/engine/install/ubuntu/), and much more.

I created them as free resources to be extended by you according to your specific use cases. 

### Warning

⚠️ Please beware that products can change over time. I do my best to keep up with the latest changes and releases, but please understand that this won’t always be the case.

### Contribution and Support

🤝 If you’d like to contribute to this project, create a pull request for the necessary changes. 



