---
name: gcp-free-server
description: "Guide a user step by step through creating a 24/7 Linux server on Google Cloud Compute Engine with $0 out of pocket: an Always Free e2-micro (Standard, not Spot) in us-central1/us-west1/us-east1 with Ubuntu 24.04, where the remaining ~$3.65–6.65/month (external IPv4 + optional 30 GB Balanced SSD) is covered by the $10/month Google Cloud credit from a Google AI Pro subscription; then extend RAM to ~5 GB with a 4 GB SSD swap, install fail2ban and the Cockpit web panel over an SSH tunnel. Use when the user wants a free/cheap always-on cloud server for AI agents, Telegram bots or scripts. Triggers: «бесплатный сервер Google Cloud», «GCP Always Free», «e2-micro», «сервер за $0», «поднять сервер в гугле», «сервер для ИИ-агентов 24/7», «купон Google AI Pro на GCP», «swap на сервере», «Cockpit»."
---

# Сервер 24/7 в Google Cloud за $0 из своего кармана

Инструкция для ИИ-агента. Для людей: [README](https://github.com/Eniggman/gcp-free-server#readme). Полная летопись в 12 главах: [`JOURNEY_AND_ARCHITECTURE_GUIDE.md`](https://github.com/Eniggman/gcp-free-server/blob/main/JOURNEY_AND_ARCHITECTURE_GUIDE.md). Отправляй к нужной главе, если человек хочет подробностей.

## Как устроена экономика (говори человеку именно так)

Сервер **не полностью бесплатный**. Он стоит **$0 из своего кармана**, потому что расходы покрывает кредит:
- Процессор и RAM `e2-micro` (Standard): **$0** по программе Always Free.
- Внешний IPv4 (Ephemeral): **~$3.65/мес** ($0.005/ч). Этот платёж обязателен.
- Диск: 30 GB `pd-standard` (HDD) — $0 по Always Free, **или** 30 GB `pd-balanced` (SSD, рекомендован гайдом из-за быстрого swap) — **$3.00/мес**.
- Итого **~$3.65/мес (HDD) или ~$6.65/мес (SSD)**. Это покрывает ежемесячный промо-кредит **$10** в Google Cloud от подписки **Google AI Pro**. Неистраченный остаток копится до срока годности купонов (по гайду 12–18 месяцев).
- Без подписки AI Pro (или без активированного купона) эти суммы списываются с привязанной карты. **Обязательно скажи это человеку.**

## Точки подтверждения (без явного «да» человека не делать)

1. Создание или выбор проекта и **привязка платёжного аккаунта** (billing) — только с согласия человека, данные карты вводит он сам.
2. Активация купона AI Pro в Billing → Credits (делает человек).
3. **Создание ВМ** — сначала покажи итоговый чек-лист параметров (ниже) и стоимость, потом «Create».
4. Любой платный выбор: SSD вместо HDD, Static IP, снапшоты, машина крупнее `e2-micro`.
5. Изменение системных файлов на сервере (`/etc/fstab`, sysctl) и установка пакетов.

Не вводи данные карты и пароли за человека, не проси прислать их в чат.

## Шаг 1. Проект и биллинг (консоль GCP, делает человек)

- Отдельный проект (например, `AI server`), Organization: `No organization`.
- Привязать Billing Account. Если на старом аккаунте висит долг, гайд советует написать в Billing Support (глава 2.2), а не заводить аккаунты-дубликаты.
- Проверить в Billing → Credits, что купон AI Pro на $10 активен.

## Шаг 2. Создание ВМ: чек-лист перед «Create» (глава 12)

- Region/Zone: `us-central1` (например, `us-central1-b`), `us-west1` или `us-east1`. В других регионах Always Free не действует.
- Machine: серия E2, **`e2-micro`** (2 vCPU shared-core, 1 GB).
- Provisioning model: **Standard** (НЕ Spot: Spot исключён из Always Free и может выключиться в любой момент).
- On VM termination: **Stop** (не Delete).
- Boot disk: **Ubuntu 24.04 LTS**, 30 GB, `pd-balanced` ($3/мес) или `pd-standard` ($0) — по выбору человека.
- Snapshot schedule: **None** (снапшоты платные, вне бесплатных квот).
- Observability: **снять галочку Ops Agent** (он съедает ~200 МБ из 1 ГБ RAM).
- Availability: On host maintenance — Migrate; Host error recovery — Default (Restart VM).
- Firewall: `Allow HTTP traffic` и `Allow HTTPS traffic` (SSH 22 и так открыт правилом `default-allow-ssh`).
- External IPv4: **Ephemeral** (у Static при остановленной ВМ тариф ~$7.30/мес за простой).

Гайд не называет лимит бесплатного исходящего трафика. Предупреди, что egress сверх бесплатной квоты Google тарифицируется отдельно, а актуальный лимит человек смотрит на странице Free Tier.

## Шаг 3. Подключение

```bash
gcloud auth login                                    # делает человек в браузере
gcloud config set project <PROJECT_ID>
gcloud compute ssh <имя_ВМ> --zone=<зона>            # например: gcloud compute ssh ai-server --zone=us-central1-b
```
`gcloud` сам находит текущий адрес ВМ, поэтому смена Ephemeral IP после перезапуска не мешает. Если `gcloud` не установлен, человек может открыть SSH из консоли (кнопка SSH у ВМ).

## Шаг 4. Swap 4 ГБ + тюнинг ядра (rerun-safe скрипт из главы 8.2)

Выполнять на сервере после подтверждения человека. Скрипт безопасен для повторного запуска.
```bash
#!/usr/bin/env bash
set -euo pipefail
SWAP_PATH="/swapfile"
SWAP_SIZE_GB=4
if [ -f "$SWAP_PATH" ]; then
    if swapon --show | grep -q "$SWAP_PATH"; then
        echo "Swapfile уже активен."
    else
        sudo chmod 600 "$SWAP_PATH"
        sudo swapon "$SWAP_PATH"
    fi
else
    if ! sudo fallocate -l "${SWAP_SIZE_GB}G" "$SWAP_PATH" 2>/dev/null; then
        sudo dd if=/dev/zero of="$SWAP_PATH" bs=1M count=$((SWAP_SIZE_GB * 1024)) status=progress
    fi
    sudo chmod 600 "$SWAP_PATH"
    sudo mkswap "$SWAP_PATH"
    sudo swapon "$SWAP_PATH"
fi
if ! grep -qF "$SWAP_PATH" /etc/fstab; then
    echo "$SWAP_PATH none swap sw 0 0" | sudo tee -a /etc/fstab
fi
sudo tee /etc/sysctl.d/99-swap-tuning.conf > /dev/null << 'CONF'
vm.swappiness=10
vm.vfs_cache_pressure=50
CONF
sudo sysctl --system > /dev/null 2>&1 || sudo sysctl -p /etc/sysctl.d/99-swap-tuning.conf
sudo swapon --show
free -h
```

## Шаг 5. Защита и веб-панель

```bash
sudo apt update && sudo apt install -y fail2ban cockpit
sudo systemctl enable --now fail2ban cockpit.socket
```
- Порт Cockpit 9090 **не открывать в интернет**. Доступ только через SSH-туннель с ПК человека:
  ```bash
  ssh -N -L 9090:localhost:9090 <ssh_алиас_сервера>
  # или через gcloud:
  gcloud compute ssh <имя_ВМ> --zone=<зона> -- -N -L 9090:localhost:9090
  ```
  Затем открыть `https://localhost:9090`.
- Для входа в Cockpit нужен пользователь Linux с паролем и sudo (в гайде это отдельный пользователь `aiserver`, глава 11.1.2). Пароль задаёт сам человек, в чат его не пересылать.

## Шаг 6. Проверка

```bash
nproc; free -h                         # 2 CPU; Mem ~1 GB, Swap ~4 GB
swapon --show                          # /swapfile, размер 4G
grep swapfile /etc/fstab               # ровно одна строка
sysctl vm.swappiness vm.vfs_cache_pressure   # 10 и 50
systemctl is-active fail2ban cockpit.socket  # active / active
```
В консоли: тип ВМ `e2-micro`, модель Standard, регион из списка Always Free. Через 1–2 дня попроси человека открыть Billing → Reports и убедиться, что расходы списываются с кредита, а не с карты.

## Частые ошибки

| Симптом | Причина и решение |
|---|---|
| В счёте списания за CPU/RAM | Не тот регион, Spot вместо Standard или машина крупнее `e2-micro`. Остановить ВМ и пересоздать или изменить её по чек-листу |
| Списания с карты | Купон AI Pro не активирован или истёк: проверить Billing → Credits |
| Счета за снапшоты | Snapshot schedule не выставлен в None: отключить расписание и удалить снапшоты (с согласия человека) |
| ВМ тормозит, OOM Killer убивает процессы | Нет swap или работает Ops Agent: шаг 4, удалить Ops Agent |
| `fallocate failed` | Скрипт сам переключится на `dd`. Это нормально, просто дольше |
| Дубликаты `/swapfile` в `/etc/fstab` | Раньше запускали «простую» версию команды: удалить лишние строки (после подтверждения), дальше использовать скрипт шага 4 |
| `gcloud compute ssh` не подключается | Неверная зона или проект (`gcloud compute instances list`), ВМ остановлена, нет правила `default-allow-ssh` |
| `https://localhost:9090` не открывается | Туннель не запущен или `cockpit.socket` не активен. Браузер предупредит о самоподписанном сертификате — это ожидаемо |
| Нужно больше мощности | Глава 5.3: Stop → Edit → Machine type → Save → Start. Данные не теряются, но **всё крупнее e2-micro платно** — только с согласия человека |
