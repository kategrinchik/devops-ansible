## 08-ansible-02-playbook

#### Назначение playbook
Playbook предназначен для установки clickhouse и vector на ВМ с Centos7.  

#### Параметры playbook
- Ansible подключается к хосту по ssh, используя ed25519 ключ.
- Версии clickhouse и vector заданы в vars и могут быть изменены.
- Для установки clickhouse и vector playbook в обоих случаях использует пакетный менеджер rpm.
- Прописана отдельная таска для создания первоначальной DB в clickhouse.
- В дополнение добавлен handler для перезапуска clickhouse при необходимости.

#### Теги playbook
Для clickhouse и vector прописаны парные теги, позволяющие:
- запускать только таски, связанные с clickhouse или vector (tags clickhouse/vector)
- запускать только таски, связанные с загрузкой или установкой (tags distrib/install)
- запускать отдельно таск с созданием DB clickhouse (tag db_create)
- задавать переменные окружения для vector (tag set_env)
