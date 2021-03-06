![ansible](./images/ansible-logo.png) ![wireguard](./images/wireguard-logo.png)

Описание всех playbooks:
------------------------
1. **wg-create-playbook**:
Данный playbook позволяет добавлять/изменять/удалять учётные записи Wireguard. 
Для указания конфигурации пользовательских данных необходимо использовать файл с переменными ```wg_server```
в директории ```host_vars```:
* Определение типа подключения к серверу (по-умолчанию используется SSH):
```yml
ansible_connection: ssh
```
* Указание SSH-порта сервера (в текущем примере используется порт TCP/2235):
```yml
ansible_ssh_port: 2235
```
* Параметр, определяющий необходимость перезапуска службы Wireguard на сервере. Данная переменная может принимать
два значения - True (перезапускать) и False (не перезапускать):
```yml
wg_reload: True
```
* Переменная, содержащая название файла конфигурации на сервере Wireguard (в текущем примере - 'wg0-server'):
```yml
wg_conf_name: 'wg0-server'
```
* Переменная, содержащая полный путь к файлу конфигурации на сервере Wireguard:
```yml
wg_conf_file: /etc/wireguard/{{ wg_conf_name }}.conf
```
* Переменная, содержащая путь к шаблону блока конфигурации (.j2) для добавления учётных записей 
(распологается в директории текущего проекта):
```yml
wg_conf_template: templates/wg0-server.j2
```
* Переменная, содержащая список с конфигурацией _добавляемых/изменяемых_ учётных данных:
  * **name** - имя и фамилия пользователя (например, Иван Иванов);
  * **login** - логин пользователя (например, iivanov);
  * **ip** - туннельный IP-адрес пользователя (например, 192.168.245.10);
  * **pubkey** - публичный ключ пользователя (например, 'XoHf+Ev47030/CKvhGyNm6Rz9SoPBsa1FP1HH/aPm2M='):
  * **privkey** - приватный ключ пользователя (например, 'qHm6wekhnbMGZc3U4B1V3m7nHoHxo2a7V0/F5/eFQXI=')
```yml
wg_add_users:
- {name: 'Иван Иванов', login: 'iivanov', ip: '192.168.245.10', pubkey: 'XoHf+Ev47030/CKvhGyNm6Rz9SoPBsa1FP1HH/aPm2M=', privkey: 'qHm6wekhnbMGZc3U4B1V3m7nHoHxo2a7V0/F5/eFQXI='}
- {name: 'Пётр Петров', login: 'ppetrov', ip: '192.168.245.11', pubkey: '/Cjx4qVvhHXEYIU/muL2B3WjkDfcMhZm+zmY+t8U83U='}
```

> Если добавление/изменение нового пользователя **не планируется** при запуске данного playbook достаточно оставить
переменную ```wg_add_users``` **без значения** (пустой):
```yml
wg_add_users:
#- {name: 'Иван Иванов', login: 'iivanov', ip: '192.168.248.10', pubkey: 'XoHf+Ev47030/CKvhGyNm6Rz9SoPBsa1FP1HH/aPm2M='}
```

* Переменная, содержащая список с логинами _удаляемых_ учётных данных:
```yml
wg_del_users:
- 'oolegov'
- 'ssergeyev'
```
> Если удаление пользователя **не планируется** при запуске данного playbook достаточно оставить
переменную ```wg_del_users``` **без значения** (пустой):
```yml
wg_del_users:
#- 'oolegov'
```

По-мимо переменных, необходимых для добавления блока конфигурации на сервер Wireguard, также используются
переменные для формирования файла конфигурации клиента. Ниже представлен пример:
```yml           
wg_server_pubkey: 9DrAt3LYdDDmdRJNznJ9/hWZc0dEIiCRK+oszZGHX0f= # Публичный ключ сервера Wireguard
wg_user_prefix: 24 # Префикс клиентской сети Wireguard
wg_user_dns: '10.201.0.53,10.202.0.53,1.1.1.1,example.com' # DNS-серверы, домены для поиска DNS клиентской сети
wg_user_endpoint: wg.example.com:39999 # Публичный адрес и порт сервера Wireguard
wg_user_allowips: '10.0.0.0/24, 192.168.205.0/24, 213.26.36.105/32, 94.248.191.60/32' # Список разрешённых маршрутов для клиента
```

2. **wg-show-playbook**:
Данный playbook позволяет просматривать информацию обо всех активных учётных записях Wireguard. Для получения данных
о пути файла конфигурации на сервере используется соответствующая переменная ```wg_conf_file``` в файле ```wg_server```.

Режимы работы wg-create-playbook:
----------------------------------
_**Добавление клиента:**_
Существует три режима данной операции:
* **Добавление клиента с указанием его публичного и секретного ключей** - в словаре внутри **wg_add_users** используются 
сразу два ключа ```pubkey``` и ```privkey```. Предполагается, что мы владеем сразу парой ключей.
Например:
```yml
wg_add_users:
- {name: 'Иван Иванов', login: 'iivanov', ip: '192.168.250.10', pubkey: 'Cjx44qVvhHXEYIU/muL2B3WjkDfcMhZm+zmY+t8U83U=', privkey: 'oD8FQxvu2Zk9mheNZ4WzxLuhdHjyAq7wJkCZFFX0/Gw='}
```
> При данном режиме в конфигурацию сервера будет записан явно переданный **public key** клиента.
А в его сформированном config-файле будет записан указанный приватный ключ.

* **Добавление клиента с указанием только публичного ключа** - в словаре внутри **wg_add_users** используется
один ключ ```pubkey```. Предполагается, что клиент нам передал лишь публичную часть пары ключей. Например:
```yml
wg_add_users:
- {name: 'Иван Иванов', login: 'iivanov', ip: '192.168.250.10', pubkey: 'Cjx44qVvhHXEYIU/muL2B3WjkDfcMhZm+zmY+t8U83U='}
```
> При данном режиме  в конфигурацию сервера будет записан явно переданный **public key** клиента.
А в его сформированном config-файле графа ```PrivateKey``` будет пустой. Предполагается, что он самостоятельно 
укажет приватный ключ в клиентском файле конфигурации.

* **Добавление клиента без указания ключей** - в словаре внутри **wg_add_users** ключи ```pubkey``` и ```privkey```
не указываются вообще. Например:
```yml
wg_add_users:
- {name: 'Иван Иванов', login: 'iivanov', ip: '192.168.250.10'}
```
> При данном режиме приватный и публичный ключи клиента будут сформированы _автоматически_. 
В файлах конфигурации сервера и клиента будут записаны случайно сгенерированные ключи.

После добавления нового клиента его файл конфигурации будет записан в директорию, путь которой указан 
в переменной ```wg_user_template``` (по-умолчанию: ```/CONFIGS``` внутри директории проекта).

_**Изменение клиента:**_

К примеру, если мы хотим произвести **изменение IP-адреса клиента**, то мы должны заполнить все ключи в словаре 
(кроме ```ip```) внутри **wg_add_users** **_старыми_** значениями: 
  * **name**;
  * **login**;
  * **pubkey**;
  * **privkey** (необязательно, опционально для записи в conf-файл клиента).

В ключе ```ip``` необходимо указать новый IP-адрес. 

><span style="color:red">*Важно!*</span> Если, к примеру, ключ ```pubkey``` в режиме изменения клиентских настроек
использовать с новым значением, либо не использовать вообще (для автоматической генерации), тогда по-мимо IP-адреса
клиента будет изменён и его публичный ключ. Поэтому, если планируется изменить только IP, необходимо явно указать 
текущий ```pubkey``` клиента, в противном случае будет изменён и он. В свою очередь, если планируется изменить только
публичный ключ, IP-адрес клиента нужно указать прежний. Значения ```name``` и ```login``` всегда необходимо указывать
без изменения - нарушение данного требования приведёт к созданию нового клиента.
> 
> Изменённый файл конфигурации клиента будет записан в директорию, путь которой указан 
в переменной ```wg_user_template``` (по-умолчанию: ```/CONFIGS``` внутри директории проекта).

>Для получения значений ```name```, ```login```, ```ip``` и ```pubkey``` существующих клиентов необходимо запустить
**wg-show-playbook**.

Мониторинг:
--------------
Имеется небольшая интеграция с проектом по сбору статистики пользователей в Prometheus: 
https://github.com/MindFlavor/prometheus_wireguard_exporter.git

Данный playbook добавляет специальный комментарии в конфигурацию сервера при добавлении каждого пользователя. Например:
```
# friendly_name = iivanov
```
Это помогает парсить блок конфигурации с описанием клиента и собирать статистику по нему в Prometheus. 
