# Домашнее задание к занятию 3 «Использование Ansible»

## Подготовка к выполнению

1. Подготовьте в Yandex Cloud три хоста: для `clickhouse`, для `vector` и для `lighthouse`.
![](img/8.3-hosts.jpg)
2. Репозиторий LightHouse находится [по ссылке](https://github.com/VKCOM/lighthouse).

## Основная часть

1. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает LightHouse.

2. При создании tasks рекомендую использовать модули: `get_url`, `template`, `yum`, `apt`.
3. Tasks должны: скачать статику LightHouse, установить Nginx или любой другой веб-сервер, настроить его конфиг для открытия LightHouse, запустить веб-сервер.
4. Подготовьте свой inventory-файл `prod.yml`.
```
делаю из шаблона терраформ получаю
---
clickhouse:
  hosts:
    clickhouse-01:
      ansible_host: 158.160.106.213
      ansible_user: centos

lighthouse:
  hosts:
    lighthouse-01:
      ansible_host: 158.160.97.155
      ansible_user: centos

vector:
  hosts:
    vector-01:
      ansible_host: 158.160.100.239
      ansible_user: centos

```
5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.
6. Попробуйте запустить playbook на этом окружении с флагом `--check`.
```yml
 ansible-playbook -i inventory/prod.yml site.yml -bK --check
BECOME password:

PLAY [Install Clickhouse] **************************************************************************

TASK [Gathering Facts] *****************************************************************************
The authenticity of host '158.160.106.213 (158.160.106.213)' can't be established.
ECDSA key fingerprint is SHA256://qbg8je8a3iL1A2PMjFRAvEhr11ymxgWCK+hYxVGrU.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
ok: [clickhouse-01] 

TASK [Get clickhouse distrib] **********************************************************************
changed: [clickhouse-01] => (item=clickhouse-client)
changed: [clickhouse-01] => (item=clickhouse-server)
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 0, "item": "clickhouse-common-static", "msg": "Request failed", "response": "HTTP Error 404: Not Found", "status_code": 404, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.noarch.rpm"}

TASK [Get clickhouse distrib] **********************************************************************
changed: [clickhouse-01]

TASK [Install clickhouse packages] *****************************************************************
changed: [clickhouse-01]

TASK [Flush handlers] ******************************************************************************

RUNNING HANDLER [Start clickhouse service] *********************************************************
changed: [clickhouse-01]

TASK [Create database] *****************************************************************************
changed: [clickhouse-01]

TASK [Create table] ********************************************************************************
ok: [clickhouse-01]

TASK [Create clickhouse-server config] *************************************************************
changed: [clickhouse-01]

TASK [ch-server restart] ***************************************************************************
changed: [clickhouse-01]

PLAY [Install Vector] ******************************************************************************

TASK [Gathering Facts] *****************************************************************************
The authenticity of host '158.160.100.239 (158.160.100.239)' can't be established.
ECDSA key fingerprint is SHA256:jkXcEdvHy09luOrvy/1K2yHYeIzMDRk4uGVslhgJmhw.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
ok: [vector-01]

TASK [Get package vector] **************************************************************************
changed: [vector-01]

TASK [Install vector package] **********************************************************************
changed: [vector-01]

TASK [Vector config file] **************************************************************************
changed: [vector-01]

TASK [Vector systemd service file] *****************************************************************
changed: [vector-01]

TASK [Add epel-release repo] ***********************************************************************
changed: [vector-01]

TASK [Install nginx] *******************************************************************************
changed: [vector-01]

TASK [Create nginx config] *************************************************************************
changed: [vector-01]

TASK [Start NGiNX] *********************************************************************************
changed: [vector-01]

RUNNING HANDLER [Start Vector service] *************************************************************
changed: [vector-01]

PLAY [Install nginx] *******************************************************************************

TASK [Gathering Facts] *****************************************************************************
The authenticity of host '158.160.97.155 (158.160.97.155)' can't be established.
ECDSA key fingerprint is SHA256:7rQ6RctRTDuRS9Pkcluvu/flOQuXOaOPAnY7sASOQEA.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
ok: [lighthouse-01]

TASK [Install epel-release] ************************************************************************
changed: [lighthouse-01]

TASK [Install nginx] *******************************************************************************
changed: [lighthouse-01]

PLAY [Install lighthouse] **************************************************************************

TASK [Gathering Facts] *****************************************************************************
ok: [lighthouse-01]

TASK [Lighthouse | Install git] ********************************************************************
changed: [lighthouse-01]

TASK [Lighthouse | Clone repository] ***************************************************************
changed: [lighthouse-01]

TASK [Create Lighthouse config] ********************************************************************
changed: [lighthouse-01]

RUNNING HANDLER [nginx restarted] ******************************************************************
changed: [lighthouse-01]

PLAY RECAP *****************************************************************************************
clickhouse-01              : ok=8    changed=6    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0
lighthouse-01              : ok=8    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
vector-01                  : ok=10   changed=9    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.
8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.


```yml
ansible-playbook -i inventory/prod.yml site.yml -bK --diff
BECOME password:
 .......
PLAY RECAP *****************************************************************************************
clickhouse-01              : ok=7    changed=1    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0
lighthouse-01              : ok=7    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
vector-01                  : ok=10   changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
 
перезапуск служб всегда показывают changed
playbook идемпотентен
----------------------
```

- Логи собираються и передаються в Clickhouse
![](img/8.3-lh.jpg)
9. Подготовьте README.md-файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.

[README.MD](playbook/README.MD)

10. Готовый playbook выложите в свой репозиторий, поставьте тег `08-ansible-03-yandex` на фиксирующий коммит, в ответ предоставьте ссылку на него.

---

### Как оформить решение задания

Выполненное домашнее задание пришлите в виде ссылки на .md-файл в вашем репозитории.

---
