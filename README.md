# Стандарты программирования для платформы 1С-Битрикс

В этом руководстве описаны рекомендуемые техники при разработке под платформу 1С-Битрикс (далее просто Битрикс).

Весь PHP код должен написан в соответствии со стандартом [PSR-1](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-1-basic-coding-standard.md). Стиль написания кода должен соответствовать стандарту [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md).

## Общие положения

Часть описанных в этом разделе требований уже указана в стандартах, приведенных выше, но отметим особенно важные.
- РЕКОМЕНДУЕТСЯ использовать только `<?php` и `<?=` теги. Тег `<?` использовать НЕ РЕКОМЕНДУЕТСЯ.
- РЕКОМЕНДУЕТСЯ использование кодировки UTF-8 без BOM.
- Используйте Tab для каждого отступа. Никаких пробелов для блоков кода.

## Отладка

### Вывод на экран

* Воспользуйтесь модулем [Bitrix Debug](http://marketplace.1c-bitrix.ru/solutions/scrollup.bxd/), [GitHub](https://github.com/ancorp/bitrix-debug).

### Вывод в файл

* [api_help](http://dev.1c-bitrix.ru/api_help/main/functions/debug/index.php)

1. Определите константу `LOG_FILENAME` в файле `/bitrix/php_interface/dbconn.php`, задавая путь к лог-файлу за пределами `DOCUMENT_ROOT`.

```php
// определяем константу LOG_FILENAME, в которой зададим путь к лог-файлу
define('LOG_FILENAME', $_SERVER['DOCUMENT_ROOT'] . '/_main.log');
```

2. Отправьте сообщение в лог
	
```php
AddMessage2Log('Произвольный текст сообщения', 'module_id');
```

Пример:

```php
AddMessage2Log( print_r($arResult, true) );
```

### SQL

* Определите переменную `DBDebugToFile` для логирования всех SQL запросов.

```php
$DBDebugToFile = true;
```

## Базовые правила при разработке под Битрикс

- при добавлении кода в файл init.php РЕКОМЕНДУЕТСЯ выносить логически сгруппированный код в отдельные файлы и подключать их внутри init.php

```php
// SomeClass
require_once __DIR__ . '/include/some_class.php';
```

- НЕ РЕКОМЕНДУЕТСЯ использовать цифровые значения в GetList, GetByID и схожих методах, которые принимают различные ID. РЕКОМЕНДУЕТСЯ создать файл со всеми необходимыми константами и вызывать их имена. У каждой константы ДОЛЖНО быть «говорящее» именование и комментарий.

**Не правильно**:
```php
$comments = CIBlockElement::GetList(array(), array('IBLOCK_ID' => 12));
```

**Правильно**:
Создаем файл constants.php и указываем в нем:
```php
//ИБ с комментариями пользователей
const COMMENTS_IBLOCK_ID = 12;
```
Подключаем этот файл в init.php
```php
//Константы проекта
require_once __DIR__ . '/includes/constants.php';
```
Используем константу
```php
$comments = CIBlockElement::GetList(array(), array('IBLOCK_ID' => COMMENTS_IBLOCK_ID));
```
- при выборках данных (например, [GetList](http://dev.1c-bitrix.ru/api_help/iblock/classes/ciblockelement/getlist.php)) ОБЯЗАТЕЛЬНО указывать поля (для GetList это 5-ый параметр arSelectFields), которые нужны для дальнейших манипуляций, кроме случаев, когда нужны все поля;

```php
$filter = array('IBLOCK_ID' => COMMENTS_IBLOCK_ID);
$select = array('ID', 'NAME');
$comments = CIBlockElement::GetList(array(), $filter, false, array(), $select);
```
- при необходмости выбрать несколько элементов по `ID`, ОБЯЗАТЕЛЬНО использовать `GetList` вместо `GetByID`

**Не правильно**:
```php
$element1 = CIBlockElement::GetByID(1);
$element2 = CIBlockElement::GetByID(2);
```

**Правильно**:
```php
$elements = CIBlockElement::GetList(array(), array(1, 2));
```
- НЕ РЕКОМЕНДУЕТСЯ использовать прямые запросы к базе данных без крайней необходимости;
- если к файлу не предусмотрен прямой доступ ОБЯЗАТЕЛЬНО в первой строке файла добавить

```php
if (!defined('B_PROLOG_INCLUDED') || B_PROLOG_INCLUDED !== true) {
	die();
}
```

## Работа с компонентами

- РЕКОМЕНДУЕТСЯ давать шаблонам компонентов осмысленные названия и в каждом проекте придерживаться общего стиля. Например, `Раздел/страница_сайта.Название.Тип`
Примеры:
 - `index.user.auth`
 - `profile.orders.list`
 - `cart.products.additional`
- НЕЛЬЗЯ модифицировать стандартные компоненты. Если возникает такая необходимость — создается копия компонента в своем пространстве имен в папке `/bitrix/components/`
- РЕКОММЕНДУЕТСЯ все шаблоны компонентов сохранять в шаблоне `.default` в папке `/bitrix/templates/.default/`
- НЕ РЕКОМЕНДУЕТСЯ делать любые манипуляции с данными в файле `template.php`. При необходимости правки логики стандартных компонентов, но недостаточной для того, что делать свой используются файлы [result_modifier.php](http://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=43&LESSON_ID=2830&LESSON_PATH=3913.4565.2830) и [component_epilog.php](http://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=43&LESSON_ID=2975&LESSON_PATH=3913.4565.2975)
- РЕКОМЕНДУЕТСЯ использовать файлы `style.css` и `script.js` в шаблонах только если они переопределяют стандартное поведение схожих элементов. РЕКОМЕНДУЕТСЯ оставить комментарий об этой особенности

## Работа с шаблонами

- РЕКОМЕНДУЕТСЯ использовать минимальное количество шаблонов
- РЕКОМЕНДУЕТСЯ подключать `header.php` и `footer.php` из одного места для всех шаблонов, если это позволяет дизайн и верстка
- РЕКОМЕНДУЕТСЯ общие картинки, скрипты и стили шаблонов сохранять в одном месте, например, в `/bitrix/templates/.default/`

## Работа с инфоблоками

- ОБЯЗАТЕЛЬНО называть все свойства инфоблоков в верхнем регистре, осмысленно (используя связку сущность-наименование), разделяя слова нижним подчеркиванием. Например:
 - Имя пользователя: USER_NAME
 - Валюта заказа: ORDER_CURRENCY
 - Список заказов: ORDER_LIST
- Использовать уникальные `CODE` для каждого отдельного инфо-блока. Рекомендуется поставить защиты:

```php
AddEventHandler('iblock', 'OnBeforeIBlockAdd', 'checkCode');
AddEventHandler('iblock', 'OnBeforeIBlockUpdate', 'checkCode');
function checkCode($params)
{
	if (isset($params['CODE']) && $params['CODE'] !== '') {
		CModule::IncludeModule('iblock');

		$iblocks = CIBlock::GetList(array(), array('CODE' => $params['CODE']));
		if ($iblock = $iblocks->Fetch()) {
			$APPLICATION->ThrowException(new CApplicationException('Инфоблок с таким CODE уже существует.'));

			return false;
		}
	}
}
```

- Рекомендуется использовать [библиотеку](https://github.com/xescoder/bitrix-iblock-tools) для простого и быстрого получения ID.

## AJAX-обработчики

Довольно часто нужно создать пустую страницу — без вывода шапки и футера, но с возможностью обращаться к классам Битрикс. Например, такое нужно для работы с AJAX. Делается это очень просто. Создаем файл и пишем в нем.

```php
define('NO_KEEP_STATISTIC', true);
define('NOT_CHECK_PERMISSIONS', true);
require $_SERVER['DOCUMENT_ROOT'] . '/bitrix/modules/main/include/prolog_before.php';
```

Теперь в этом файле можно легко обращаться к любым классам Битрикс. Не забывайте только модули подключить нужные:

```php
CModule::IncludeModule('название_модуля');
```

[Источник](http://olegorestov.ru/this/create_an_empty_page_with_access_to_the_api_bitrix/)