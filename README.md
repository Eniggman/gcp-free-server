# 🚀 Бесплатный сервер от Google Cloud: 24/7 AI-сервер за $0/месяц

[![Google Cloud](https://img.shields.io/badge/Google_Cloud-4285F4?style=for-the-badge&logo=google-cloud&logoColor=white)](https://cloud.google.com/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![Always Free](https://img.shields.io/badge/Always_Free-$0.00-success?style=for-the-badge)](https://cloud.google.com/free/docs/free-cloud-features#compute)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)

> 🤖 **ВАЖНО: AI Agent First!**  
> Самый простой и эффективный способ настроить сервер — **не читать весь огромный документ вручную, а передать Markdown-файл [`JOURNEY_AND_ARCHITECTURE_GUIDE.md`](./JOURNEY_AND_ARCHITECTURE_GUIDE.md) или [`SKILL.md`](./SKILL.md) вашему AI-агенту** (Google Antigravity, Cursor, Windsurf, Claude Dev).  
> Агент шаг за шагом проведет вас через весь процесс в реальном времени, перепроверит каждый чекбокс в консоли GCP, чтобы исключить скрытые платные опции, и выполнит команды настройки Linux. Это в разы быстрее, удобнее и надежнее, чем просто читать текст!

---

## 📌 О проекте

Практическое руководство и летопись реальных инженерных решений по запуску вечного сервера в **Google Cloud Platform (GCP)** в рамках официальной программы **Always Free Tier**.

Сервер идеально оптимизирован для автономной круглосуточной работы AI CLI-агентов, фоновых ботов, микросервисов и удаленного управления через веб-интерфейс.

### 💎 Что вы получаете за $0.00 / месяц:
* **Вычислительная мощность:** 2 vCPU (shared-core `e2-micro` с мгновенным аппаратным ускорением CPU Bursting до 200%).
* **Оперативная память:** **5.0 ГБ рабочей памяти** (1 ГБ быстрой физической RAM + 4 ГБ скоростного Swap на SSD).
* **Накопитель:** 30 GB Balanced Persistent Disk (SSD) с производительностью до 3,000 IOPS.
* **Мониторинг и UI:** Графическая панель управления **Cockpit** (порт 9090) через безопасный зашифрованный SSH-туннель.
* **Защита:** Защита от брутфорса `fail2ban` и строгие правила изоляции портов.
* **100% покрытие расходов:** Процессор и машина бесплатны навсегда в рамках лимита 744 часа/мес (Always Free), а минимальные сопутствующие копейки за диск/IP полностью покрываются ежемесячными ваучерами или стартовым бонусом.

---

## ⚡ Краткий гайд по запуску (Quickstart)

### Шаг 1. Подготовка аккаунта Google Cloud
1. Откройте [Google Cloud Console](https://console.cloud.google.com/).
2. Создайте новый изолированный проект (например, `ai-server`) без привязки к организациям (`No organization`).
3. Привяжите Billing Account (при первой регистрации начисляется $300 пробных средств, либо используются ежемесячные ваучеры $10 от подписки Google AI Pro).

### Шаг 2. Создание инстанса ВМ (Always Free Tier)
Перейдите в **Compute Engine ➡️ VM instances ➡️ Create instance**:
- **Name:** `ai-server`
- **Region:** `us-central1` (Айова), `us-west1` (Орегон) или `us-east1` (Южная Каролина) — *только эти регионы входят в Always Free*.
- **Machine configuration:** Серия **E2**, тип машины **`e2-micro`** (2 vCPU, 1 GB memory).
- **Provisioning model:** строго **Standard** *(Внимание: НЕ Spot! Режим Spot не входит в Always Free)*.
- **Boot disk:** Нажмите **Change**:
  - Операционная система: **Ubuntu**
  - Версия: **Ubuntu 24.04 LTS (x86_64)**
  - Размер диска: **30 GB**
  - Тип диска: **Balanced persistent disk** (SSD)
- **Скрытые ловушки (обязательно отключить):**
  - Во вкладке **Observability** отключите галочку **Install Ops Agent** (сбережет ~200 МБ RAM).
  - В настройках управления инстансом установите политику удаления на **Stop** вместо **Delete**.
  - Отключите автоматические платные резервные копии (снапшоты).
- Нажмите кнопку **Create** и дождитесь запуска зеленого индикатора.

### Шаг 3. Быстрый доступ по SSH
Добавьте ваш публичный ключ в метаданные инстанса и пропишите алиас в `~/.ssh/config` на вашем локальном компьютере:
```ssh-config
Host ai
    HostName <ВНЕШНИЙ_IP_СЕРВЕРА>
    User aiserver
    IdentityFile ~/.ssh/id_rsa
```
Подключение выполняется одной короткой командой:
```bash
ssh ai
```

### Шаг 4. Секретное оружие: Расширение памяти до 5 ГБ (SSD Swap)
По умолчанию 1 ГБ RAM недостаточно для тяжелых задач. Создаем скоростной файл подкачки на 4 ГБ:
```bash
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```
Тюнинг ядра Linux для бережного сброса фоновых процессов в Swap:
```bash
sudo tee /etc/sysctl.d/99-swap.conf << 'EOF'
vm.swappiness=10
vm.vfs_cache_pressure=50
EOF
sudo sysctl --system
```

### Шаг 5. Защита от брутфорса
Установите службу `fail2ban`, которая автоматически заблокирует атакующих ботов после 4 неудачных попыток входа:
```bash
sudo apt update && sudo apt install -y fail2ban
sudo systemctl enable --now fail2ban
```

### Шаг 6. Графическая панель Cockpit
Установите веб-интерфейс Cockpit:
```bash
sudo apt install -y cockpit
sudo systemctl enable --now cockpit.socket
```
Для безопасного входа без открытия 9090 порта в глобальный интернет пробросьте туннель:
```bash
ssh -N -L 9090:localhost:9090 ai
```
Откройте в вашем браузере: `https://localhost:9090`.

---

## 📖 Полная летопись инженерных решений

Для глубокого погружения во все тонкости архитектуры, расчёты биллинга, работу с калькулятором GCP и сравнение протоколов взаимодействия читайте полную книгу проекта:

👉 **[JOURNEY_AND_ARCHITECTURE_GUIDE.md](./JOURNEY_AND_ARCHITECTURE_GUIDE.md)**

### Краткое оглавление глав:
1. **Выбор рабочего стека:** Opera MCP vs Vision vs Computer Use
2. **Пролог:** Философия проекта и нулевой бюджет ($0 из своего кармана)
3. **Обучение с нуля:** Compute Instance, vCPU, shared-core и CPU Bursting
4. **Финансовый детектив:** Калькуляторная ловушка GCP
5. **Интуиция автора:** Экономия на e2-micro и смена тарифа за 30 секунд
6. **Препарирование настроек:** Ops Agent, снапшоты и сетевой файрвол
7. **Главная эврика:** Google Cloud Always Free Tier навсегда
8. **Секретное оружие:** Магия Swap в Linux (из 1 ГБ делаем 5 ГБ)
9. **Матрица решений:** Zero vs Balance vs Spot
10. **Развенчиваем миф:** Накопление бонусов Promotional Credits по FIFO
11. **Финальная реализация:** Cockpit UI и автотаймер
12. **Золотой чек-лист создания сервера в GCP**

---

## 📄 Лицензия

Распространяется под лицензией [MIT](./LICENSE).
Автор: [@Eniggman](https://github.com/Eniggman)
