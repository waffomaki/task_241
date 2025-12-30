# Настриваем

1. Какой по умолчанию используется порт для поключения?<br>
22 (TCP)

2. Можно ли его изменить? если да то как?<br>
Да. Открыть файл конфигурации ssh сервера, найти строку Port и указать нужный.

3. Какая служба отвечает за обработку запросов на подключения по ssh?<br>
sshd (ssh daemon)

4. Какой файл конфигурации отвечает за его настройку?<br>
```/etc/openssh/sshd_config``` Это основной конфигурационный файл SSH-сервера. Все настройки здесь.<br>

5. Попробуйте подключиться по ssh к предоставленному вам серверу<br>
```ssh student@ternar.io -p 241```, -p указывает на нестандартный порт<br>
Вводим пароль.<br>
![src/t1_1.png](src/t1_1.png)

6. Отредактируйте файл настроек на сервере так, чтобы была возможность подключиться к серверу используя пользователя root<br>
Перед началом работы:
```
sudo apt-get update
sudo apt-get install nano
```
Теперь можно использовать nano.
Открываем конфиг: ```sudo nano /etc/openssh/sshd_config```<br>
Ищем строку ```#PermitRootLogin prohibit-password``` и заменеяем на ```PermitRootLogin yes```<br>
![src/t1_2.png](serc/t1_2.png)
Это разрешает вход по паролю.<br>
Перезапускаем SSH: ```sudo systemctl restart sshd```<br>

7. Измените колличество ошибок ввода пароля перед сборосом соединения, покажите эти измененения<br>
Настраивается через ```MaxAuthTries``` в ```/etc/openssh/sshd_config```:<br>
Пусть будет 3, тогда: ```MaxAuthTries 3```<br>
![src/t1_3.png](serc/t1_3.png)
Выведем изменения: ```sudo grep -i maxauthtries /etc/openssh/sshd_config```
![src/t1_4.png](serc/t1_4.png)
Перезапускаем SSH: ```sudo systemctl restart sshd```

8. Создайте пользователя ssh-user и попробуйте им подключиться к серверу<br>
Создаем юзера:<br>
```sudo useradd -m -s /bin/bash ssh-user```<br>
Задаем пароль:<br>
```sudo passwd ssh-user```
Подключаемся:<br>
```exit```
```ssh ssh-user@ternar.io -p 241```
Вводим пароль.<br>
![src/t1_5.png](serc/t1_5.png)<br>

9. Ограничте ему возможность подключения к серверу<br>
Добавим в ```/etc/openssh/sshd_config``` следующее: ```DenyUsers ssh-user```<br>
![src/t1_6.png](serc/t1_6.png)
Перезапускаем SSH: ```sudo systemctl restart sshd```

10. Как вы это сделали?<br>
Я добавил строку ```DenyUsers ssh-user``` в файл конфигурации ```/etc/ssh/sshd_config```, что запрещает пользователю ssh-user подключаться по SSH. После перезапуска sshd изменения вступили в силу.
![src/t1_7.png](serc/t1_7.png)

11. Что хранится в файле known_hosts?<br>
Файл хранит отпечатки (fingerprints) публичных ключей удалённых серверов, к которым ты подключался.