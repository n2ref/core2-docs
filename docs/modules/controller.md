
Файл контроллер реализует логику работы интерфейса модуля и обеспечивает возможность получения доступа к ресурсам модуля. 
Файл обязателен и всегда находится в корне модуля. Его называние состоит из `Mod`, названия модуля и `Controller.php`. Например `ModOrdersController.php`

Файл содержит класс с названием аналогичным названию файла. 
Внутри класса могут находиться методы, которые вызываются для отображения интерфейса системы. 
Такие методы начинаются с названия `action_` и заканчиваются названием субмодуля который они описывают. Например `action_index`.

По умолчанию пользователь может зайти на главную страницу модуля, например `index.php#?module=orders`, и попасть в метод `action_index`. 
Если требуется чтобы у модуля были другие субмодули, то они должны быть добавлены в базу данных в таблицу субмодулей и для них должны быть добавлены соответствующие методы `action_example`.
В таком случае, адрес в строке браузера будет выглядеть примерно так `index.php#?module=orders&action=example`. 

РЕКОМЕНДУЕТСЯ добавлять в doсblock класса описания моделей таблиц этого класса, для более удобной работы в IDE в дальнейшем.

Пример файла

```php
/**
 * @property \Orders $dataOrders
 */
class ModOrdersController extends Common {


    /**
     * @return string
     * @throws Exception
     */
    public function action_index(): string {
        
        return 'text content'; 
    }


    /**
     * @return string
     * @throws Exception
     */
    public function action_example(): string {
        
        return 'text content 2'; 
    }
}
```

Кроме методов `action` в модуле могут быть реализованы другие, свои публичные методы, которые могут быть вызваны в других модулях, чтобы получить какой-то функционал из этого модуля.


### Дополнительная функциональность

В модуле могут быть реализованы интерфейсы ядра, при наличии которых модуль получает дополнительные возможности, описанные через интерфейс.

#### Delete

Дает возможность обрабатывать удаления совершенные через интерфейс таблиц. 
Обязывает реализовать метод `action_delete`.

Пример
```php
    /**
     * @param string $resource
     * @param string $id
     * @return array|bool
     */
    public function action_delete($resource, $id) {

        switch ($resource) {
            case 'orders':
                $orders_id = explode(',', $ids);

                foreach ($orders_id as $order_id) {
                    $order = $this->dataOrders->find($order_id)->current();                    
                    $order->delete();
                }
                
                return true;
                break;
        }

        return false;
    }
```

#### Subscribe

Дает возможность обрабатывать события совершенные в системе и различных ее модулях.
Обязывает реализовать метод `listen`.

Пример
```php
    /**
     * @param string $module_id
     * @param string $event
     * @param array  $data
     * @return void
     * @throws Zend_Db_Adapter_Exception
     * @throws Exception
     */
    public function listen($module_id, $event, $data) {

        switch ($event) {
            case 'update_order_person_data': (new Orders\Events())->updateOrderPerson($data); break;
            case 'payment_update':           (new Orders\Events())->updatePayment($data); break;
        }
    }
```

#### File

Дает возможность обрабатывать запросы связанные с получением списка файлов для интерфейса в полях форм. 
Обязывает реализовать метод `action_filehandler`.

Пример
```php
    /**
     * @param string $context
     * @param string $table
     * @param string $id
     * @return true|void
     * @throws \Core2\HttpException
     */
    public function action_filehandler($context, $table, $id) {

        if ($table == 'mod_orders_products_fobox') {
            $field_name = $context == 'field_img' ? 'img' : 'video';
            $files      = $this->dataOrdersProductsFoboxFiles->getRowsByServiceFoboxIdField($id, $field_name);

            $result = [];
            foreach ($files as $file) {

                $file_content = $this->dataOrdersProductsFoboxFilesContent->getDataById($file->content_id);

                $result[] = [
                    'name'          => $file->filename,
                    'hash'          => $file_content['hash'],
                    'size'          => $file_content['filesize'],
                    'type'          => $file_content['type'],
                    'thumbnail_url' => preg_match('~image/*~', $file_content['type']) ? "orders/index/order/fobox/file/{$file->id}/thumb" : '',
                    'url'           => "orders/index/order/fobox/file/{$file->id}/content",
                    'delete_id'     => $file->id,
                ];
            }

            header('Content-type: application/json');
            echo json_encode(['files' => $result]);
            return true;
        }
    }
```

#### Switches

Дает возможность обрабатывать запросы связанные переключением активности в колонках таблиц.
Обязывает реализовать метод `action_switch`.

Пример
```php
    /**
     * @param string $resource_name
     * @param string $field
     * @param string $id
     * @param string $value
     * @return bool|array
     * [ status => 'Текст ошибки' ]
     * Если возвращено true, то будет считаться, что вы самостоятельно переключили нужный объект
     * Если возвращено false будет считаться, что нужно применить стандартную процедуру переключения
     */
    public function action_switch(string $resource_name, string $field, string $id, string $value) {

        switch ($resource_name) {
            case 'ordersxxx_products':
                $product = $this->dataOrdersProducts->getRowById($id);

                if ($product) {
                    $product->is_active_sw = $value;
                    $product->save();
                    return true;
                }
                break;
        }

        return false;
    }
```

#### Sequence

Дает возможность обрабатывать запросы связанные сортировкой строк в таблицах модуля.
Обязывает реализовать метод `sequence`.

Пример
```php
    /**
     * @param string $resource_name
     * @param array  $records
     * @return bool
     * @throws Zend_Db_Table_Exception
     * Если возвращено true, то будет считаться, что вы самостоятельно сортировали объекты
     * Если возвращено false будет считаться, что нужно применить стандартную процедуру сортировки
     */
    public function sequence(string $resource_name, array $records): bool {

        switch ($resource_name) {
            case 'ordersxxx_products':
                $products = $this->dataOrdersProducts->find($records);

                foreach ($products as $product) {
                    if (isset($records[$product->id]) &&
                        $records[$product->id] > 0 &&
                        $product->seq != (int)$records[$product->id]
                    ) {
                        $product->seq = (int)$records[$product->id];
                        $product->save();
                    }
                }

                return true;
                break;
        }

        return false;
    }
```

#### TopCss

Дает возможность загружать css скрипты в корень системы при ее открытии. 
Таким образом можно добавлять различную функциональность в системе, которая будет доступна отовсюду.  
Обязывает реализовать метод `topCss`.

Пример
```php    
    /**
     * @return array
     * @throws Exception
     */
    public function topCss(): array {

        $mod_src = $this->getModuleSrc('discuss');

        return [
            "{$mod_src}/assets/chat/dist/chat.min.css",
        ];
    }
```

#### TopJs

Дает возможность загружать js скрипты в корень системы при ее открытии. 
Таким образом можно добавлять различную функциональность в системе, которая будет доступна отовсюду.  
Обязывает реализовать метод `topJs`.

Пример
```php    
    /**
     * @return array
     * @throws Exception
     */
    public function topJs(): array {

        $mod_src = $this->getModuleSrc('discuss');

        return [
            "{$mod_src}/assets/chat/dist/chat.min.js",
        ];
    }
```


#### Navigation

Дает возможность добавлять свои пункты в основное меню системы.   
Обязывает реализовать метод `navigationItems`.

Пример
```php
    /**
     * Добавление пунктов меню
     * @param \Core2\Navigation $nav
     * @throws Exception
     */
    public function navigationItems(\Core2\Navigation $nav) {

        $item = $dropdown->addLink("Корзина", "index.php#module=ordering&basket=1");
        $item->setIcon('<i class="fa fa-fw fa-shopping-cart"></i>');


        $item = $dropdown->addFile("Импорт");
        $item->setIcon('<i class="fa fa-fw fa-download"></i>');
        $item->setOnChange("OrderingBasket.importFile(this);");

        $dropdown->addDivider();

        $item = $dropdown->addLink("Очистить", "#");
        $item->setIcon('<i class="fa fa-fw fa-trash"></i>');
        $item->setOnClick("OrderingBasket.clearBasket();return false");
    }
```
