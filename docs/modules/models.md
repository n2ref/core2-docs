
Для работы с таблицами базы данных **НУЖНО** использовать модели таблиц. 
Для этого в папке модуля `Models` требуется добавить все относящиеся к модулю таблицы в виде классов в своих отдельных файлах.

Класс **ДОЛЖЕН** наследоваться от класса `\Zend_Db_Table_Abstract`, который дает требуемую функциональность.
Далее, в таком классе обязательно **ДОЛЖНО** быть указано название таблицы в параметре `$_name`, остальные данные опциональны.

Таким образом если таблица в базе данных называется `mod_orders_products`, то файл **ДОЛЖЕН** иметь в своем названии
такое же название, но без приставки `mod_` в формате CamelCase. Соответственно так же должен называться и класс.

РЕКОМЕНДУЕТСЯ добавлять названия таких моделей в docblock контроллера модуля, для более удобной работы в IDE.

Пример
```php
/**
 * @property \TableOrders         $dataTableOrders
 * @property \TableOrdersProducts $dataTableOrdersProducts
 */
class ModOrderingController extends Common {}
```


В классе можно добавлять методы для получения объектов строк, РЕКОМЕНДУЕТСЯ называть их начиная с `getRowBy` или `getRowsBy` 
и далее указывать поля ко которым будет происходить выборка, например `getRowById`. 
Внутри класса РЕКОМЕНДУЕТСЯ работать только с данными этой таблицы, к которой он относится. 


Пример файла 
```php
/**
 *
 */
class TableOrdersProducts extends Zend_Db_Table_Abstract {
    
    protected $_name = 'mod_orders_products';
    
    
    /**
     * Получение объекта по id
     * @param int $id
     * @return Zend_Db_Table_Row_Abstract|null
     */
    public function getRowById(int $id):? Zend_Db_Table_Row_Abstract {

        $select = $this->select()->where("id = ?", $id);

        return $this->fetchRow($select);
    }
}
```