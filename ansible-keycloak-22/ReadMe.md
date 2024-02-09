## Применить плейбук на группу хостов(test): 
```bash
$ ansible-playbook keycloak.yml \
-l test \
--extra-vars postgresql_password=***
```

При прогоне плейбука будут сгенерированы файлы `/etc/hosts` и `standalone-ha.xml` на основе тех хостов, которые добавлены в соответствующую группу, поэтому ВАЖНО(!!!) применять плейбук сразу на всю группу.

## Список переменных:
- postgresql_password - пароль от БД

## Список тегов:
- os - подготовка операционной системы(установка пакетов, обновление параметров ядра, создание пользователя/группы)
- keycloak - установка keycloak, добавление системной службы

Пример использования тега:
```shell
$ ansible-playbook keycloak.yml \
-l test \
-t os \
--extra-vars postgresql_password=***
```

## Принципы формирования параметров в `keycloak.conf`
- Названия переменных состоят из слов, разделенных дефисами
- Каждое слово - наименование раздела и подраздела в xml файле
- Если конфиг находится в properties файле - для его перезаписи в keycloak.conf точки заменены на дефисы
- Камел кейс переводится в конфиг похожим образом: все заглавные буквы заменяются на прописные, слова отделяются дефисами

Пример:
Конфиг из xml файла
```xml
            <spi name="tokenMessageService">
                <provider name="default" enabled="true">
                    <properties>
                        <property name="phoneCodeExpiresInSeconds" value="240"/>
		            </properties>
                </provider>
            </spi>
```
будет обозначен в keycloak.conf как
```properties
spi-token-message-service-default-phone-code-expires-in-seconds=240
```

## Заметки:
- Необходимо скачать файл `newton-module-4.0.0-RELEASE.jar` и положить его в директорию `./files/`
- Необходимо скачать файл [keycloak-metrics-spi-4.0.0.jar](https://github.com/aerogear/keycloak-metrics-spi/releases/download/3.0.0/keycloak-metrics-spi-3.0.0.jar) и положить его в директорию `./files/`

Чтобы добавить хост в кластер, необходимо прописать хост в файл `hosts.ini` и прогнать плейбук:
```bash
$ ansible-playbook keycloak.yml \
-l test \
-t os \
-t keycloak \
--extra-vars postgresql_password=***
```

Для успешного старта на БД от старой версии необходимо обновить чексуммы ченджсетов.
Для этого выполнить скрипт `./files/update-script-kc-18.sql` 

**Внимание!!! Изменения несовместимы с keycloak v15. Старая версия серера будет работать, но не сможет свалидировать БД при старте, поэтому невозможно будет рестартануть старый сервер.**
