# 🚀 Бесплатный 24/7 сервер в Google Cloud ($0/мес)

[![Google Cloud](https://img.shields.io/badge/Google_Cloud-4285F4?style=for-the-badge&logo=google-cloud&logoColor=white)](https://cloud.google.com/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![Always Free](https://img.shields.io/badge/Always_Free-$0.00-success?style=for-the-badge)](https://cloud.google.com/free/docs/free-cloud-features#compute)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)

> 🤖 **ВАЖНО: Проще всего делать с AI-агентом!**  
> Самый быстрый способ развернуть этот сервер — **не читать весь огромный мануал вручную, а передать файл [`JOURNEY_AND_ARCHITECTURE_GUIDE.md`](./JOURNEY_AND_ARCHITECTURE_GUIDE.md) (или [`SKILL.md`](./SKILL.md)) вашему AI-агенту** (Google Antigravity, Cursor, Windsurf, Claude Dev).  
> Агент шаг за шагом проведет вас по процессу, перепроверит параметры в консоли GCP во избежание платных опций и сам выполнит команды настройки Linux. Это намного проще, быстрее и безопаснее, чем делать всё вслепую!

---

## 📌 О проекте

Пошаговое практическое руководство по запуску круглосуточного Linux-сервера в **Google Cloud Compute Engine** по программе **Always Free Tier**. 

Идеально для запуска AI CLI-агентов, телеграм-ботов, фоновых скриптов и удаленного управления через веб-дашборд.

* **Философия бюджета:** $0.00 личных средств. Расходы на 100% покрываются программой Always Free и ежемесячными купонами $10 от подписки Google AI Pro.  
* *Подписку Google AI Pro (за ~1$) можно приобрести через Telegram-бота: [@ShopAethelBot](https://t.me/ShopAethelBot?start=ref_8251436466).*

---

## 📊 Сравнительная матрица вариантов развертывания (Глава 9)

| Параметр | 🥉 Вариант 1: «Абсолютный ноль» | 🥇 Вариант 2: «Баланс и Скорость» (**Наш выбор!**) | 🥈 Вариант 3: «Spot-эксперимент» |
| :--- | :--- | :--- | :--- |
| **Режим машины (Model)** | **Standard** (Always Free Tier) | **Standard** (Always Free Tier) | **Spot VM** (Прерываемая) |
| **Тип инстанса** | `e2-micro` (2 vCPU, 1 GB RAM) | `e2-micro` (2 vCPU, 1 GB RAM) | `e2-micro` или `e2-small` |
| **Регион размещения** | `us-central1` (Iowa) | `us-central1` (Iowa) | `us-central1` (Iowa) |
| **Плата за CPU + RAM** | **$0.00** (скидка Always Free) | **$0.00** (скидка Always Free) | ~$2.11 – $7.34 / мес |
| **Тип и объем диска** | 30 GB HDD (`pd-standard`) | **30 GB Balanced SSD (`pd-balanced`)** | 30 GB Balanced SSD (`pd-balanced`) |
| **Плата за диск** | **$0.00** (входит в Always Free) | **$3.00 / месяц** | $3.00 / месяц |
| **Плата за IPv4-адрес** | ~$3.65 / месяц | ~$3.65 / месяц | ~$3.65 / месяц |
| **ИТОГО расходы в месяц** | **~$3.65 / месяц** | **~$6.65 / месяц** | **~$8.76 – $10.34 / месяц** |
| **Покрытие купоном $10** | Полное с запасом | **Полное с запасом** | Впритык или дефицит (-$0.34) |
| **Чистая сдача в копилку** | **+$6.35 / месяц 💰** | **+$3.35 / месяц 💵** | +$1.24 или -$0.34 ❌ |
| **Стабильность 24/7** | **100% (Никогда не выключается)** | **100% (Никогда не выключается)** | ⚠️ Риск выключения в любой момент |
| **Скорость Swap (IOPS)** | ~45 IOPS (глухие зависания) | **до 3,000 IOPS burst (Мгновенно)** | до 3,000 IOPS burst (Мгновенно) |
| **Итоговый вердикт** | Годится только для фонового cron | **Идеальный выбор для AI-агентов** | Неоправданный риск ради копеек |

---

## ⚡ Краткий алгоритм запуска

1. **Создать проект GCP:** Привязать Billing Account (пробные $300 или купон $10 от Google AI Pro).
2. **Создать ВМ `e2-micro`:**
   - Регион: `us-central1`, `us-west1` или `us-east1`.
   - Режим: **Standard** (НЕ Spot!).
   - Диск: **30 GB Balanced Persistent Disk (SSD)**, Ubuntu 24.04 LTS.
   - Отключить `Ops Agent` во вкладке Observability (экономия 200 МБ RAM).
   - Отключить автоснапшоты.
3. **Расширить память до 5 ГБ (Swap 4 ГБ на SSD):**
   ```bash
   sudo fallocate -l 4G /swapfile && sudo chmod 600 /swapfile && sudo mkswap /swapfile && sudo swapon /swapfile
   echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
   sudo tee /etc/sysctl.d/99-swap.conf << 'EOF'
   vm.swappiness=10
   vm.vfs_cache_pressure=50
   EOF
   sudo sysctl --system
   ```
4. **Установить защиту и веб-панель Cockpit:**
   ```bash
   sudo apt update && sudo apt install -y fail2ban cockpit
   sudo systemctl enable --now fail2ban cockpit.socket
   ```
5. **Безопасное подключение через SSH-туннель:**
   ```bash
   ssh -N -L 9090:localhost:9090 <ваш_сервер>
   ```
   Откройте в браузере: `https://localhost:9090`.

---

## 📖 Полная летопись проекта

Детальный разбор всех 12 глав, математика калькулятора GCP, решение ошибок и советы архитектора:

👉 **[JOURNEY_AND_ARCHITECTURE_GUIDE.md](./JOURNEY_AND_ARCHITECTURE_GUIDE.md)**

---

## 📄 Лицензия

MIT License © [Eniggman](https://github.com/Eniggman)
