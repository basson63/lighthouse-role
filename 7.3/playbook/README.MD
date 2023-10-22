###Описание Playbook
## Инфаструктура задается terraform и им же создаеться файл inventory/prod.yml

1. PLAY Install Clickhouse
на хосте clickhouse-01
tasks: Get clickhouse distrib
- скачивает дистрибутив clickhouse(client,server)
tasks: Install clickhouse packages
-устанавливает пакеты clickhouse и запускает сервер
tasks: Create database
- создает database с именем logs
tasks: Create table
- создает в database таблицу с именем vector_logs
tasks: Create clickhouse-server config
- копируем конфиг для clickhouse-server из templates чтобы слушать на всех адресах
tasks: ch-server restart
- перезапускаем clickhouse-server с измененным конфигом 

2. PLAY Install Vector
на хосте vector-01
tasks: Get package vector
- скачиваем дистрибутив vector 
tasks: Install vector package
-  устанавливаем пакет vector
tasks: Vector config file
- копируем конфиг для vector из templates
tasks: Vector systemd service file
- копируем конфиг файла для службы vector из templates
и идем в notify: Start Vector service для перезапуска службы
tasks: Add epel-release repo
- ставим репозиторий epel-release
tasks: Install nginx
- устанавливаем веб сервер nginx 
tasks: Create nginx config
- копируем конфиг для nginx из templates для создания логов для vector
tasks: Start NGiNX
- включаем службу и перезапускаем nginx
 и идем в notify: Start Vector service для перезапуска службы vector

3. PLAY Install nginx
на хосте lighthouse-01
tasks: Add epel-release
- ставим репозиторий epel-release
tasks: Install nginx
- устанавливаем веб сервер nginx 


4. PLAY Install lighthouse
на хосте lighthouse-01
pre_tasks: 
- установка git
tasks: Lighthouse | Clone repository
- скачиваем lighthouse через git и копируем в папку {{ lighthouse_dir }} 
tasks: Create Lighthouse config
- копикопируем конфиг для lighthouse для веб сервера nginx из templates 
и последующее перезапуск веб сервера notify: nginx restarted

5. Variables используемые 
- group_vars\clickhouse\clickhouse.yml
clickhouse_version -- задаем версию приложения для установки
clickhouse_packages -- список пакетов для установки
- group_vars\lighthouse\lighthouse.yml
lighthouse_url -- ссылка где находится дистрибутив
lighthouse_dir -- директория куда копировать дистрибутив и откуда запускается веб-приложение
lighthouse_port -- порт на котором будет доступен lighthouse
- group_vars\vecto\vector.yml
vector_version -- задаем версию для установки пакета vector
- templates\vector.toml.j2
{{ hostvars['clickhouse-01'].ansible_host }} -- получение IP адреса хоста clickhouse-01 
