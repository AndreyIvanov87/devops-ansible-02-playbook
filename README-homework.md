# Домашнее задание к занятию "08.02 Работа с Playbook"

## Подготовка к выполнению

1. Создайте свой собственный (или используйте старый) публичный репозиторий на github с произвольным именем.
2. Скачайте [playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.
3. Подготовьте хосты в соответствии с группами из предподготовленного playbook.
```
vagrant@server4:~/devops-ansible-02-playbook$ ansible-vault create group_vars/clickhouse/password.yml
New Vault password: 
..............
Confirm New Vault password: 
TASK [Gathering Facts] ******************************************************************************
fatal: [clickhouse-01]: FAILED! => {"msg": "to use the 'ssh' connection type with passwords or pkcs11_provider, you must install the sshpass program"}
.............
vagrant@server4:~/devops-ansible-02-playbook$ sudo apt-get install sshpass

```
## Основная часть

1. Приготовьте свой собственный inventory файл `prod.yml`.
2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает [vector](https://vector.dev).
3. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.
4. Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, установить vector.
5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.
6. Попробуйте запустить playbook на этом окружении с флагом `--check`.
```bash
vagrant@server4:~/devops-ansible-02-playbook$ ansible-playbook -i inventory/prod.yml   --ask-vault-pass site.yml  --check
Vault password: 

PLAY [Install Clickhouse] ***************************************************************************

TASK [Gathering Facts] ******************************************************************************
ok: [clickhouse-01]

TASK [Get clickhouse distrib] ***********************************************************************
changed: [clickhouse-01] => (item=clickhouse-client)
changed: [clickhouse-01] => (item=clickhouse-server)
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 10, "item": "clickhouse-common-static", "msg": "Request failed", "response": "HTTP Error 404: Not Found", "status_code": 404, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.noarch.rpm"}

TASK [Get clickhouse distrib] ***********************************************************************
changed: [clickhouse-01]

TASK [Install clickhouse packages] ******************************************************************
fatal: [clickhouse-01]: FAILED! => {"changed": false, "msg": "No RPM file matching 'clickhouse-common-static-22.3.3.44.rpm' found on system", "rc": 127, "results": ["No RPM file matching 'clickhouse-common-static-22.3.3.44.rpm' found on system"]}

PLAY RECAP ******************************************************************************************
clickhouse-01              : ok=2    changed=1    unreachable=0    failed=1    skipped=0    rescued=1    ignored=0   
```
7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.
```bash
vagrant@server4:~/devops-ansible-02-playbook$ ansible-playbook -i inventory/prod.yml   --ask-vault-pass site.yml  --diff
Vault password: 

PLAY [Install Clickhouse] ***************************************************************************

TASK [Gathering Facts] ******************************************************************************
ok: [clickhouse-01]

TASK [Get clickhouse distrib] ***********************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 0, "gid": 1000, "group": "vagrant", "item": "clickhouse-common-static", "mode": "0774", "msg": "Request failed", "owner": "vagrant", "response": "HTTP Error 404: Not Found", "secontext": "unconfined_u:object_r:user_home_t:s0", "size": 246310036, "state": "file", "status_code": 404, "uid": 1000, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.noarch.rpm"}

TASK [Get clickhouse distrib] ***********************************************************************
ok: [clickhouse-01]

TASK [Install clickhouse packages] ******************************************************************
ok: [clickhouse-01]

TASK [Create database] ******************************************************************************
ok: [clickhouse-01]

PLAY [Install Vector] *******************************************************************************

TASK [Gathering Facts] ******************************************************************************
ok: [clickhouse-01]

TASK [fix curl problem] *****************************************************************************
changed: [clickhouse-01]

TASK [fix ca serts] *********************************************************************************
ok: [clickhouse-01]

TASK [fetch repository] *****************************************************************************
ok: [clickhouse-01]

TASK [update repo list] *****************************************************************************
changed: [clickhouse-01]

TASK [install vector package] ***********************************************************************
changed: [clickhouse-01]

RUNNING HANDLER [start vector] **********************************************************************
changed: [clickhouse-01]

PLAY RECAP ******************************************************************************************
clickhouse-01              : ok=11   changed=4    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0   
```
8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.
```bash
...
PLAY [Install Vector] *******************************************************************************

TASK [Gathering Facts] ******************************************************************************
ok: [clickhouse-01]

TASK [fix curl problem] *****************************************************************************
changed: [clickhouse-01]

TASK [fix ca serts] *********************************************************************************
ok: [clickhouse-01]

TASK [fetch repository] *****************************************************************************
ok: [clickhouse-01]

TASK [update repo list] *****************************************************************************
changed: [clickhouse-01]

TASK [install vector package] ***********************************************************************
ok: [clickhouse-01]

PLAY RECAP ******************************************************************************************
clickhouse-01              : ok=10   changed=2    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0   
```
  
```bash
[vagrant@localhost ~]$ sudo service vector status
Redirecting to /bin/systemctl status  vector.service
● vector.service - Vector
   Loaded: loaded (/usr/lib/systemd/system/vector.service; disabled; vendor preset: disabled)
   Active: active (running) since Пт 2022-06-24 13:36:38 UTC; 8min ago
```
9. Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.
10. Готовый playbook выложите в свой репозиторий, поставьте тег `08-ansible-02-playbook` на фиксирующий коммит, в ответ предоставьте ссылку на него.

---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
