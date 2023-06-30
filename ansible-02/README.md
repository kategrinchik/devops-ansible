# Домашнее задание к занятию 2 «Работа с Playbook»

## Подготовка к выполнению

1. * Необязательно. Изучите, что такое [ClickHouse](https://www.youtube.com/watch?v=fjTNS2zkeBs) и [Vector](https://www.youtube.com/watch?v=CgEhyffisLY).
2. Создайте свой публичный репозиторий на GitHub с произвольным именем или используйте старый.
3. Скачайте [Playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.
4. Подготовьте хосты в соответствии с группами из предподготовленного playbook.

## Основная часть

1. Подготовьте свой inventory-файл `prod.yml`.  
  
[prod.yml](https://github.com/kategrinchik/devops-ansible/blob/main/ansible-02/playbook/inventory/prod.yml) с одним хостом готов.  
  
Сначала была выполнена проверка site.yml только с clickhouse, чтобы убедиться, что с хостом проблем нет:  
  
```Ruby
 ⚡ root@parallels-Parallels-Virtual-Platform  ~/ans/02/playbook   main  ansible-playbook -i inventory/prod.yml site.yml

PLAY [Install Clickhouse] *********************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************
ok: [clickhouse-01]

TASK [Get clickhouse distrib] *****************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 5, "gid": 1000, "group": "test", "item": "clickhouse-common-static", "mode": "0664", "msg": "Request failed", "owner": "test", "response": "HTTP Error 404: Not Found", "secontext": "unconfined_u:object_r:user_home_t:s0", "size": 246310036, "state": "file", "status_code": 404, "uid": 1000, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.noarch.rpm"}

TASK [Get clickhouse distrib] *****************************************************************************************************
ok: [clickhouse-01]

TASK [Install clickhouse packages] ************************************************************************************************
fatal: [clickhouse-01]: FAILED! => {"msg": "Missing sudo password"}

PLAY RECAP ************************************************************************************************************************
clickhouse-01              : ok=2    changed=0    unreachable=0    failed=1    skipped=0    rescued=1    ignored=0 
```  
  
Пользователь test приведен в соответствие.  
  
На созданном окружении таска, описанная в rescue, позволяет установить нужный пакет. Чтобы не видеть ошибку failed в процессе play соответствующие строки в site.yml закомментированны.  
  
Результат по clickhouse:  
  
```Ruby  
root@parallels-Parallels-Virtual-Platform  ~/ans/02/playbook   main  ansible-playbook -i inventory/prod.yml site.yml

PLAY [Install Clickhouse] ****************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
ok: [clickhouse-01]

TASK [Get clickhouse distrib] ************************************************************************************************************
ok: [clickhouse-01]

TASK [Install clickhouse packages] *******************************************************************************************************
ok: [clickhouse-01]

TASK [Create database] *******************************************************************************************************************
ok: [clickhouse-01]

PLAY RECAP *******************************************************************************************************************************
clickhouse-01              : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```  
  
1. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает [vector](https://vector.dev).
2. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.
3. Tasks должны: скачать дистрибутив нужной версии, выполнить распаковку в выбранную директорию, установить vector.
4. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.
  
```Ruby
 ⚡ root@parallels-Parallels-Virtual-Platform  ~/ans/02/playbook   main  ansible-lint site.yml 
WARNING  Overriding detected file kind 'yaml' with 'playbook' for given positional argument: site.yml
```
  
Данный warning по информации от Ansible не требует никаких действий.  
  
5. Попробуйте запустить playbook на этом окружении с флагом `--check`.  
  
```Ruby
TASK [Install Vector] *************************************************************************************************************
fatal: [clickhouse-01]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'vector' is undefined\n\nThe error appears to be in '/home/parallels/ansible_hw/02/playbook/site.yml': line 41, column 7, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n  tasks:\n    - name: Install Vector\n      ^ here\n"}

PLAY RECAP ************************************************************************************************************************
clickhouse-01              : ok=4    changed=0    unreachable=0    failed=1    skipped=1    rescued=0    ignored=0  
```  
  
Прописала требуемые variables для vector: [vars.yml](https://github.com/kategrinchik/devops-ansible/blob/main/ansible-02/playbook/group_vars/clickhouse/vars.yml)  

Check запущен еще раз:  
  
```Ruby
 ⚡ root@parallels-Parallels-Virtual-Platform  ~/ans/02/playbook   main  ansible-playbook -i inventory/prod.yml site.yml --check

PLAY [Install Clickhouse] ****************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
ok: [clickhouse-01]

TASK [Get clickhouse distrib] ************************************************************************************************************
ok: [clickhouse-01]

TASK [Install clickhouse packages] *******************************************************************************************************
ok: [clickhouse-01]

TASK [Create database] *******************************************************************************************************************
skipping: [clickhouse-01]

PLAY [Install vector] ********************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
ok: [clickhouse-01]

TASK [Get vector distrib] ****************************************************************************************************************
changed: [clickhouse-01]

TASK [Create directory vector] ***********************************************************************************************************
changed: [clickhouse-01]

TASK [Unarchive vector] ******************************************************************************************************************
fatal: [clickhouse-01]: FAILED! => {"changed": false, "msg": "dest '/prod/vector' must be an existing dir"}

PLAY RECAP *******************************************************************************************************************************
clickhouse-01              : ok=6    changed=2    unreachable=0    failed=1    skipped=1    rescued=0    ignored=0 
```  
  
Во время --check папка не создается.  
  
6. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.  
  
```Ruby
✘ ⚡ root@parallels-Parallels-Virtual-Platform  ~/ans/02/playbook   main  ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Install Clickhouse] ****************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
ok: [clickhouse-01]

TASK [Get clickhouse distrib] ************************************************************************************************************
ok: [clickhouse-01]

TASK [Install clickhouse packages] *******************************************************************************************************
ok: [clickhouse-01]

TASK [Create database] *******************************************************************************************************************
ok: [clickhouse-01]

PLAY [Install vector] ********************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
ok: [clickhouse-01]

TASK [Get vector distrib] ****************************************************************************************************************
ok: [clickhouse-01]

TASK [Create directory vector] ***********************************************************************************************************
ok: [clickhouse-01]

TASK [Unarchive vector] ******************************************************************************************************************
fatal: [clickhouse-01]: FAILED! => {"changed": false, "msg": "Failed to find handler for \"./vector-0.30.0-x86_64-unknown-linux-musl.tar.gz\". Make sure the required command to extract the file is installed. Command \"/bin/gtar\" could not handle archive. Command \"/bin/unzip\" could not handle archive."}

PLAY RECAP *******************************************************************************************************************************
clickhouse-01              : ok=7    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
```  

Документация Vector настоятельно не рекомендует устанавливать Vector с помощью архива:  
[Источник](https://vector.dev/docs/setup/installation/manual/from-archives/)  
Ошибка вызвана разночтениями действий различных версий ansible на разных хостах при разархивировании и является распространенной и известной сообществу.  
Vector рекомендуется ставить пакетным менеджером. В результате изменила site.yml, отказавшись от закачки архива в пользу rpm. Установка yum из rpm также не требует блока создания соотвествующей директории:  
  
```Ruby
⚡ root@parallels-Parallels-Virtual-Platform  ~/ans/02/playbook   main  ansible-playbook -i inventory/prod.yml site.yml --diff 

PLAY [Install Clickhouse] ****************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
ok: [clickhouse-01]

TASK [Get clickhouse distrib] ************************************************************************************************************
ok: [clickhouse-01]

TASK [Install clickhouse packages] *******************************************************************************************************
ok: [clickhouse-01]

TASK [Create database] *******************************************************************************************************************
ok: [clickhouse-01]

PLAY [Install vector] ********************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
ok: [clickhouse-01]

TASK [Get vector distrib] ****************************************************************************************************************
changed: [clickhouse-01]

TASK [Install vector package] ************************************************************************************************************
changed: [clickhouse-01]

TASK [Set vector environment] ************************************************************************************************************
--- before
+++ after: /root/.ansible/tmp/ansible-local-698608jp263_j/tmp_ip2g0i9/vector.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export VECTOR_HOME=/prod/vector - vector
+export PATH=$PATH:$VECTOR_HOME/bin

changed: [clickhouse-01]

PLAY RECAP *******************************************************************************************************************************
clickhouse-01              : ok=8    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

7. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.  
  
Повторный запуск с флагом --diff не показывает изменений:
  
```Ruby
 ⚡ root@parallels-Parallels-Virtual-Platform  ~/ans/02/playbook   main  ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Install Clickhouse] ****************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
ok: [clickhouse-01]

TASK [Get clickhouse distrib] ************************************************************************************************************
ok: [clickhouse-01]

TASK [Install clickhouse packages] *******************************************************************************************************
ok: [clickhouse-01]

TASK [Create database] *******************************************************************************************************************
ok: [clickhouse-01]

PLAY [Install vector] ********************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
ok: [clickhouse-01]

TASK [Get vector distrib] ****************************************************************************************************************
ok: [clickhouse-01]

TASK [Install vector package] ************************************************************************************************************
ok: [clickhouse-01]

TASK [Set vector environment] ************************************************************************************************************
ok: [clickhouse-01]

PLAY RECAP *******************************************************************************************************************************
clickhouse-01              : ok=8    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
  
8.  Подготовьте README.md-файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.

[README.md](https://github.com/kategrinchik/devops-ansible/blob/main/ansible-02/playbook/README.md)  
  
9.  Готовый playbook выложите в свой репозиторий, поставьте тег `08-ansible-02-playbook` на фиксирующий коммит, в ответ предоставьте ссылку на него.  
  
[ansible-02](https://github.com/kategrinchik/devops-ansible/tree/main/ansible-02)