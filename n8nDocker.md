# Krok 1: Skonfiguruj Mikrusa

Po zakupie i otrzymaniu odpowiednich dostępów, wystarczy zalogować się na swój serwer VPS.

Tam poleceniem `start` można przeprowadzić wstępną konfigurację.

Na potrzeby n8n, najważniejsze jest, żeby potwierdzić kiedy padnie pytanie o instalację Dockera.

---

## Krok 2: Przygotuj katalog na dane n8n

```bash
mkdir ~/.n8n
chmod 777 ~/.n8n
```

---

## Krok 4: Uruchom n8n po raz pierwszy

Na początek wystarczy polecenie:

```bash
docker run -d --name n8n --rm -p TWÓJ_PORT:5678 -v ~/.n8n:/home/node/.n8n n8nio/n8n:latest
```

Wartość `TWÓJ_PORT` należy zastąpić jednym z Twoich portów otwartych na świat. Może być pierwszy z brzegu. Port znajdziesz na przykład zaglądając do pliku `/etc/motd`.

**Ważne!** W tej konfiguracji n8n uruchomi się z bazą SQLite. Na razie nam to nie przeszkadza, ale jeżeli od razu chcesz wystartować z PostgreSQL, to będzie potrzeba kilka dodatkowych parametrów.

Teraz polecenie `docker ps` powinno pokazać uruchomiony kontener n8n, a pod adresem `http://srvNN.mikr.us:TWÓJ_PORT` (NN musisz zamienić na to co otrzymasz od Mikrusa) Twoim oczom ukaże się coś podobnego:

Wszystko pod kontrolą 🙂

---

## Krok 5: Skonfiguruj subdomenę na Mikrusie

W panelu zarządzania Mikrusem znajdziesz zakładkę Subdomeny, a w niej prosty formularz: wprowadzasz wymyśloną przez siebie subdomenę, wybierasz jedną z dostępnych domen i podajesz `TWÓJ_PORT` tam gdzie pyta o port.

Powiedzmy, że wynikowo otrzymujesz `twoja_subdomena.domena_mikr.us`. Po chwili, pod tym adresem powinno znaleźć się coś podobnego:

Dobra nasza! Mamy n8n na Mikrusie 🙂

---

## Krok 6: Poproś o dostęp do współdzielonego Postgresa na Mikrusie

Będąc nadal w panelu zarządzania Mikrusem, w zakładce PostgreSQL (w sekcji Bazy danych) znajduje się przycisk: **Poproszę o nowe dane dostępowe**.

Kliknij na niego, a otrzymasz coś takiego:

```
Server: serwer_pgsql.mikr.us
login: twoj_login
Haslo: twoje_haslo
Baza: twoja_baza_danych
```

---

## Krok 7: Uruchom n8n w pełni produkcyjnie

Teraz możemy zebrać wszystko do kupy i przygotować docelową konfigurację.

Najpierw tworzymy plik `docker-compose.yml` (w dowolnym katalogu), pamiętając o tym, żeby zastąpić co trzeba swoimi danymi:

```yaml
version: "3.7"

services:
  n8n:
    image: n8nio/n8n
    container_name: n8n
    restart: always
    ports:
      - "TWÓJ_PORT:5678"
    environment:
      - TZ=Europe/Warsaw
      - N8N_HOST=twoja_subdomena.domena_mikr.us
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - NODE_ENV=production
      - WEBHOOK_URL=https://twoja_subdomena.domena_mikr.us/
      - GENERIC_TIMEZONE=Europe/Warsaw
      - N8N_RUNNERS_ENABLED=true
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=twoja_subdomena.domena_mikr.us
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=twoja_baza_danych
      - DB_POSTGRESDB_USER=twoj_login
      - DB_POSTGRESDB_PASSWORD=twoje_haslo
    volumes:
      - ~/.n8n:/home/node/.n8n
```

Następnie wyłączamy działający kontener:

```bash
docker stop n8n
```

I uruchamiamy przed chwilą skonfigurowany. Będąc w katalogu z plikiem `docker-compose.yml` uruchamiamy polecenie:

```bash
docker compose up -