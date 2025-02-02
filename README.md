# Домашнее задание к занятию "Отказоустойчивость в облаке" - Еноктаев Олег


### Инструкция по выполнению домашнего задания

   1. Сделайте `fork` данного репозитория к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/git-hw или  https://github.com/имя-вашего-репозитория/7-1-ansible-hw).
   2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
   3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
      - впишите вверху название занятия и вашу фамилию и имя
      - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
      - для корректного добавления скриншотов воспользуйтесь [инструкцией "Как вставить скриншот в шаблон с решением](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
      - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции  по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
   4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
   5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
   6. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.
   
Желаем успехов в выполнении домашнего задания!
   
### Дополнительные материалы, которые могут быть полезны для выполнения задания

1. [Руководство по оформлению Markdown файлов](https://gist.github.com/Jekins/2bf2d0638163f1294637#Code)

---

### Задание 1

Возьмите за основу решение к заданию 1 из занятия «Подъём инфраструктуры в Яндекс Облаке».

1. Теперь вместо одной виртуальной машины сделайте terraform playbook, который:
- создаст 2 идентичные виртуальные машины. Используйте аргумент count для создания таких ресурсов;
- создаст таргет-группу. Поместите в неё созданные на шаге 1 виртуальные машины;
- создаст сетевой балансировщик нагрузки, который слушает на порту 80, отправляет трафик на порт 80 виртуальных машин и http healthcheck на порт 80 виртуальных машин.

Рекомендуем изучить документацию сетевого балансировщика нагрузки для того, чтобы было понятно, что вы сделали.
2. Установите на созданные виртуальные машины пакет Nginx любым удобным способом и запустите Nginx веб-сервер на порту 80.


3. Перейдите в веб-консоль Yandex Cloud и убедитесь, что:
- созданный балансировщик находится в статусе Active, 
- обе виртуальные машины в целевой группе находятся в состоянии healthy.
4. Сделайте запрос на 80 порт на внешний IP-адрес балансировщика и убедитесь, что вы получаете ответ в виде дефолтной страницы Nginx.

В качестве результата пришлите:

1. Terraform Playbook.

2. Скриншот статуса балансировщика и целевой группы.

3. Скриншот страницы, которая открылась при запросе IP-адреса балансировщика.

Terraform playbook
```
resource "yandex_vpc_network" "net" {
  name = "sminex"
}

resource "yandex_vpc_subnet" "subnet1" {
  name           = "sminex"
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.net.id
  v4_cidr_blocks = ["192.168.0.0/24"]
}

/*resource "yandex_vpc_subnet" "subnet2" {
  name           = "sminex2"
  zone           = "ru-central1-b"
  network_id     = yandex_vpc_network.net.id
  v4_cidr_blocks = ["10.0.0.0/24"]
}


/*resource "yandex_compute_instance" "devops" {
  name        = "devops"
  hostname    = "devops"
  platform_id = "standard-v1"
  zone        = "ru-central1-a"

  resources {
    cores  = 2
    memory = 1
    core_fraction = 20
  }

  boot_disk {
    initialize_params {
      image_id = "fd8hr27j6bkf6bd0tdsh"
      size = 15
      type = "network-hdd"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet1.id
    ip_address = "192.168.0.4"
    nat = true
  }

  metadata = {
    user-data          = file("./cloud-init.yml")
    serial-port-enable = 1
  }

  timeouts {
    create = "10m"
  }
}


resource "yandex_compute_instance" "devops2" {
  name        = "vpn"
  hostname    = "vpn"
  platform_id = "standard-v1"
  zone        = "ru-central1-b"

  resources {
    cores  = 2
    memory = 1
    core_fraction = 20
  }

  boot_disk {
    initialize_params {
      image_id = "fd8hr27j6bkf6bd0tdsh"
      size = 15
      type = "network-hdd"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet2.id
    ip_address = "10.0.0.27"
    nat = true
  }

  metadata = {
    user-data          = file("./cloud-init.yml")
    serial-port-enable = 1
  }

  timeouts {
    create = "10m"
  }
}

output "public_ip_devops" {
  value = yandex_compute_instance.devops.network_interface.0.nat_ip_address
}

output "public_ip_devops2" {
  value = yandex_compute_instance.devops2.network_interface.0.nat_ip_address
}
*/

resource "yandex_lb_network_load_balancer" "balancer" {
  name = "balancer"

  listener {
    name = "balancer"
    port = 80
    external_address_spec {
      ip_version = "ipv4"
    }
  }

  attached_target_group {
    target_group_id = yandex_lb_target_group.group1.id

    healthcheck {
      name = "http"
      http_options {
        port = 80
        path = "/"
      }
    }
  }
}
resource "yandex_lb_target_group" "group1" {
  name      = "group1"
  region_id = "ru-central1"

  target {
    subnet_id = yandex_vpc_subnet.subnet1.id
    address   = "192.168.0.10"  // Убедитесь, что этот IP находится в диапазоне вашей подсети
  }

  target {
    subnet_id = yandex_vpc_subnet.subnet1.id
    address   = "192.168.0.11"  // Убедитесь, что этот IP находится в диапазоне вашей подсети
  }
}

# Создание двух виртуальных машин
resource "yandex_compute_instance" "vm" {
  count = 2
  name        = "vm-${count.index}"
  hostname    = "vm-${count.index}"
  platform_id = "standard-v1"
  zone        = "ru-central1-a"
  
  boot_disk {
    initialize_params {
      image_id = "fd8gb53770b38drt7f1h" // debian 12
      size = 8
      type = "network-hdd"
    }
  }

  resources {
    cores  = 2
    memory = 1
    core_fraction = 20
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet1.id
    ip_address = "192.168.0.${10 + count.index}"  # Уникальные IP-адреса
    nat = true
  }

  metadata = {
    user-data          = file("./cloud-init.yml")
    serial-port-enable = 1
  }
}
output "vm-nat_ip_address" {
  value = yandex_compute_instance.vm[0].network_interface.0.nat_ip_address
}

output "vm-nat_ip_address2" {
  value = yandex_compute_instance.vm[1].network_interface.0.nat_ip_address
}


```

`При необходимости прикрепитe сюда скриншоты
![Название скриншота 1](ссылка на скриншот 1)`
 
Внесем данные в hosts ansible ип адреса:
vm-nat_ip_address = "158.160.32.164"
vm-nat_ip_address2 = "158.160.60.146"
 и установим nginx
```
oleg@DESKTOP-6TMQOI1:/etc/ansible$ ansible-playbook -i hosts ./playbooks/mc-deploy.yml

PLAY [nginx] *********************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************************************************************************************
ok: [nginx2]
ok: [nginx1]

TASK [update] ********************************************************************************************************************************************************************************************************************************************************
changed: [nginx1]
changed: [nginx2]

TASK [Install nginx] *************************************************************************************************************************************************************************************************************************************************
changed: [nginx1]
changed: [nginx2]

PLAY RECAP ***********************************************************************************************************************************************************************************************************************************************************
nginx1                     : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
nginx2                     : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

![balancer](https://github.com/incid3nt/otkazoustoichivost/blob/main/image/chrome_KS07YHp7XX.png)`

и видим что балансировщик работает:
```
oleg@DESKTOP-6TMQOI1:~/otkazoust$ curl http://158.160.154.224/
<!DOCTYPE html>
<html>
<head>
<title>Welcome to wm0</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to wm0!</h1>
</body>
</html>
oleg@DESKTOP-6TMQOI1:~/otkazoust$ curl http://158.160.154.224/
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to wm1</h1>
</body>
</html>
```
---

