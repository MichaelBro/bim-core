# Bitrix Migration (BIM)
[![Latest Stable Version](https://poser.pugx.org/cjp2600/bim-core/v/stable.svg)](https://packagist.org/packages/cjp2600/bim-core) [![Total Downloads](https://poser.pugx.org/cjp2600/bim-core/downloads.svg)](https://packagist.org/packages/cjp2600/bim-core) [![Latest Unstable Version](https://poser.pugx.org/cjp2600/bim-core/v/unstable.svg)](https://packagist.org/packages/cjp2600/bim-core) [![License](https://poser.pugx.org/cjp2600/bim-core/license.svg)](https://packagist.org/packages/cjp2600/bim-core)

Версионная миграция структуры БД для **[1С Битрикс CMS](http://bitrix.ru)**

# Оглавление
- [Автоматическая установка BIM](#auto)
- [Ручная установка](#hand)
- [Настройка](#prop)

# <a name="auto"></a>1.1 Автоматическая установка BIM

Для установки и инициализации bim для bitrix проекта необходимо выполнить следующиие действия из корня проекта:

- Установить Composer:
```    
curl -s https://getcomposer.org/installer | php
```
- Выполнить установочный скрипт:

``` bash
php -r "readfile('https://raw.githubusercontent.com/cjp2600/bim/master/install');" | php
```
> Автоматические действия установщика:

> 1. Добавление файла **bim** в корень проекта.
> 2. Инициализация **composer autoloader** в файле **init.php**
> 3. Создание файла **composer.json** в корне проекта со ссылкой на **bim** репозиторий **"require": { "cjp2600/bim-core": "dev-master"}**

## <a name="hand"></a>1.2 Ручная установка BIM

Для ручной установки bim необходимо:

- Установить Composer:
   
```    
curl -s https://getcomposer.org/installer | php
```

- Создать файл bim (без расширения) в корне bitrix проекта с содержимым:
```php
#!/usr/bin/php
<?php

ini_set('max_execution_time', 36000);

define("NO_KEEP_STATISTIC", true);
define("NO_AGENT_STATISTIC", true);
define("NOT_CHECK_PERMISSIONS", true);
$_SERVER["DOCUMENT_ROOT"] = dirname(__FILE__);
require($_SERVER["DOCUMENT_ROOT"] . "/bitrix/modules/main/include/prolog_before.php");

\Bim\Migration::init();

```
- Добавть инициализацию composer (в файл init.php добавить запись):

```bash
if (file_exists($_SERVER['DOCUMENT_ROOT'] . '/vendor/autoload.php'))
    require_once $_SERVER['DOCUMENT_ROOT'] . '/vendor/autoload.php';
```

- Создать в корне сайта файл **composer.json** с содержимым:

```json
{
	"require": {
		"cjp2600/bim-core": "dev-master"
	}
}
```

- В **.gitignore** добавить запись:

```
/vendor
*.lock
```


# 2 <a name="prop"></a>Настройка

Для начала работы обновляем **composer** и создаем миграционную таблицу в БД:

``` bash
php composer.phar update
```
Создаём таблицу миграций : 

```bash
php bim init
```


# 3 Развертывание [BIM UP]

- Общее применение:
```bash
php bim up
```
Применяет весь список не применённых миграционных классов отсортированых по названию (**timestamp**).

- Еденичное применение:
```bash
php bim up 1423660766
```
Применяет указанную в праметрах миграцию.

- Применение по временному периоду:
```bash
php bim up --from="29.01.2015 00:01" --to="29.01.2015 23:55"
```
- Применение по тегу:
```bash
php bim up --tag=iws-123
```
Применяет все миграции где найден указанный тег в описании.

Например:

> Description: add new migration #iws-123


# 4 Откат  [BIM DOWN]

- Общий откат:
```bash
php bim down
```
Применяет весь список применённых миграционных классов.

- Еденичный откат:
``` bash
php bim down 1423660766
```
Откатывает указанную в праметрах миграцию.

- Откат по временному периоду:
```bash
php bim down --from="29.01.2015 00:01" --to="29.01.2015 23:55"
```
- Откат по тегу:
```bash
php bim up --tag=iws-123
```
Откатывает все миграции где найден указанный тег в описании.

Например:
> Description: add new migration #iws-123

# 5 Вывод списка миграций [BIM LS]
- Общей список:
```bash
php bim ls
```
- Список применённых миграций:
```bash
php bim ls --a
```
- Список новых миграций:
```bash
php bim ls --n
```
- Список миграций за определённый период времени:
```bash
php bim ls --from="29.01.2015 00:01" --to="29.01.2015 23:55" 
```
- Список миграций по тегу:
```bash
php bim ls --tag=iws-123
```

# 6 Создание новых миграций [BIM GEN]

Существует два способа создания миграций:
 
## 1) Создание пустой миграции:
Создается пустой шаблон миграционного класса. Структура класса определена интерфейсом *Bim/Revision* и включает следующие
обязательные методы:
 
  - *up();* - метод развертывания.
  - *down();* - метод отката.
  - *getDescription();* - получения описания.
  - *getAuthor();* - получение автора.
 
Задача - развертывание и откат кастомного кода изменения схем bitrix БД.

Дополнительно запрашивается:
- [Description]
 
**Пример:**

``` bash
php bim gen
```
Также возможно передать description опционально:
``` bash  
php bim gen --d="new description #iws-123"
```

> После выполнения команды создается файл миграции вида: */[migrations_path]/[timestamp].php*

> Например: /migrations/123412434.php
 
## 2) Создание миграционного кода по наличию:

Создается код развертывания/отката существующего элемента схемы bitrix БД.
На данный момент доступно генерация по наличию для следующих элементов bitrix БД:
 
### 2.1 IblockType *( php bim gen IblockType:[add|delete] )*:

Создается Миграционный код "**Типа ИБ**" включая созданные для него *(UserFields, IBlock, IblockProperty)*
 
Дополнительно запрашивается:
- [IBLOCK_TYPE_ID]
- [Description]
 
**Пример:**

``` bash 
php bim gen IblockType:add
``` 
Также возможно передать iblock type id и description опционально:
``` bash  
php bim gen IblockType:add --typeId=catalog --d="new description #iws-123"
``` 
### 2.2 Iblock *( php bim gen Iblock:[add|delete] )*:

Создается Миграционный код "**ИБ**" включая созданные для него *(IblockProperty)*

Дополнительно запрашивается:
- [IBLOCK_CODE]
- [Description]

**Пример:**
``` bash  
php bim gen Iblock:add
``` 
Также возможно передать iblock code и description опционально:
``` bash  
php bim gen Iblock:add --code=goods --d="new description #iws-123"
``` 

### 2.3 IblockProperty *( php bim gen IblockProperty:[add|delete] )*:

Создается Миграционный код "**Свойства ИБ**"

Дополнительно запрашивается:
- [IBLOCK_CODE]
- [PROPERTY_CODE]
- [Description]

**Пример:**
``` bash  
php bim gen IblockProperty:add
``` 
Также возможно передать iblock code, property code и description опционально:
``` bash  
php bim gen IblockProperty:add --code=goods --propertyCode=NEW_ITEM --d="new description #iws-123"
``` 

### 2.4 Hlblock *( php bim gen Hlblock:[add|delete] )*:

Создается Миграционный код "**Highloadblock**" включая созданные для него *(UserFields)*

Дополнительно запрашивается:
- [HLBLOCK_ID]
- [Description]

**Пример:**
``` bash  
php bim gen Hlblock:add
``` 
Также возможно передать hlblock id и description опционально:
``` bash  
php bim gen IHlblock:add --id=82 --d="new description #iws-123"
``` 



> Обратите внимание! что миграционные классы созданные по наличию, применяются автоматически.
 
