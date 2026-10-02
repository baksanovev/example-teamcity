# Домашнее задание «TeamCity»

## Подготовка инфраструктуры

Для выполнения задания были развернуты:

- TeamCity Server;
- TeamCity Agent;
- Nexus Repository.

TeamCity Agent подключён и авторизован на сервере TeamCity.

## Репозиторий

Для выполнения задания создан fork репозитория:

https://github.com/baksanovev/example-teamcity

В TeamCity создан проект, подключённый к данному репозиторию.

## Nexus

В Nexus используется Maven-репозиторий `maven-releases`.

В `pom.xml` настроена публикация Maven-артефактов:

```xml
<distributionManagement>
    <repository>
        <id>nexus</id>
        <url>http://10.128.0.18:8081/repository/maven-releases/</url>
    </repository>
</distributionManagement>
```

Учётные данные Nexus передаются в TeamCity через параметры окружения. Пароль хранится как защищённый параметр TeamCity.

## Настройка сборки TeamCity

Для основной ветки `master` используется Maven:

```text
clean deploy
```

Для остальных веток используется:

```text
clean test
```

Условие для `master`:

```text
teamcity.build.branch.is_default equals true
```

Условие для остальных веток:

```text
teamcity.build.branch.is_default does not equal true
```

## Feature branch

Для выполнения задания была создана ветка:

```text
feature-add-reply
```

В класс `Welcomer` добавлен метод:

```java
public String sayHunter() {
    return "Good luck, hunter!";
}
```

Для метода добавлена проверка в `WelcomerTest`:

```java
assertThat(welcomer.sayHunter(), containsString("hunter"));
```

Сборка feature-ветки в TeamCity успешно прошла, все 5 тестов выполнены успешно.

## Pull Request

Изменения были объединены с `master` через Pull Request:

https://github.com/baksanovev/example-teamcity/pull/1

PR `Add hunter reply` успешно объединён с веткой `master`.

## Сборка master и публикация в Nexus

После merge была выполнена успешная сборка основной ветки.

Итоговая версия Maven-артефакта:

```text
org.netology:plaindoll:0.0.4
```

Артефакт `plaindoll-0.0.4.jar` успешно опубликован в Nexus в репозиторий `maven-releases`.

## Artifacts TeamCity

В Build Configuration настроено сохранение JAR-файлов:

```text
target/*.jar
```

Успешная сборка `Build #8` сохранила JAR-файл во вкладке `Artifacts` TeamCity:

```text
plaindoll-0.0.4.jar
```

## Versioned Settings

Конфигурация TeamCity сохранена в репозитории в формате Kotlin DSL:

```text
.teamcity/
├── pluginData/
├── pom.xml
└── settings.kts
```

В `settings.kts` присутствует правило публикации артефактов:

```kotlin
artifactRules = "target/*.jar"
```

Для основной ветки сохранён Maven-шаг:

```kotlin
maven {
    conditions {
        equals("teamcity.build.branch.is_default", "true")
    }
    goals = "clean deploy"
    userSettingsSelection = "nexus-settings"
}
```

Для остальных веток сохранён отдельный Maven-шаг:

```kotlin
maven {
    name = "Test"

    conditions {
        doesNotEqual("teamcity.build.branch.is_default", "true")
    }
    goals = "clean test"
}
```

Пароль Nexus в открытом виде в репозиторий не добавлен. TeamCity использует защищённый credential.

## Результат

В ходе выполнения задания:

- настроены TeamCity Server и TeamCity Agent;
- настроен Nexus Repository;
- TeamCity подключён к GitHub-репозиторию;
- настроена Maven-сборка;
- настроена публикация артефактов в Nexus;
- настроена отдельная логика сборки `master` и feature-веток;
- добавлен новый метод и тест;
- feature-ветка успешно протестирована;
- изменения объединены через Pull Request;
- `master` успешно собран;
- JAR опубликован в Nexus;
- JAR сохранён в Artifacts TeamCity;
- конфигурация TeamCity сохранена в `.teamcity` в формате Kotlin DSL.
