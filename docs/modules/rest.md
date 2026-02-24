
Для работы с модулем по средствам архитектурного стиля REST api в корень модуля нужно добавить файл api контроллер.
Такой файл должен называться начиная с `Mod`, далее название модуля и `Api.php`, например `ModOrdersApi.php`.
Внутри файла должен находиться класс с таким же названием, например `ModOrdersApi`.


Для обработки REST запросов, внутри api класса **ДОЛЖНЫ** находиться публичные методы, 
названия которых начинается на `action_` и далее название субмодуля или index, например `action_index`. 
Авторизованные запросы будет попадать в такие методы, если у роли пользователя достаточно прав на доступ к ним.
Метод `action_index` является методом главной страницы и он доступен если есть доступ к модулю. 


Чтобы запросы были обработаны модулем, они **ДОЛЖНЫ** иметь адрес начинающийся на название модуля и далее субмодуля (или index).
Например `https://host.com/orders/index` чтобы запрос обработал метод `action_index` или `https://host.com/orders/example` 
чтобы его обработал метод `action_example`. Важно, если субмодуль не зарегистрирован в системе, то метод с его названием не будет открыт, 
даже если он будет существовать.  

При попадании запроса в необходимый метод РЕКОМЕНДУЕТСЯ использовать системный класс `\Core2\Routing\Router` для его обработки и ответа.
Описание работы с этим классом можно посмотреть в соответствующем разделе документации.

После добавления обработки соответствующих роутов их требуется описать в файле `schema.json`. 
Файл содержит json в формате `openapi`. Роуты добавленные в него будут доступны в `swagger` и `scalar` системы.
Их можно открыть для просмотра по адресу `https://example.com/core2/swagger/index.html` или `https://example.com/core2/scalar/index.html`.

Пример содержимого файла 
```php
require_once DOC_ROOT . "core2/inc/classes/CommonApi.php";
require_once DOC_ROOT . "core2/inc/classes/Routing/Router.php";

/**
 * 
 */
class ModOrdersApi extends CommonApi {

    /**
     * @return mixed
     * @throws Zend_Db_Table_Exception
     * @throws Exception
     */
    public function action_index(): mixed {

        $router = new \Core2\Routing\Router('/orders/index', [
            ''                  => ['get' => [Api\Index::class, 'getOrders']],
            '/order'            => ['post' => [Api\Order::class, 'createOrder']],
            '/order/{order_id}' => [
                'get'    => [Api\Order::class, 'getOrder'],
                'patch'  => [Api\Order::class, 'patchOrder'],
                'delete' => [Api\Order::class, 'deleteOrder'],
            ]
        ]);

        return $this->runRouter($router);
    }
    
    
    /**
     * @return mixed
     * @throws Exception
     */
    public function action_example(): mixed {

        $router = new \Core2\Routing\Router('/orders/example', [
            '/test' => ['post' => [Api\Example::class, 'testMethod']],
        ]);

        return $this->runRouter($router);
    }
}
```