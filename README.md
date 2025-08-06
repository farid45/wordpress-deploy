# Установка WordPress на Debian ARM64 в UTM

Этот документ описывает все шаги, которые я выполнил для развертывания WordPress на виртуальной машине Debian ARM64 с использованием UTM и Ansible.


## 1. Подготовка виртуальной машины

1. **Скачал образ Debian ARM64** с официального сайта.  
2. **Запустил виртуальную машину в UTM**:  
   - Выбрал архитектуру ARM64  
   - Настроил сеть (рекомендуется Shared Network для автоматического получения IP)  
   - Установил логин: debian, пароль: debian  
3. **Определил IP-адрес ВМ**:  
```bash
ip a
```
(192.168.64.10)


## 2. Настройка Ansible

1. **Создал рабочую папку**:
```bash
mkdir ~/wordpress-deploy && cd ~/wordpress-deploy
```
2. **Создал inventory.ini (указал IP виртуальной машины)**:
```ini
[webservers]
192.168.64.10 ansible_user=debian ansible_ssh_pass=debian
[webservers:vars]
ansible_python_interpreter=/usr/bin/python3
```
3. **Установил необходимые коллекции Ansible**:
```bash
ansible-galaxy collection install community.mysql
```

## 3. Подготовка Playbook и конфигов

1. **Создал install_wordpress.yml (основной сценарий развертывания)**.
2. **Создал шаблоны конфигурации**:
* wordpress.conf.j2 (виртуальный хост Apache)
* wp-config.php.j2 (настройки WordPress)
3. **Проверил доступность сервера**:
```bash
ansible webservers -i inventory.ini -m ping
```

## 4. Запуск развертывания

1. **Выполнил playbook (с записью логов)**:
```bash
ansible-playbook -i inventory.ini install_wordpress.yml -vvv > install.log 2>&1
```
2. **Проверил результат**:
* Открыл в браузере: http://192.168.64.10
* Если видна стандартная страница Apache (It works!), выполнил:
```bash
ansible webservers -i inventory.ini -a "sudo systemctl restart apache2"
```
## 5. Решение проблем

### Случай 1: Ошибка 403 Forbidden

1. **Проверил права доступа**:
```bash
sudo chown -R www-data:www-data /var/www/wordpress
sudo chmod -R 755 /var/www/wordpress
```
2. **Проверил конфиг Apache**:
```bash
sudo nano /etc/apache2/sites-available/wordpress.conf
```
Убедился, что есть:

```apache
<Directory /var/www/wordpress>
    Options FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```
3. **Перезапустил Apache**:
bash
sudo systemctl restart apache2
### Случай 2: Отсутствует index.php

1. **Проверил содержимое папки**:
```bash
ls -la /var/www/wordpress
```
2. **Если папка пуста — переустановил WordPress**:
```bash
sudo rm -rf /var/www/wordpress/*
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xzvf latest.tar.gz
sudo cp -r wordpress/* /var/www/wordpress/
```
## 6. Проверка работоспособности

1. **Открыл WordPress в браузере**:
```text
http://192.168.64.10
```
2. **Вход в админку**:
```text
http://192.168.64.10/wp-admin
```
* Логин: admin
* Пароль: admin_secure_password (из install_wordpress.yml)
7. Полезные команды для диагностики


|Команда| Описание |
|:---------------------------|--------------:|
|sudo tail -f /var/log/apache2/error.log | Просмотр логов Apache |
|curl -I http://localhost|	Проверка HTTP-ответа |
|sudo apache2ctl configtest |	Проверка конфигурации|

## Итог

* #### WordPress успешно развернут на Debian ARM64 в UTM.
* #### Все настройки сохранены в Ansible-playbook для повторного использования.
* ##### Логи развертывания сохранены в install.log.