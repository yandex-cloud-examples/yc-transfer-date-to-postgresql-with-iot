# Пример передачи данных от устройств через Yandex IoT Core в Managed PostgreSQL с помощью Yandex Cloud Functions

В данном примере вы организуете запись данных с устройств [Yandex IoT Core](https://yandex.cloud/ru/docs/iot-core/) с помощью функции [Yandex Cloud Functions](https://yandex.cloud/ru/docs/functions/). Для хранения данных будет использоваться кластер [Yandex Managed Service for PostgreSQL](https://yandex.cloud/ru/docs/managed-postgresql/).

В качестве примера, используется сигнал датчика воздуха, измеряющего следующие параметры:
* Температура.
* Влажность.
* Давление.
* Уровень содержания СO2.

Датчик выдает результат в формате JSON. Например:

```json
{
"DeviceId":"0e3ce1d0-1504-4325-972f-55c961319814",
"TimeStamp":"2020-05-21T22:53:16Z",
"Values":[
    {"Type":"Float","Name":"Humidity","Value":"25.281837"},
    {"Type":"Float","Name":"CarbonDioxide","Value":"67.96608"},
    {"Type":"Float","Name":"Pressure","Value":"110.7021"},
    {"Type":"Float","Name":"Temperature","Value":"127.708824"}
    ]
}
```


## Создание инфраструктуры

Пример содержит файл `iotadapter.py` с кодом функции Yandex Cloud Functions и файлы конфигурации Terraform для создания инфраструктуры.

[Terraform](https://www.terraform.io/) позволяет быстро создать облачную инфраструктуру в Yandex Cloud и управлять ею с помощью файлов конфигураций. В файлах конфигураций хранится описание инфраструктуры на языке HCL (HashiCorp Configuration Language). Terraform и его провайдеры распространяются под лицензией [Business Source License](https://github.com/hashicorp/terraform/blob/main/LICENSE).

Подробную информацию о ресурсах провайдера смотрите в документации на сайте [Terraform](https://www.terraform.io/docs/providers/yandex/index.html) или в [зеркале](https://terraform-provider.yandexcloud.net/).

При изменении файлов конфигураций Terraform автоматически определяет, какая часть вашей конфигурации уже развернута, что следует добавить или удалить.

Чтобы создать инфраструктуру для записи информации с устройства в базу данных с помощью Terraform:

1. [Установите Terraform](https://cloud.yandex.ru/docs/tutorials/infrastructure-management/terraform-quickstart#install-terraform), [получите данные для аутентификации](https://cloud.yandex.ru/docs/tutorials/infrastructure-management/terraform-quickstart#get-credentials) и укажите источник для установки провайдера Yandex Cloud (раздел [Настройте провайдер](https://cloud.yandex.ru/docs/tutorials/infrastructure-management/terraform-quickstart#configure-provider), шаг 1).

1. Склонируйте репозиторий с конфигурационными файлами.

1. В файле `terraform.auto.tfvars` задайте пользовательские параметры:
    * `yc_cloud_id` — [идентификатор облака](https://cloud.yandex.ru/docs/resource-manager/operations/cloud/get-id).
    * `yc_folder_id` — [идентификатор каталога](https://cloud.yandex.ru/docs/resource-manager/operations/folder/get-id).
    * `yc_main_zone` — [зона доступности](https://yandex.cloud/ru/docs/overview/concepts/geo-scope).

1. Создайте ресурсы:

    1. В терминале перейдите в папку, где вы отредактировали конфигурационный файл.

    1. Проверьте корректность конфигурационного файла с помощью команды:

        ```bash
        terraform validate
        ```
        
        Если конфигурация является корректной, появится сообщение:

        ```text
        Success! The configuration is valid.
        ```

    1. Выполните команду:

        ```bash
        terraform plan
        ```
        
        В терминале будет выведен список ресурсов с параметрами. На этом этапе изменения не будут внесены. Если в конфигурации есть ошибки, Terraform на них укажет.

    1. Примените изменения конфигурации:

        ```bash
        terraform apply
        ```

    1. Подтвердите изменения: введите в терминале слово `yes` и нажмите **Enter**.

После этого в указанном каталоге будут созданы все требуемые ресурсы. 

В терминале отобразятся идентификаторы и пароли созданных IoT устройств, реестра, кластера БД.
