# Домашнее задание к занятию «Введение в Terraform»

## Решение к Задаче 1

>Изучите файл **.gitignore**. В каком terraform файле согласно этому **.gitignore** допустимо сохранить личную, секретную информацию?

Личную или секретную информацию можно хранить в любом файле указанным в **.gitignore**. На то и нужен **.gitignore** чтобы файлы содержащие чувствительную информацию не попали в открытый репозиторий.

>Изучите файл **.gitignore**. В каком terraform файле допустимо сохранить личную, секретную информацию?

```bash
# own secret vars store.
personal.auto.tfvars
```

>Выполните код проекта. Найдите  в State-файле секретное содержимое созданного ресурса **random_password**, пришлите в качестве ответа конкретный ключ и его значение.

```bash
"result": "c18AUA5V5vwly0Tx"
```

```bash
{
  "version": 4,
  "terraform_version": "1.4.0",
  "serial": 1,
  "lineage": "60aba224-e859-edb1-147b-f67214c923ec",
  "outputs": {},
  "resources": [
    {
      "mode": "managed",
      "type": "random_password",
      "name": "random_string",
      "provider": "provider[\"registry.terraform.io/hashicorp/random\"]",
      "instances": [
        {
          "schema_version": 3,
          "attributes": {
            "bcrypt_hash": "$2a$10$1G9MDOrm4fuyxrtcwRLR8.iOwIM3p4WfbrdPTUE.u5GewkrsFSmjW",
            "id": "none",
            "keepers": null,
            "length": 16,
            "lower": true,
            "min_lower": 1,
            "min_numeric": 1,
            "min_special": 0,
            "min_upper": 1,
            "number": true,
            "numeric": true,
            "override_special": null,
            "result": "c18AUA5V5vwly0Tx",
            "special": false,
            "upper": true
          },
          "sensitive_attributes": []
        }
      ]
    }
  ],
  "check_results": null
}

```

>Раскомментируйте блок кода, примерно расположенный на строчках 29-42 файла **main.tf**.  
>Выполните команду ```terraform validate```.  
>Объясните в чем заключаются намеренно допущенные ошибки? Исправьте их.

```bash
vagrant@VM2:/src$ terraform validate
╷
│ Error: Missing name for resource
│
│   on main.tf line 23, in resource "docker_image":
│   23: resource "docker_image" {
│
│ All resource blocks must have 2 labels (type, name).
╵
╷
│ Error: Invalid resource name
│
│   on main.tf line 28, in resource "docker_container" "1nginx":
│   28: resource "docker_container" "1nginx" {
│
│ A name must start with a letter or underscore and may contain only letters, digits, underscores, and dashes.

vagrant@VM2:/src$ terraform validate

│ Error: Reference to undeclared resource
│
│   on main.tf line 30, in resource "docker_container" "nginx":
│   30:   name  = "example_${random_password.random_string_FAKE.resulT}"
│
│ A managed resource "random_password" "random_string_FAKE" has not been declared in the root module.
╵
vagrant@VM2:/src$ terraform validate
╷
│ Error: Unsupported attribute
│
│   on main.tf line 30, in resource "docker_container" "nginx":
│   30:   name  = "example_${random_password.random_string.resulT}"
│
│ This object has no argument, nested block, or exported attribute named "resulT". Did you mean "result"?
```

Ошибка в 23 строке **main.tf**: Все блоки ресурсов должны иметь 2 метки. Исправление: Добавляем имя "nginx"

Правильный вариант блока кода:

```hcl
resource "docker_image" "nginx" {
  name = "nginx:latest"
  keep_locally = true
}
```

Ошибка в 28 строке **main.tf**: Имя должно начинаться с буквы или символа подчеркивания и может содержать только буквы, цифры, знаки подчеркивания и тире. Исправление: исправляем имя "1nginx" на "nginx"

Ошибка в 30 строке **main.tf**: Управляемый ресурс "random_password" "random_string_FAKE" не был объявлен в корневом модуле. Исправление: исправляем "random_string_FAKE" на "random_string"

Ошибка в 30 строке **main.tf**: Объект не имеет аргумента, вложенного блока или экспортируемого атрибута с именем "resulT". Исправление: исправляем "resulT" на "result"

Правильный вариант блока кода:

```hcl
resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "example_${random_password.random_string.result}"

...

}
```

>Выполните код. В качестве ответа приложите вывод команды ```docker ps```

```bash
vagrant@VM2:/src$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS          PORTS                  NAMES
85118e22f9bb   89da1fb6dcb9   "/docker-entrypoint.…"   About a minute ago   Up 58 seconds   0.0.0.0:8000->80/tcp   example_t3GWKTLRAxlGUh6Q
```

>Замените имя docker-контейнера в блоке кода на ```hello_world```, выполните команду ```terraform apply -auto-approve```.
>Объясните своими словами, в чем может быть опасность применения ключа  ```-auto-approve``` ?

Использование ```-auto-approve``` флага является опасным, поскольку некоторые ошибки в **main.tf** могут привести к необратимой потере данных или уничтожению базы данных. Эти ошибки будут применены без запроса одобрения.

>В качестве ответа дополнительно приложите вывод команды ```docker ps```

```bash
vagrant@VM2:/src$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                  NAMES
87f8db035a48   89da1fb6dcb9   "/docker-entrypoint.…"   18 seconds ago   Up 13 seconds   0.0.0.0:8000->80/tcp   hello_world_t3GWKTLRAxlGUh6Q
```

>Уничтожьте созданные ресурсы с помощью **terraform**. Убедитесь, что все ресурсы удалены. Приложите содержимое файла **terraform.tfstate**.

```hcl
{
  "version": 4,
  "terraform_version": "1.5.3",
  "serial": 11,
  "lineage": "8c1da195-b693-008d-a92b-db267c48defd",
  "outputs": {},
  "resources": [],
  "check_results": null
}
```

>Объясните, почему при этом не был удалён docker-образ nginx:latest. Ответ обязательно подкрепите строчкой из документации terraform провайдера docker. (ищите в классификаторе resource docker_image )

```bash
resource "docker_image" "nginx-last" {
  name         = "nginx:latest"
  keep_locally = true
}
```
```text
Команда keep_locally = true сохранила образ в локальные образы Docker.
```
