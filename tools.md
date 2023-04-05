# Установка и настройка рабочих инструментов с нуля

Краткое перечисление необходимых операций для того, чтобы развернуть рабочую среду с нуля.

Для работы понадобится:
- Yandex Cloud CLI
- AWS CLI
- S3cmd
- ...

## Создание сервисного аккаунта в Yandex Cloud

[Создание сервисного аккаунта](https://cloud.yandex.ru/docs/iam/operations/sa/create)

[Назначение роли сервисному аккаунту](https://cloud.yandex.ru/docs/iam/operations/sa/assign-role-for-sa)

## Создание статического ключа доступа

[Создание статических ключей](https://cloud.yandex.ru/docs/iam/operations/sa/create-access-key)

## yc

### Документация

[yc]()

### Brief
#### Установка
```
brew install yandex-cloud-cli
```

1. Получить OAuth-токен по [cсылке](https://oauth.yandex.ru/authorize?response_type=token&client_id=1a6990aa636648e9b2ef855fa7bec2fb)
2. Выполнить `yc init`

Можно создать несколько профилей: [Создание профиля](https://cloud.yandex.ru/docs/cli/operations/profile/profile-create)

## aws-cli

### Документация
[AWS Command Line Interface](https://cloud.yandex.ru/docs/storage/tools/aws-cli)

### Brief
#### Установка
1. Создать статический ключ доступа для сервисного аккаунта
```
yc iam service-account list
```

```
yc iam access-key create --service-account-name <имя> \
--description "<описание>"
```

2. Конфигурация
```
aws configure
```

`AWS Access Key ID` — идентификатор статического ключа
`AWS Secret Access Key` — содержимое статического ключа
`Default region name` — всегда `ru-central1`

Команда создает файлы `.aws/credentials`:
```
[default]
  aws_access_key_id = id
  aws_secret_access_key = secretKey
```

`.aws/config`:
```
[default]
  region=ru-central1
```

3. Добавить алиас для удобства:
```
alias ycs3='aws s3 --endpoint-url=https://storage.yandexcloud.net'
```

#### Примеры операций

##### Создать бакет
```
aws --endpoint-url=https://storage.yandexcloud.net s3 mb s3://bucket-name
```

##### Загрузить объекты
```
aws --endpoint-url=https://storage.yandexcloud.net \
  s3 cp --recursive local_files/ s3://bucket-name/path_style_prefix/
```

##### Получить список объектов
```
aws --endpoint-url=https://storage.yandexcloud.net \
  s3 ls --recursive s3://bucket-name
```

##### Удалить объекты

###### Удалить все объекты с заданным префиксом
```
aws --endpoint-url=https://storage.yandexcloud.net \
  s3 rm s3://bucket-name/path_style_prefix/ --recursive
```

###### Удалить объекты в фильтре `--include` и пропустить объекты из фильтра `--exclude`
```
aws --endpoint-url=https://storage.yandexcloud.net \
  s3 rm s3://bucket-name/path_style_prefix/ --recursive
```

###### Удалить объекты по одному
```
aws --endpoint-url=https://storage.yandexcloud.net \
  s3 rm s3://bucket-name/path_style_prefix/textfile.txt
```

###### Получить объект
```
aws --endpoint-url=https://storage.yandexcloud.net \
  s3 cp s3://bucket-name/textfile.txt textfile.txt
```

## S3cmd

### Документация
[S3cmd](https://cloud.yandex.ru/docs/storage/tools/s3cmd)

### Brief

#### Установка
```
brew install s3cmd
```

#### Настройка
```
s3cmd --configure
```

`Access Key` — идентификатор статического ключа
`Secret Key` — секретный статический ключ
`Default Region` — всегда `ru-central1`
`S3 Endpoint` — `storage.yandexcloud.net`
`DNS-style bucket+hostname:port template for accessing a bucket` — `%(bucket)s.storage.yandexcloud.net`
Остальные параметры — по умолчанию

Команда создает файл `~/.s3cfg`:
```
[default]
access_key = id
secret_key = secretKey
bucket_location = ru-central1
host_base = storage.yandexcloud.net
host_bucket = %(bucket)s.storage.yandexcloud.net
```

Для корректной работы команд для управления хостингом статических сайтов, нужно вручную добавить в конфиг:
```
website_endpoint = http://%(bucket)s.website.yandexcloud.net
```

По умолчанию при загрузке инструмент добавляет метаинформацию (права доступа, владельцы файла, время создания, изменения, доступа). Чтобы отключить это поведение:
- добавить в `~/.s3cfg` параметр `preserve_attrs = False`
- или ключ `--no-preserve` при запуске команды

#### Примеры операций

##### Получить список бакетов
```
s3cmd ls
```

##### Создать бакет
```
s3cmd mb s3://bucket
```

##### Загрузить объект в холодное хранилище
```
s3cmd --storage-class COLD put local_files s3://bucket/object
```

##### Получить список объектов
```
s3cmd ls s3://bucket
```

##### Получить объект
```
s3cmd get s3://bucket/object local_file
```

##### Удалить объект
```
s3cmd del s3://bucket/object
```