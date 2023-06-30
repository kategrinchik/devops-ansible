# Домашнее задание к занятию 1 «Введение в Ansible»

## Подготовка к выполнению

1. Установите Ansible версии 2.10 или выше.
2. Создайте свой публичный репозиторий на GitHub с произвольным именем.
3. Скачайте [Playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.

## Основная часть

1. Попробуйте запустить playbook на окружении из `test.yml`, зафиксируйте значение, которое имеет факт `some_fact` для указанного хоста при выполнении playbook.   
  
Значение some_fact - 12.  
  
```Ruby
 ⚡ root@parallels-Parallels-Virtual-Platform  ~/ans/01/playbook   main  ansible-playbook -i inventory/test.yml site.yml

PLAY [Print os facts] *************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************
ok: [localhost]

TASK [Print OS] *******************************************************************************************************************
ok: [localhost] => {
    "msg": "Ubuntu"
}

TASK [Print fact] *****************************************************************************************************************
ok: [localhost] => {
    "msg": 12
}

PLAY RECAP ************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```  
  
2. Найдите файл с переменными (group_vars), в котором задаётся найденное в первом пункте значение, и поменяйте его на `all default fact`.  
  
Выполнено:  
  
```Ruby
 ⚡ root@parallels-Parallels-Virtual-Platform  ~/ans/01/playbook/group_vars/all   main  cat examp.yml 
---
  some_fact: all default fact
```  
  
3. Воспользуйтесь подготовленным (используется `docker`) или создайте собственное окружение для проведения дальнейших испытаний.  
  
Готово:  
  
```Ruby
 ✘ ⚡ root@parallels-Parallels-Virtual-Platform  ~/ans/01/playbook/inventory   main  docker ps -a
CONTAINER ID   IMAGE           COMMAND       CREATED              STATUS              PORTS     NAMES
438781f39c79   ubuntu:latest   "/bin/bash"   About a minute ago   Up About a minute             ubuntu
d7694dcd57ac   centos:7        "/bin/bash"   About a minute ago   Up About a minute             centos7
```  
  
4. Проведите запуск playbook на окружении из `prod.yml`. Зафиксируйте полученные значения `some_fact` для каждого из `managed host`.  
  
some_fact centos7 - el  
some_fact ubuntu - deb  
  
```Ruby
 ⚡ root@parallels-Parallels-Virtual-Platform  ~/ans/01/playbook   main  ansible-playbook -i inventory/prod.yml site.yml

PLAY [Print os facts] ********************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] **************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ************************************************************************************************************************
ok: [centos7] => {
    "msg": "el"
}
ok: [ubuntu] => {
    "msg": "deb"
}

PLAY RECAP *******************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
  
5. Добавьте факты в `group_vars` каждой из групп хостов так, чтобы для `some_fact` получились значения: для `deb` — `deb default fact`, для `el` — `el default fact`.  
  
Готово:  
  
```Ruby
 ⚡ root@parallels-Parallels-Virtual-Platform  ~/ans/01/playbook   main  cat group_vars/*/examp.yml
---
  some_fact: all default fact
---
  some_fact: "deb default fact"
---
  some_fact: "el default fact"
```
  
6.  Повторите запуск playbook на окружении `prod.yml`. Убедитесь, что выдаются корректные значения для всех хостов.  
  
Готово:  
  
```Ruby
 ⚡ root@parallels-Parallels-Virtual-Platform  ~/ans/01/playbook   main  ansible-playbook -i inventory/prod.yml site.yml

PLAY [Print os facts] ********************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] **************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP *******************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
  
7. При помощи `ansible-vault` зашифруйте факты в `group_vars/deb` и `group_vars/el` с паролем `netology`.  
  
Готово:  
  
```Ruby
 ⚡ root@parallels-Parallels-Virtual-Platform  ~/ans/01/playbook   main  cat group_vars/el/examp.yml                   
$ANSIBLE_VAULT;1.1;AES256
33626138326530373139303433613732333066393238386537643836326330643630613631383862
6235373435633865663035303061316438306532313139330a343761663035393638663532666334
34373764623537626133316630353130656337626632383736653935323466383332393762363738
6265616563363937330a313930623266636230646262393432643439383363363639373332303462
62346532353563313064343861343038313738386331323438343261306239386430393363613633
6434646461623937323635643237303561313532636530636163
 ⚡ root@parallels-Parallels-Virtual-Platform  ~/ans/01/playbook   main  cat group_vars/deb/examp.yml                   
$ANSIBLE_VAULT;1.1;AES256
65663630643565643864653435663031343838633635643566653464373261303133386365306235
6232343166653163383262363465353136636662346331320a386662663465323565303562623532
37366330626563343961336462626539343363653066383463356339613563356434363233623838
3132396464643263380a373034356333386534313533376461656238623263633935323937366333
39303737383264393263663231653332333336653865326534313636623866653761646162663737
6130363738313631636232353330353963343434633530356530
```
  
8. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь в работоспособности.  
  
Пароль запрошен.  
  
```Ruby
 ⚡ root@parallels-Parallels-Virtual-Platform  ~/ans/01/playbook   main  ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
Vault password: 

PLAY [Print os facts] ********************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] **************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP *******************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
  
9.  Посмотрите при помощи `ansible-doc` список плагинов для подключения. Выберите подходящий для работы на `control node`.  
  
```Ruby
⚡ root@parallels-Parallels-Virtual-Platform  ~/ans/01/playbook   main  ansible-doc -t connection -l
----------------------------------------------------
local                          execute on controller 
----------------------------------------------------
```
  
Полагаю, речь о плагине local:  
  
Synopsis  
This connection plugin allows ansible to execute tasks on the Ansible ‘controller’ instead of on a remote host.  
  
10. В `prod.yml` добавьте новую группу хостов с именем  `local`, в ней разместите localhost с необходимым типом подключения.  

Готово:  
  
```Ruby
 ⚡ root@parallels-Parallels-Virtual-Platform  ~/ans/01/playbook   main  cat inventory/prod.yml
---
  el:
    hosts:
      centos7:
        ansible_connection: docker
  deb:
    hosts:
      ubuntu:
        ansible_connection: docker

  local:
    hosts:
      localhost:
        ansible_connection: local
```
    
11. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь, что факты `some_fact` для каждого из хостов определены из верных `group_vars`.  
  
some_fact localhost - all default fact  
some_fact ubuntu - deb default fact
some_fact centos7 - el default fact  
  
```Ruby
 ⚡ root@parallels-Parallels-Virtual-Platform  ~/ans/01/playbook   main  ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
Vault password: 

PLAY [Print os facts] ********************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
ok: [localhost]
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] **************************************************************************************************************************
ok: [localhost] => {
    "msg": "Ubuntu"
}
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ************************************************************************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP *******************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```  
  
12. Заполните `README.md` ответами на вопросы. Сделайте `git push` в ветку `master`. В ответе отправьте ссылку на ваш открытый репозиторий с изменённым `playbook` и заполненным `README.md`.
  
Готово.