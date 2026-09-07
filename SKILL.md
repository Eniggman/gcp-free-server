---
name: gcp-free-server
description: "Получаем бесплатный сервер от Google Cloud ($0/мес): пошаговый архитектурный гайд и экспертный скилл для ИИ-агентов по созданию 24/7 Linux-сервера в Google Cloud Compute Engine (Always Free Tier), расширению RAM до 5 ГБ через SSD Swap и запуску веб-дашборда Cockpit."
---

# Google Cloud Free Server ($0/month Always Free Tier)

Экспертный навык для ИИ-ассистентов по развертыванию, безопасной настройке и круглосуточной поддержке облачного сервера в Google Cloud Platform с нулевым личным бюджетом ($0 из своего кармана).

Полное подробное руководство со всеми 12 главами и летописью архитектурных решений находится в файле [`JOURNEY_AND_ARCHITECTURE_GUIDE.md`](./JOURNEY_AND_ARCHITECTURE_GUIDE.md).

---

## 🧭 Как агенту вести пользователя по шагам

При получении запроса от пользователя («Помоги поднять бесплатный сервер в гугле», «Как настроить GCP Always Free», «Разверни сервер за $0») следуйте этому алгоритму:

1. **AI Agent First подход:**
   - Не отправляйте пользователя читать весь файл вручную.
   - Проводите пользователя по шагам, объясняя смысл настроек и сверяя параметры в консоли GCP перед подтверждением создания.

2. **Железные критерии программы Google Cloud Always Free:**
   - **Machine type:** строго `e2-micro` (2 vCPU, shared-core, 1 GB RAM).
   - **Регионы:** только `us-central1` (Iowa), `us-west1` (Oregon) или `us-east1` (South Carolina).
   - **Provisioning Model:** строго **Standard** (Внимание: **НЕ Spot**! Spot-инстансы исключены из Always Free и будут платными).
   - **Boot Disk:** до 30 GB (Balanced Persistent Disk или Standard Persistent Disk, ОС Ubuntu 24.04 LTS).
   - **Сеть:** Ephemeral IPv4 (динамический публичный IP).

3. **Защита от скрытых списаний GCP:**
   - Отключить `Ops Agent` (Cloud Logging / Monitoring) во вкладке Observability — экономит ~200 МБ ценной оперативной памяти.
   - В политике удаления указать `Keep disk / Stop instance` (вместо удаления диска/ВМ).
   - Отключить автоснапшоты (Snapshots).

4. **Обязательная настройка Linux сразу после первого входа:**
   - **Создание 4 ГБ Swap на SSD:**
     ```bash
     sudo fallocate -l 4G /swapfile
     sudo chmod 600 /swapfile
     sudo mkswap /swapfile
     sudo swapon /swapfile
     echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
     ```
   - **Тюнинг ядра Linux для плавной работы агентов (`/etc/sysctl.d/99-swap.conf`):**
     ```bash
     sudo tee /etc/sysctl.d/99-swap.conf << 'EOF'
     vm.swappiness=10
     vm.vfs_cache_pressure=50
     EOF
     sudo sysctl --system
     ```
   - **Защита от брутфорса:**
     ```bash
     sudo apt update && sudo apt install -y fail2ban
     sudo systemctl enable --now fail2ban
     ```
   - **Веб-панель Cockpit (порт 9090):**
     ```bash
     sudo apt install -y cockpit
     sudo systemctl enable --now cockpit.socket
     ```
     Подключение с локального ПК через безопасный SSH-туннель:
     ```bash
     ssh -N -L 9090:localhost:9090 <ваш_ssh_алиас>
     ```
     Открыть в браузере: `https://localhost:9090`.

---

## 📚 Справочник глав руководства

Если пользователю требуется глубокое понимание конкретного вопроса, ссылайтесь на соответствующие главы [`JOURNEY_AND_ARCHITECTURE_GUIDE.md`](./JOURNEY_AND_ARCHITECTURE_GUIDE.md):
- **Глава 1:** Выбор стека взаимодействия (Opera MCP vs Vision vs Computer Use).
- **Глава 2:** Философия проекта и нулевой бюджет ($0 личных средств, списание старых долгов через тикет).
- **Глава 3:** Разбор понятий Compute Instance, vCPU, shared-core и CPU Bursting.
- **Глава 4:** Финансовый детектив и калькуляторная ловушка GCP.
- **Глава 5:** Выбор тарифа e2-micro и смена конфигурации без потери данных.
- **Глава 6:** Скрытые настройки GCP (Ops Agent, снапшоты, защита от удаления).
- **Глава 7:** Программа Always Free Tier (почему процессор бесплатен навсегда).
- **Глава 8:** Магия Swap в Linux (превращение 1 ГБ в 5 ГБ стабильной памяти).
- **Глава 9:** Итоговая матрица решений (Zero vs Balance vs Spot).
- **Глава 10:** Механика бонусов Promotional Credits (FIFO, несгораемая сдача).
- **Глава 11:** Графический интерфейс Cockpit, SSH-туннелирование и скрипты запуска.
- **Глава 12:** Золотой чек-лист создания сервера в GCP.
