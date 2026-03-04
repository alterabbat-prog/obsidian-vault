---
tags:
  - learning
  - project
  - linux
aliases:
---



1.  **Подготовительный этап**
    1.1 шагом проверяем текущую версию ядра и ОС
     ==uname -r==  
     ==lsb_release -a==  
     ==cat /etc/astra_version==
    1.2  определение типа системы
     Важно понять, **сертифицированная ли система (ГОСТ, ФСТЭК)**.  
     Если да — обновление ядра может **нарушить сертификацию**.
2.  **Анализ совместимости**
    2.1 ## Проверить поддержку оборудования
    - RAID/диски
    - Сетевые карты
    - GPU
    - Специфические драйверы (HBA, Mellanox, Intel i40e и т.д.)
     Команда 
     ==lspci -nn==  
     ==lsmod==
    2.2 ## Проверить зависимости
      Некоторые модули и ПО привязаны к версии ядра:
     - DKMS
     - SELinux / Astra Mandatory Control
     - Криптопровайдеры (КриптоПро, ViPNet)

3. **Выбор стратегии обновления**
     - из репозитория Astra
        команды:
        ==apt update==  
        ==apt search linux-image==
        ==apt install linux-image-<версия>==

4. **Резервное копирование и откат**
    4.1. ## Создать snapshot (если есть) 
       LVM snapshot
       VM snapshot (VMware, KVM, Hyper-V)
   4.2. ## Сохранить текущее ядро
     **Не удалять старое ядро!**

5. **Установка нового ядра**
     пример команды: ==apt install linux-image-6.x.x linux-headers-6.x.x==
     Обновить загрузчик: ==update-grub==

6. **Тестирование**
      reboot
      uname -r

7. **Настройка GRUB (FALLback)** Оставить старое ядро по умолчанию или таймер поставить
      команды:
      ==nano /etc/default/grub==  
      ==GRUB_TIMEOUT=5==

8.  **План отката (Rollback)**
    - В GRUB выбрать старое ядро
    - Если система не грузится — LiveCD + chroot
    - Удалить новое ядро: ==apt remove linux-image-6.x.x  , update-grub==



-------------------------------------------------------------------------------
# 🧪 2. План тестирования нового ядра (Enterprise)

## ✅ 2.1. Цель тестирования

Подтвердить:

- стабильность системы
    
- корректность драйверов
    
- отсутствие регрессий безопасности
    
- соответствие эксплуатационным требованиям
    

---

# 📊 2.2. Типы тестов

## 🔹 1) Smoke Test (базовый)

|Проверка|Команда|Ожидаемый результат|
|---|---|---|
|Версия ядра|`uname -r`|Новая версия|
|Загрузка|`dmesg|grep error`|
|Сервисы|`systemctl status`|Active|

---

## 🔹 2) Hardware Test

### Диски

lsblk  
smartctl -a /dev/sda

### Сеть

ethtool eth0  
iperf3

### RAID/HBA

cat /proc/mdstat  
storcli show

---

## 🔹 3) Performance Test

### CPU / Memory

stress-ng --cpu 8 --vm 4 --timeout 1h

### Disk IO

fio --name=test --rw=randrw --size=10G --numjobs=4

---

## 🔹 4) Stability Test (48–72 часа)

- system load
    
- journald logs
    
- kernel panic / OOPS
    

journalctl -k

---

## 🔹 5) Security Test (для Astra)

### Mandatory Access Control

astra-macctl status

### Integrity

aide --check

### Trusted Boot

mokutil --sb-state

---

## 🔹 6) Application Test

- DB (PostgreSQL, Oracle)
    
- VPN (ViPNet, IPsec)
    
- CryptoPro CSP
    
- Docker / Kubernetes

# 📋 2.3. Acceptance Criteria (критерии приемки)

Система считается готовой если:

- ✔ нет kernel panic / OOPS
    
- ✔ uptime ≥ 72 часа
    
- ✔ performance degradation < 5%
    
- ✔ все сервисы в ACTIVE
    
- ✔ MAC политики работают
    
- ✔ сертифицированные криптомодули функционируют