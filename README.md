# Infrastructure Automation Practice

## Описание

Этот проект — практическая автоматизация развёртывания и настройки трёх виртуальных машин в Яндекс.Облаке с помощью **Terraform** и **Ansible**.  
Цель: научиться автоматически создавать инфраструктуру и настраивать роли приложений средствами DevOps.

---

## Структура проекта

```
.
├── terraform/
│   ├── main.tf
│   ├── output.tf
│   ├── providers.tf
│   ├── terraform.tfvars         # <- индивидуальные переменные, см. пример ниже!
│   └── variables.tf
|   └── versions.tf
└── ansible/
├── inventory.yaml
├── group\_vars/
│   └── all.yml              # <- переменные для Ansible, см. пример ниже!
├── playbook.yaml
└── roles/
├── base/
│   └── tasks/main.yml
├── nginx\_proxy/
│   ├── handlers/main.yml
│   └── tasks/main.yml
└── nginx\_upstream/
├── tasks/main.yml
└── templates/index.j2

````

---

## Пример файла `terraform.tfvars`

```hcl
yc_cloud_id   = "b1gjfshjrrho60pr7d1b"
yc_folder_id  = "b1gues80g3ra6l16lotu"
yc_zone       = "ru-central1-a"
instance_image = "fd80qm01ah03dkqb14lc"
ssh_public_key = "ssh-ed25519 AAAAC3NzaC1... (ключ id_rsa.pub)"
instance_user  = "ubuntu"
service_account_key_file = "/home/youruser/.config/yandex-cloud/sa-key.json"

virtual_machines = {
  "proxy" = {
    vm_name   = "proxy"
    hostname  = "proxy"
    vm_cpu    = 2
    ram       = 2
    disk      = 30
    disk_name = "proxy-disk"
    role      = "proxy"
  },
  "upstream" = {
    vm_name   = "upstream"
    hostname  = "upstream"
    vm_cpu    = 2
    ram       = 2
    disk      = 30
    disk_name = "upstream-disk"
    role      = "upstream"
  },
  "common" = {
    vm_name   = "common"
    hostname  = "common"
    vm_cpu    = 2
    ram       = 2
    disk      = 30
    disk_name = "common-disk"
    role      = "common"
  }
}
````

---

## Пример файла `ansible/group_vars/all.yml`

```yaml
ansible_user: ubuntu
ansible_ssh_private_key_file: ~/.ssh/id_rsa
ansible_python_interpreter: /usr/bin/python3
ansible_ssh_common_args: '-o StrictHostKeyChecking=no'

proxy_listen_port: 3000
upstream_listen_port: 80
proxy_site_conf: proxy3000
```

---

## Инструкция по запуску

### 1. Сгенерировать SSH-ключ (если у тебя нет своего)

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_rsa
```

(Публичный ключ `~/.ssh/id_rsa.pub` указывается в terraform.tfvars, приватный — в Ansible.)

---

### 2. Развернуть инфраструктуру через Terraform

```bash
cd terraform
terraform init
terraform apply -auto-approve
```

* После успешного выполнения сохрани внешний IP всех машин из вывода (output).

---

### 3. Указать IP в Ansible-инвентаре

Открой файл `ansible/inventory.yaml` и впиши IP из Terraform:

```yaml
all:
  hosts:
    proxy:
      ansible_host: <proxy_external_ip>
    upstream:
      ansible_host: <upstream_external_ip>
    common:
      ansible_host: <common_external_ip>
```

---

### 4. Проверить соединение

```bash
cd ../ansible
ansible all -m ping -i inventory.yaml
```

---

### 5. Запустить автоматическую настройку ролей

```bash
ansible-playbook playbook.yaml -i inventory.yaml
```

---

### 6. Проверить результат

* Открой в браузере:
  `http://<proxy_external_ip>:3000`
  Должна открыться страница с текстом "Hello from upstream!".

---

## Примечания

* Параметр `ansible_ssh_common_args: '-o StrictHostKeyChecking=no'` используется для автоматического подтверждения ключей ssh-хостов при первом подключении.
* Все роли для Ansible хранятся в папке `roles`, задания playbook распределяются по ролям в `playbook.yaml`.
* В качестве облака используется Яндекс.Облако, провайдер Terraform — yandex-cloud.

