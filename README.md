# Домашнее задание к занятию «Введение в Terraform» - `Гилельс К.M`

### Цели задания

1. Установить и настроить Terrafrom.
2. Научиться использовать готовый код.

------

### Чек-лист готовности к домашнему заданию

1. Скачайте и установите **Terraform** версии >=1.12.0 . Приложите скриншот вывода команды ```terraform --version```.
2. Скачайте на свой ПК этот git-репозиторий. Исходный код для выполнения задания расположен в директории **01/src**.
3. Убедитесь, что в вашей ОС установлен docker.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. Репозиторий с ссылкой на зеркало для установки и настройки Terraform: [ссылка](https://github.com/netology-code/devops-materials).
2. Установка docker: [ссылка](https://docs.docker.com/engine/install/ubuntu/). 
------
### Внимание!! Обязательно предоставляем на проверку получившийся код в виде ссылки на ваш github-репозиторий!
------

### Задание 1

1. Перейдите в каталог [**src**](https://github.com/netology-code/ter-homeworks/tree/main/01/src). Скачайте все необходимые зависимости, использованные в проекте. 
2. Изучите файл **.gitignore**. В каком terraform-файле, согласно этому .gitignore, допустимо сохранить личную, секретную информацию?(логины,пароли,ключи,токены итд)
3. Выполните код проекта. Найдите  в state-файле секретное содержимое созданного ресурса **random_password**, пришлите в качестве ответа конкретный ключ и его значение.
4. Раскомментируйте блок кода, примерно расположенный на строчках 29–42 файла **main.tf**.
Выполните команду ```terraform validate```. Объясните, в чём заключаются намеренно допущенные ошибки. Исправьте их.
5. Выполните код. В качестве ответа приложите: исправленный фрагмент кода и вывод команды ```docker ps```.
6. Замените имя docker-контейнера в блоке кода на ```hello_world```. Не перепутайте имя контейнера и имя образа. Мы всё ещё продолжаем использовать name = "nginx:latest". Выполните команду ```terraform apply -auto-approve```.
Объясните своими словами, в чём может быть опасность применения ключа  ```-auto-approve```. Догадайтесь или нагуглите зачем может пригодиться данный ключ? В качестве ответа дополнительно приложите вывод команды ```docker ps```.
8. Уничтожьте созданные ресурсы с помощью **terraform**. Убедитесь, что все ресурсы удалены. Приложите содержимое файла **terraform.tfstate**. 
9. Объясните, почему при этом не был удалён docker-образ **nginx:latest**. Ответ **ОБЯЗАТЕЛЬНО НАЙДИТЕ В ПРЕДОСТАВЛЕННОМ КОДЕ**, а затем **ОБЯЗАТЕЛЬНО ПОДКРЕПИТЕ** строчкой из документации [**terraform провайдера docker**](https://library.tf/providers/kreuzwerker/docker/latest).  (ищите в классификаторе resource docker_image )

### Решение

Чеклист:
Установлены актуальные версии `docker` (все контейнеры с прошлых дз выключены), `terraform` (архивом с зеркала яндекс) и склонирован репозитоий 
`https://github.com/netology-code/ter-homeworks.git`

![0](/0.png)

Были внесены первичные изменения в `main.tf`:
параметр `required_version = "~>1.12.0"` был заменен на `required_version = ">=1.12.0"` так как установлен `Terraform v1.15.6`.

Так как работаю из под root файл `.terraformrc` был скопирован в `~/.terraformrc`

1. `cd ter-homeworks/01/src/ && terraform init`

![1](/1.png)

2. В файле `.gitignore` явно указано:
```
# own secret vars store.
personal.auto.tfvars
```
В каком terraform-файле, согласно этому `.gitignore`, допустимо сохранить личную, секретную информацию?(логины,пароли,ключи,токены итд) -- 
допустимо хранить в файле `personal.auto.tfvars`, так как он указан в `.gitignore` и не должен попадать в Git-репозиторий.

3. Была выполнена команда `terraform apply`, с помощью поиска был обнаружен random password.

![3](/3.png)

![3.1](/3.1.png)

пришлите в качестве ответа конкретный ключ и его значение: `"result": "ldJ814E82fqZLvzh"`

4. Выполнена команда `terraform validate`

![4](/4.png)

Первично найдено две ошибки, отсутствие локального имени у докер образа `All resource blocks must have 2 labels (type, name).` и 
начало имени докер контейнера с цифры `A name must start with a letter or underscore and may contain only letters, digits, underscores, and dashes.`
После исправления этих ошибок повторная валидация выдала:

![4.1](/4.1.png)

Незадикларированы ресурсы + проблемы с регистром `A managed resource "random_password" "random_string_FAKE" has not been declared in the root module.`

Исправленный блок кода:
```tf
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
    }
  }
  required_version = ">=1.12.0" /* Многострочный комментарий.
 Требуемая версия terraform */
}
provider "docker" {}

#однострочный комментарий

resource "random_password" "random_string" {
  length      = 16
  special     = false
  min_upper   = 1
  min_lower   = 1
  min_numeric = 1
}


resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = true
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "example_${random_password.random_string.result}"

  ports {
    internal = 80
    external = 9090
  }
}
```

![4.2](/4.2.png)


5. Выполнил процедуру `terraform apply` + `docker ps`

![5](/5.png)

6. Заменил имя контейнера с "example_${random_password.random_string.result}" на "hello_world"
и выполнил `terraform apply -auto-approve`
Флаг или ключ -auto-approve как и переводится авто разрешение, не требует подтверждения от человека, его опасность в общем то такая же как и любого рода испоьзование ключа -f (force), чревато потерями данных, сбоями и прочими нестыковками при ошибках в коде или скрипте.
Подходит для тестовой среды при массовом развертывании, в учебных целях или на уже отлаженных проверенных процессах.

![6](/6.png)

7. удаляем ресурсы с помощью `terraform destroy`

![7](/7.png)

![7.1](/7.1.png)

8. Причина по которой не удаляется образ это параметр `keep_locally = true` в `main.tf`

![8](/8.png)

В документации сказано "If true, then the Docker image won't be deleted on destroy operation." следовательно образ останется в локальном хранилище образов docker



