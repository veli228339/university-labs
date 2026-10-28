[README.md](https://github.com/user-attachments/files/34080962/README.md)
# Лабораторная работа №2. Управление системой и пользователями

*Мамурзаев Вели ИС-25
**ОС:** Debian GNU/Linux 13 (Oracle VirtualBox)

---

## Задание 1. Создание пользователей и групп

Создали трёх пользователей командой `useradd` (опция `-m` — домашний каталог, `-s /bin/bash` — оболочка) и задали им пароли командой `passwd`:

```bash
sudo useradd -m -s /bin/bash admin
sudo useradd -m -s /bin/bash user1
sudo useradd -m -s /bin/bash user2

sudo passwd admin
sudo passwd user1
sudo passwd user2
```

![Создание пользователей admin, user1, user2 и задание паролей](img/01-create-users.png)

Создали группы `developers` и `testers` и распределили пользователей: admin и user1 — в developers, user2 — в testers. Добавление выполняется командой `usermod -aG` (опция `-a` добавляет в группу, не удаляя из остальных). Состав групп проверен командой `getent group`:

```bash
sudo groupadd developers
sudo groupadd testers

sudo usermod -aG developers admin
sudo usermod -aG developers user1
sudo usermod -aG testers user2

getent group developers
getent group testers
```

![Создание групп developers и testers и добавление пользователей](img/02-add-to-groups.png)

---

## Задание 2. Настройка прав доступа

```bash
sudo mkdir -p /home/admin/data /home/admin/logs

sudo chown admin:admin      /home/admin/data
sudo chmod 700              /home/admin/data

sudo chown admin:developers /home/admin/logs
sudo chmod 770              /home/admin/logs
```

![Создание каталогов data и logs, настройка прав chown/chmod](img/03-permissions.png)

Кратко о командах: **`chown`** — меняет владельца и группу файла (`admin:admin` — это `пользователь:группа`), **`chmod`** — меняет права доступа. Числовая запись: **`700`** = `rwx------` — полный доступ только владельцу (каталог `data`), **`770`** = `rwxrwx---` — полный доступ владельцу и группе developers (каталог `logs`). На скриншоте видно, что посторонний пользователь `vboxuser` получает «Отказано в доступе» — данные admin изолированы.

---

## Задание 3. Команды управления пользователями

Изученные команды:

| Команда | Назначение |
|---------|------------|
| `useradd` | создание пользователя |
| `usermod` | изменение учётной записи (группы, блокировка) |
| `groupadd` | создание группы |
| `passwd` | смена пароля |
| `chown` | смена владельца/группы файла |
| `chmod` | смена прав доступа |

Сменили пароль user1, заблокировали user2 (`usermod -L`), проверили статус командой `passwd -S` (буква `L` — locked, учётная запись заблокирована), затем разблокировали обратно (`usermod -U`):

```bash
sudo passwd user1
sudo usermod -L user2
sudo passwd -S user2
sudo usermod -U user2
```

![Смена пароля user1, блокировка user2 (статус L) и разблокировка](img/04-user-commands.png)

---

## Задание 4. Мониторинг активности пользователей

- **`who`** — кто находится в системе сейчас: имя, терминал, время входа;
- **`w`** — то же, плюс uptime, средняя загрузка и текущая команда каждого пользователя;
- **`last`** — история входов в систему из журнала `/var/log/wtmp`.

![Мониторинг активности: who, w, last](img/05-monitoring.png)

На скриншоте: `who` и `w` показывают локальные сессии (`seat0`, `tty2`) и SSH-сессию (`sshd pts/1`) с адреса `10.0.2.2` — это подключение с основного ПК из лабораторной работы №1. `last` отображает историю входов, включая перезагрузки системы (`reboot`, `crash`).

---

## Задание 5. Автоматизация управления пользователями

Файл `users.txt` (формат строки: `логин группа`):

```
alice developers
bob testers
charlie developers
```

Скрипт `create_users.sh`:

```bash
#!/bin/bash
# Массовое создание пользователей и групп из файла users.txt
# Формат строки: <логин> <группа>

USERS_FILE="users.txt"

if [ ! -f "$USERS_FILE" ]; then
    echo "Error: file $USERS_FILE not found!"
    exit 1
fi

while read -r username group; do
    [ -z "$username" ] && continue          # пропуск пустых строк

    # создать группу, если её ещё нет
    if ! getent group "$group" >/dev/null; then
        echo "Creating group $group..."
        groupadd "$group"
    fi

    # создать пользователя, если его ещё нет
    if id "$username" >/dev/null 2>&1; then
        echo "User $username already exists, skip"
    else
        echo "Creating user $username (group $group)..."
        useradd -m -s /bin/bash -G "$group" "$username"
        echo "User $username was successfully created"
    fi
done < "$USERS_FILE"

echo "Program end"
```

Скрипт построчно читает файл: проверки `getent group` и `id` защищают от повторного создания существующих групп и пользователей. Запуск:

```bash
chmod +x create_users.sh
sudo ./create_users.sh
```

![Текст скрипта create_users.sh](img/06-script.png)

![Выполнение скрипта: созданы alice, bob, charlie](img/07-script-run.png)

---

## Выводы

В ходе работы созданы пользователи и группы, настроены права доступа к каталогам средствами `chown` и `chmod`, выполнены смена пароля и блокировка учётной записи, изучен мониторинг активности (`who`, `w`, `last`) и автоматизировано массовое создание пользователей bash-скриптом с защитой от дублирования учётных записей.

