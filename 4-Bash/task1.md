# Скриптуем по полной

1. Что такое шебанг?<br>
Это первая строчка в скрипте. Путь к интерпретатору скрипта.<br>

2. Обязательно ли исполняемый файл дожен иметь соотвествующее расширение?<br>
Нет, исполняемость файла определяется битами прав доступа (флаг x - execute ), а также самим содержимым файла (шебанг)<br>

3. Напишите скрипт который выполнит автоматически действия из блока работы с файлами. ( не забудьте включить set -euo pipefail для того что бы ваш скрипт было удобнее отлаживать. Опишите что включают эти флаги)<br>
```
#!/bin/bash

# Строгий режим для отладки
set -euo pipefail

# Переменные
DISK="/dev/sdb"
PARTITION="${DISK}1"
MOUNT_POINT="/mnt"
FSTAB_FILE="/etc/fstab"

# Существует ли диск
if [ ! -b "$DISK" ]; then
    echo "Ошибка: Диск $DISK не найден. Убедитесь, что он добавлен в ВМ." >&2
    exit 1
fi

# Создание раздела (MBR + один primary)
echo "Создание раздела на $DISK..."
echo -e "o\nn\np\n1\n\n\nw" | sudo fdisk "$DISK" >/dev/null 2>&1

# Ожидание обновления инфы о разделах
sleep 2

# Проверка на раздел
if [ ! -b "$PARTITION" ]; then
    echo "Ошибка: Раздел $PARTITION не создан." >&2
    exit 1
fi

# Создание ФС ext4
echo "Создание ФС ext4 на $PARTITION..."
sudo mkfs.ext4 -F "$PARTITION" >/dev/null

# Монтирование
echo "Монтирование $PARTITION в $MOUNT_POINT..."
sudo mkdir -p "$MOUNT_POINT"
sudo mount "$PARTITION" "$MOUNT_POINT"

# Создание тестовых файлов
echo "Создание тестовых файлов..."
echo "Этот файл создан автоматически." | sudo tee "$MOUNT_POINT/test_file.txt" >/dev/null
sudo touch "$MOUNT_POINT/.keep"

# Отмонтирование
echo "Отмонтирование $MOUNT_POINT..."
sudo umount "$MOUNT_POINT"

# Получаем UUID раздела
UUID=$(sudo blkid -s UUID -o value "$PARTITION")
if [ -z "$UUID" ]; then
    echo "Не удалось получить UUID для $PARTITION." >&2
    exit 1
fi

# Добавление записи в /etc/fstab (если нет)
FSTAB_ENTRY="UUID=$UUID $MOUNT_POINT ext4 defaults 0 2"

if ! grep -qxF "$FSTAB_ENTRY" "$FSTAB_FILE"; then
    echo "Добавление записи в $FSTAB_FILE..."
    echo "$FSTAB_ENTRY" | sudo tee -a "$FSTAB_FILE" >/dev/null
else
    echo "Запись уже есть в $FSTAB_FILE."
fi

# Проверка fstab
echo "Проверка корректности /etc/fstab..."
if sudo mount -a; then
    echo "Диск автоматически смонтирован"
else
    echo "mount -a завершился с ошибкой. Нужно п роверить /etc/fstab вручную." >&2
    exit 1
fi

# Проверка
if mount | grep -q "$MOUNT_POINT"; then
    echo "Диск $PARTITION успешно смонтирован в $MOUNT_POINT."
else
    echo "Диск не смонтирован после mount -a." >&2
    exit 1
fi
```
По флагам ```set -euo pipefail```:<br>
```-e```: скрипт завершится сразу при любой ошибке.<br>
```-u```: использование несуществующей переменной вызовет ошибку.<br>
```-o pipefail```: если команда в пайплайне упадёт, весь пайплайн считается ошибочным.<br>

