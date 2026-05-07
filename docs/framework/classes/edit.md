# editTable

## Общее описание
Класс `editTable` представляет собой компонент для создания и управления формами редактирования данных в административной панели. 
Основная роль класса заключается в построении интерфейса для просмотра, редактирования и сохранения записей из базы данных 
с поддержкой различных типов полей, валидации, группировки и пользовательских настроек. 


## Список методов (оглавление)
- [__construct()](#__construct) — инициализация формы редактирования
- [setTemplate()](#settemplate) — установка HTML-шаблона формы
- [setTemplateControl()](#settemplatecontrol) — настройка шаблонов элементов управления
- [setReadonly()](#setreadonly) — установка режима только для чтения
- [setReadonlyFields()](#setreadonlyfields) — установка полей только для чтения
- [addReadonlyFields()](#addreadonlyfields) — добавление полей только для чтения
- [clearReadonlyFields()](#clearreadonlyfields) — очистка полей только для чтения
- [addControl()](#addcontrol) — добавление нового элемента управления
- [addGroup()](#addgroup) — добавление группы элементов
- [addButton()](#addbutton) — добавление кнопки
- [addButtonSwitch()](#addbuttonswitch) — добавление переключателя (switch)
- [addButtonCustom()](#addbuttoncustom) — добавление произвольной кнопки
- [setSessFormField()](#setsessformfield) — сохранение значения в служебных полях
- [setLeaveChecking()](#setleavechecking) — настройка проверки изменений при уходе со страницы
- [setTable()](#settable) — установка таблицы базы данных
- [setWidthLabels()](#setwidthlabels) — установка ширины меток полей
- [setData()](#setdata) — установка данных записи
- [render()](#render) — рендеринг формы с возвратом HTML
- [setSaveHandler()](#setsavehandler) — установка обработчика сохранения
- [saveSuccess()](#savesuccess) — установка скрипта при успешном сохранении (устарело)
- [addSuccessScript()](#addsuccessscript) — добавление скрипта при успешном сохранении
- [addSuccessNotice()](#addsuccessnotice) — добавление уведомления при успешном сохранении
- [setSuccessUrl()](#setsuccessurl) — установка URL для перехода после сохранения
- [addParams()](#addparams) — добавление параметров

---

## Детальное описание методов

### __construct()
**Описание**: Инициализация формы редактирования с установкой базовых параметров, ресурса и настроек доступа.  
**Параметры**:
- `$name` (string) — имя ресурса/формы

**Пример**:
```php
$edit = new \editTable('mod_storages');
```

---


### setTemplate()
**Описание**: Установка пользовательского HTML-шаблона для формы редактирования.  
**Сигнатура**: `public function setTemplate($html): self`  
**Параметры**:
- `$html` (string) — HTML-код шаблона

**Возвращает**: `$this`

**Пример**:
```php
$edit->setTemplate('<div id="custom_form">[default]</div>');
```

---

### setTemplateControl()
**Описание**: Настройка произвольного шаблона для элемента управления определенного типа.  
**Сигнатура**: `public function setTemplateControl($type, $filename): self`  
**Параметры**:
- `$type` (string) — тип элемента управления (например, 'xfile', 'date2')
- `$filename` (string) — абсолютный путь к файлу шаблона

**Возвращает**: `$this`

**Пример**:
```php
$edit->setTemplateControl('xfile', Theme::get("html-edit-files"));
```

---

### setReadonly()
**Описание**: Установка формы в состояние "только для чтения".  
**Сигнатура**: `public function setReadonly(bool $is_readonly): void`  
**Параметры**:
- `$is_readonly` (bool) — флаг режима только для чтения

**Пример**:
```php
$edit->setReadonly(true);
```

---

### setReadonlyFields()
**Описание**: Установка списка полей, которые должны быть в состоянии только для чтения (с предварительной очисткой).  
**Сигнатура**: `public function setReadonlyFields(array $fields): void`  
**Параметры**:
- `$fields` (array) — массив имен полей

**Пример**:
```php
$edit->setReadonlyFields(['name', 'code', 'status']);
```

---

### addReadonlyFields()
**Описание**: Добавление полей в список полей только для чтения.  
**Сигнатура**: `public function addReadonlyFields(array $fields): void`  
**Параметры**:
- `$fields` (array) — массив имен полей для добавления

**Пример**:
```php
$edit->addReadonlyFields(['created_at', 'updated_at']);
```

---

### clearReadonlyFields()
**Описание**: Очистка списка полей, которые должны быть в состоянии только для чтения.  
**Сигнатура**: `public function clearReadonlyFields(): void`

**Пример**:
```php
$edit->clearReadonlyFields();
```

---

### addControl()
**Описание**: Добавление нового элемента управления (поля) к форме.  
**Сигнатура**: `public function addControl($name, $type, $in = "", $out = "", $default = "", $req = false): self`  
**Параметры**:
- `$name` (string) — заголовок/имя поля
- `$type` (string) — тип элемента управления (например, 'text', 'select', 'date')
- `$in` (string|array) — атрибуты поля
- `$out` (string) — внешний HTML
- `$default` (string) — значение по умолчанию
- `$req` (bool) — обязательное поле

**Возвращает**: `$this`

**Пример**:
```php
$edit->addControl('Наименование', 'text', '', '', '', true);
```

---

### addGroup()
**Описание**: Добавление группы элементов (сворачиваемый блок).  
**Сигнатура**: `public function addGroup($name, $collapsed = false): self`  
**Параметры**:
- `$name` (string) — имя группы
- `$collapsed` (bool) — свернута ли группа по умолчанию

**Возвращает**: `$this`

**Пример**:
```php
$edit->addGroup('Основные данные', false);
```

---

### addButton()
**Описание**: Добавление стандартной кнопки к форме.  
**Сигнатура**: `public function addButton($value, $action = ''): self`  
**Параметры**:
- `$value` (string) — текст на кнопке
- `$action` (string) — JavaScript-действие при клике

**Возвращает**: `$this`

**Пример**:
```php
$edit->addButton('Сохранить', 'this.form.onsubmit();');
```

---

### addButtonSwitch()
**Описание**: Создание кнопки-переключателя (switch) для полей с значениями Y/N.  
**Сигнатура**: `public function addButtonSwitch($field_name, $value): self`  
**Параметры**:
- `$field_name` (string) — имя поля
- `$value` (string) — текущее значение (Y или N)

**Возвращает**: `$this`

**Пример**:
```php
$edit->addButtonSwitch('is_active', 'Y');
```

---

### addButtonCustom()
**Описание**: Добавление произвольной кнопки с пользовательским HTML.  
**Сигнатура**: `public function addButtonCustom($html = ''): self`  
**Параметры**:
- `$html` (string) — HTML-код кнопки

**Возвращает**: `$this`

**Пример**:
```php
$edit->addButtonCustom('<button class="btn">Кнопка</button>');
```

---

### setSessFormField()
**Описание**: Сохранение значения в служебных полях формы (сессия).  
**Сигнатура**: `public function setSessFormField($id, $value): self`  
**Параметры**:
- `$id` (string) — идентификатор поля
- `$value` (mixed) — значение

**Возвращает**: `$this`

**Пример**:
```php
$edit->setSessFormField('maxWidth', 800);
```

---

### setLeaveChecking()
**Описание**: Установка проверки изменений на форме при уходе со страницы.  
**Сигнатура**: `public function setLeaveChecking(bool $leave_checking): self`  
**Параметры**:
- `$leave_checking` (bool) — включить проверку

**Возвращает**: `$this`

**Пример**:
```php
$edit->setLeaveChecking(true);
```

---

### setTable()
**Описание**: Установка таблицы базы данных для формы.  
**Сигнатура**: `public function setTable(string $table): self`  
**Параметры**:
- `$table` (string) — имя таблицы

**Возвращает**: `$this`

**Пример**:
```php
$edit->setTable('mod_storages_suppliers');
```

---

### setWidthLabels()
**Описание**: Установка ширины для названий полей.  
**Сигнатура**: `public function setWidthLabels(string|int $width): self`  
**Параметры**:
- `$width` (string|int) — ширина (например, '200px' или 200)

**Возвращает**: `$this`

**Пример**:
```php
$edit->setWidthLabels(230);
```

---

### setData()
**Описание**: Установка данных записи для редактирования.  
**Сигнатура**: `public function setData(array $record): self`  
**Параметры**:
- `$record` (array) — ассоциативный массив данных

**Возвращает**: `$this`

**Пример**:
```php
$edit->setData(['id' => 1, 'name' => 'Тест', 'status' => 'Y']);
```

---

### render()
**Описание**: Рендеринг формы с возвратом HTML-кода.  
**Сигнатура**: `public function render(array $options = []): string`  
**Параметры**:
- `$options` (array) — опции отображения

**Возвращает**: `string` — HTML-код формы

**Пример**:
```php
echo $edit->render(['scroll_to_form' => true]);
```

---


### setSaveHandler()
**Описание**: Установка обработчика сохранения через xajax.  
**Сигнатура**: `public function setSaveHandler(string $handler): self`  
**Параметры**:
- `$handler` (string) — имя обработчика

**Возвращает**: `$this`

**Пример**:
```php
$edit->setSaveHandler("saveShipmentStep1");
```

---

### saveSuccess()
**Описание**: Установка скрипта при успешном сохранении (устарело, используйте addSuccessScript).  
**Сигнатура**: `public function saveSuccess(string $func): self`  
**Параметры**:
- `$func` (string) — JavaScript-код

**Возвращает**: `$this`

**Пример**:
```php
$edit->saveSuccess('alert("Сохранено");');
```

---

### addSuccessScript()
**Описание**: Добавление скрипта при успешном сохранении.  
**Сигнатура**: `public function addSuccessScript(string $script): self`  
**Параметры**:
- `$script` (string) — JavaScript-код

**Возвращает**: `$this`

**Пример**:
```php
$edit->addSuccessScript('CoreUI.notice.create("Сохранено");');
```

---

### addSuccessNotice()
**Описание**: Добавление уведомления при успешном сохранении.  
**Сигнатура**: `public function addSuccessNotice(string $text = null): self`  
**Параметры**:
- `$text` (string) — текст уведомления

**Возвращает**: `$this`

**Пример**:
```php
$edit->addSuccessNotice('Данные успешно обновлены');
```

---

### setSuccessUrl()
**Описание**: Установка URL для перехода после успешного сохранения.  
**Сигнатура**: `public function setSuccessUrl(string $url): self`  
**Параметры**:
- `$url` (string) — URL адрес

**Возвращает**: `$this`

**Пример**:
```php
$edit->setSuccessUrl('index.php?module=list');
```

---

### addParams()
**Описание**: Добавление параметров к форме.  
**Сигнатура**: `public function addParams($va, $value = ''): self`  
**Параметры**:
- `$va` (string) — имя параметра
- `$value` (string) — значение

**Возвращает**: `$this`

**Пример**:
```php
$edit->addParams('field-name', '12345-abc');
```


---

## Типичный пример использования класса целиком

```php
<?php
// Подключение необходимых файлов
require_once("core2/inc/classes/class.edit.php");

// Создание формы редактирования поставщика
$edit = new \editTable('storages_suppliers');

// Настройка формы
$edit->setTable('mod_storages_suppliers')                    // Указываем таблицу БД
     ->setWidthLabels(200)                                   // Ширина меток полей
     ->setSuccessUrl('index.php?module=storages&suppliers')  // URL после сохранения
     ->addSuccessNotice('Поставщик успешно сохранен')        // Уведомление после сохранения
     ->addSuccessScript('console.log("Сохранено");')         // Скрипт после сохранения
     ->setSaveHandler("saveSupplierData");                   // Настройка метода сохранения

// $edit->setReadonly(true)  // Включает режим только для чтения

// Устанавливаем данные (если редактируем)
$edit->setData([
    'id'        => 123, 
    'name'      => 'ООО Тест',
    'inn'       => '1111111',
    'kpp'       => '222222',
    'address'   => 'г. Минск, уд. Революционная, д. 1',
    'is_active' => 'Y',
    'type'      => 'legal',
    'discount'  => 15,
]);          

// Добавление полей формы
$edit->addControl('Название компании',  $edit::TYPE_TEXT, '', '', '', true);      // Обязательное поле
$edit->addControl('ИНН',                $edit::TYPE_TEXT, '');
$edit->addControl('КПП',                $edit::TYPE_TEXT, '');
$edit->addControl('Юридический адрес',  $edit::TYPE_TEXTAREA, '');
$edit->addControl('Активен',            $edit::TYPE_SWITCH);                      // Переключатель да/нет
$edit->addControl('Тип поставщика',     $edit::TYPE_SELECT, '', '', '', false);   // Выпадающий список


// Настройка списка для поля select (пример)
$edit->selectSQL[0] = [
    '1' => 'Основной поставщик',
    '2' => 'Резервный поставщик',
    '3' => 'Тестовый поставщик'
];

// Добавление группы дополнительных настроек
$edit->addGroup('Дополнительные параметры', true);  // true - свернута по умолчанию
$edit->addControl('Скидка, %',   $edit::TYPE_NUMBER);
$edit->addControl('Комментарий', $edit::TYPE_TEXTAREA);

// Добавление кнопок
$edit->addButton('Отмена', 'index.php?module=storages&suppliers');


// Отображение формы
echo $edit->render();
```

## Пример простого использования (минимальный вариант)

```php
<?php
// Быстрая форма для добавления записи
$edit = new \editTable('users_index');
$edit->setTable('mod_users')
    ->setSaveHandler("saveUser");
    
$edit->setData([
    'id'    => 123, 
    'login' => 'admin',
    'email' => 'admin@example.com',
    'pass'  => '12345',
]);    

$edit->addControl('Логин',  $edit::TYPE_TEXT,     '', '', '', true);
$edit->addControl('Email',  $edit::TYPE_TEXT,     '', '', '', true);
$edit->addControl('Пароль', $edit::TYPE_PASSWORD, '', '', '', true);

echo $edit->render();
```