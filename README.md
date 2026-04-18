# 🚀 GitLab + Docker + Reverse Proxy (Meraki)

Kompletny system do prowadzenia laboratoriów (GitLab + CI/CD + środowiska studentów) oparty o Dockera i reverse proxy.

---

# 📌 Architektura

* 🌍 **Domena:** edukacjawisniowa.pl
* 🌐 **Public IP (Meraki):** 83.0.94.101
* 🖥️ **Serwer Docker:** 172.31.115.249
* 🔁 **Reverse Proxy:** NGINX (Docker)
* ⚙️ **GitLab:** Docker
* 🤖 **GitLab Runner:** Docker
* 👨‍🎓 **Środowiska studentów:** nginx

---

# 🐳 1. Instalacja Dockera

```bash
sudo zypper refresh
sudo zypper install docker docker-compose
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
```

➡️ Wyloguj się i zaloguj ponownie

---

# 📁 2. Struktura katalogów

```bash
mkdir -p ~/gitlab_docker_v1/nginx/conf.d
mkdir -p /srv/gitlab/{config,logs,data}
mkdir -p /srv/gitlab-runner/config

cd ~/gitlab_docker_v1
```

---

# ⚙️ 3. docker-compose.yml

Utwórz plik:

```bash
nano docker-compose.yml
```

Wklej:

```yaml
version: '3.8'

services:

  nginx:
    image: nginx:latest
    container_name: reverse-proxy
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
    depends_on:
      - gitlab
    networks:
      - gitlab-net

  gitlab:
    image: gitlab/gitlab-ee:latest
    container_name: gitlab
    hostname: edukacjawisniowa.pl
    restart: always

    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://edukacjawisniowa.pl'
        nginx['listen_port'] = 80
        nginx['listen_https'] = false
        gitlab_rails['gitlab_shell_ssh_port'] = 2222

    ports:
      - "8080:80"
      - "2222:22"

    volumes:
      - /srv/gitlab/config:/etc/gitlab
      - /srv/gitlab/logs:/var/log/gitlab
      - /srv/gitlab/data:/var/opt/gitlab

    shm_size: '256m'
    networks:
      - gitlab-net

  runner:
    image: gitlab/gitlab-runner:latest
    container_name: gitlab-runner
    restart: always
    depends_on:
      - gitlab
    volumes:
      - /srv/gitlab-runner/config:/etc/gitlab-runner
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - gitlab-net

  student1:
    image: nginx
    container_name: student1
    restart: always
    networks:
      - gitlab-net

  student2:
    image: nginx
    container_name: student2
    restart: always
    networks:
      - gitlab-net

networks:
  gitlab-net:
```

---

# 🌐 4. Konfiguracja Reverse Proxy (NGINX)

```bash
nano nginx/conf.d/gitlab.conf
```

```nginx
server {
    listen 80;
    server_name edukacjawisniowa.pl;

    location / {
        proxy_pass http://gitlab:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /student1/ {
        proxy_pass http://student1:80/;
    }

    location /student2/ {
        proxy_pass http://student2:80/;
    }
}
```

---

# 🚀 5. Uruchomienie systemu

```bash
docker-compose up -d
```

Sprawdzenie:

```bash
docker ps
docker logs -f gitlab
```

⏳ Pierwsze uruchomienie GitLab: ~10 minut

---

# 🔐 6. Hasło root

```bash
docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

---

# 🌍 7. DNS

Dodaj rekord A:

```
edukacjawisniowa.pl → 83.0.94.101
```

---

# 🔁 8. NAT (Meraki)

| Port publiczny | Wewnętrzny          |
| -------------- | ------------------- |
| 80             | 172.31.115.249:80   |
| 443            | 172.31.115.249:80   |
| 2222           | 172.31.115.249:2222 |

---

# 🔒 9. SSL

## Opcja A (zalecana)

SSL obsługiwany przez Meraki

## Opcja B

Let's Encrypt (opcjonalnie)

---

# 🧪 10. Test

GitLab:

```
https://edukacjawisniowa.pl
```

Studenci:

```
https://edukacjawisniowa.pl/student1
https://edukacjawisniowa.pl/student2
```

---

# 🤖 11. GitLab Runner

Rejestracja:

```bash
docker exec -it gitlab-runner gitlab-runner register
```

Podaj:

```
URL: https://edukacjawisniowa.pl
```

Token:
GitLab → Settings → CI/CD → Runners

---

# 📦 12. .gitlab-ci.yml

```yaml
stages:
  - deploy

deploy:
  stage: deploy
  script:
    - echo "Deploy projektu"
```

---

# 🎓 13. Laboratoria

* LAB1 — Repozytorium
* LAB2 — Git
* LAB3 — CI/CD
* LAB4 — Runner
* LAB5 — Deployment
* LAB6 — praca zespołowa

---

# 🛠️ 14. Diagnostyka

```bash
docker logs gitlab
docker logs reverse-proxy
docker network inspect gitlab-net
```

---

# 💾 15. Backup

```bash
docker exec -t gitlab gitlab-backup create
```

---

# ✅ Gotowe

✔ GitLab działa
✔ Reverse proxy działa
✔ Runner działa
✔ Środowiska studentów gotowe

---

# 🔥 Rozszerzenia

* automatyczne środowiska dla studentów
* system oceniania CI/CD
* panel nauczyciela
* monitoring

---
