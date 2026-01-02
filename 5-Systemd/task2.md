# Пишем юниты

1. Создайте скрипт который создаёт папку заполняет её файлами ( имена 1-4 ) и записывает в них информацию
о текущей дате, версии ядра, имени компьютера и списе всех файлов в домашнем каталоге пользователя от которого выполняется скрипт( не забудьте сдлеать проверку на существование файлов и папок)
Ответ: Cкрипт:
 ```bash
#!/bin/bash
TARGET_DIR="script_folder"

if [ ! -d "$TARGET_DIR" ]; then
    echo "Создаём папку: $TARGET_DIR"
    mkdir -p "$TARGET_DIR"
else
    echo "Папка $TARGET_DIR уже существует."
fi

CURRENT_DATE=$(date)
KERNEL_VERSION=$(uname -r)
HOSTNAME=$(hostname)
HOME_FILES=$(ls -la ~)

write_info_to_file() {
    local file="$1"
    cat > "$file" <<EOF
Текущая дата: $CURRENT_DATE
Версия ядра: $KERNEL_VERSION
Имя компьютера: $HOSTNAME
Файлы в домашней директории:
$HOME_FILES
EOF
}
for i in {1..4}; do
    filepath="$TARGET_DIR/$i"
    if [ ! -f "$filepath" ]; then
        echo "Создаём файл: $filepath"
        touch "$filepath"
    else
        echo "Файл $filepath уже существует."
    fi

    write_info_to_file "$filepath"
done

echo "Скрипт завершён."
```  
2. Создайте юнит который будет вызывать этот скрипт при запуске. Проверьте
Ответ: Создаём файл юнита: `nano /etc/systemd/system/folder-service.service`.
`systemctl daemon-reload`
`systemctl enable folder-service.service`
`systemctl start folder-service.service`
3. Создайте таймер который будет вызывать выполнение одноимённого systemd юнита каждые 5 минут.
Ответ: В начале создаём таймер: `sudo nano /etc/systemd/system/folder-service.timer`.
```bash
[Unit]  
Description=Run folder-service every 5 minutes  

[Timer]  
OnBootSec=5min  
OnUnitActiveSec=5min
Unit=folder-service.service

[Install]  
WantedBy=timers.target  
```
После включения проверяем статус таймера командами:
```bash  
sudo systemctl daemon-reload  
sudo systemctl enable folder-service  
sudo systemctl start folder-service  
systemctl status folder-service  
```
4. От какого пользователя вызыаются юниты поумолчанию?
Ответ: По умолчанию юниты вызываются от root, если в них не указано другой пользователь
5. Создайте пользователя от имени которого будет выполняться ваш скрипт.
Ответ:
 ```bash 
#!/bin/bash
sudo useradd /bin/bash -m unituser  
```
6. Дополните юнит информацией о пользователе от которого должен выплняться скрипт.
Ответ: Изменение folder-service.service  
```bash
[Service]  
Type=oneshot  
User=unituser  
ExecStart=/usr/local/bin/folder-script.sh  
```
7. Дополните ваш скрипт так, что бы он независимо от местоположения всега выполнялся в домашней папке того кто его вызывает.
Ответ: В самое начало добавим переход в домашний каталог: cd "$HOME"  