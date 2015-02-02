# events

Блок предоставляет механизмы для работы с событиями JavaScript, реализованные в виде псевдоклассов:

* `Event` – для создания объекта события, изменения и проверки его состояний.
* `Emitter` – для генерации событий и подписки на них.

Блок `events` реализован в технологии `vanila.js` и подходит для использования как на клиенте, так и на сервере.

Блок служит основой механизма событий в `bem-core`. Базовый блок `BEM` наследуется от «класса» `Emitter`. Поэтому нет необходимости использовать `events`, если вы подключаете блок `i-bem`. 

## `Event`

С помощью псевдокласса можно инстанцировать объект события, указав при необходимости тип и триггер события:

```js
modules.define('event-test', ['events'], function(provide, events) {
    var myevent = new events.Event('myevent', this);
    console.log(myevent);

provide(myevent);
});
```

Функция-конструктор, выполняемая в ходе инстанцирования объекта события, принимает аргументы:

* `type` `{String}` – тип события.
* `target` `{Object}` – объект (триггер) на котором событие возникло.

Значения аргументов присваиваются одноименным полям объекта событий. Кроме указанных, новый объект событий содержит следующий набор полей: 

* `{*}` `result` – по умолчанию `undefined`. Поле для сохранения результата выполнений функции-обработчика события.
* `{*}` `data` – по умолчанию `undefined`. Поле для хранения произвольных данных, передаваемых обработчику события.
* `{Boolean}` `_isDefaultPrevented` – по умолчанию `false`. Предотвращать стандартное действие для HTML-элемента (со значением `true`).
* `{Boolean}` `_isPropagationStopped` – по умолчанию `false`. Остановить трансляцию события (со значением `true`).

### Методы объекта события

#### `preventDefault`

Служит для предотвращения выполнения стандартного действия, заданного для HTML-элемента. Фактически, метод устанавливает полю `_isDefaultPrevented` значение `true`. Метод не принимает аргументов.

```js
modules.define('event-test', ['events'], function(provide, events) {
    var myevent = new events.Event('myevent', this);

    myevent.preventDefault();    
    console.log(myevent);

provide(myevent);
});
```


#### `isDefaultPrevented`

Позволяет проверить было ли предотвращено выполнение стандартного действия для события. Метод не принимает аргументов и возвращает `true`, если было:

```js
modules.define('event-test', ['events'], function(provide, events) {
    var myevent = new events.Event('myevent', this);

    console.log(myevent.isDefaultPrevented()); // false
    myevent.preventDefault();
    console.log(myevent.isDefaultPrevented()); // true

provide(myevent);
});
```


#### `stopPropagation` 

Служит для остановки трансляции события. Фактически, метод устанавливает полю `_isPropagationStopped` значение `true`. Метод не принимает аргументов.

```js
modules.define('event-test', ['events'], function(provide, events) {
    var myevent = new events.Event('myevent', this);

    myevent.stopPropagation();    
    console.log(myevent);

provide(myevent);
});
```


#### `isPropagationStopped`

Позволяет проверить была ли выполнена остановка трансляции события. Метод не принимает аргументов и возвращает `true`, если была:

```js
modules.define('event-test', ['events'], function(provide, events) {
    var myevent = new events.Event('myevent', this);

    console.log(myevent.isPropagationStopped()); // false
    myevent.stopPropagation();
    console.log(myevent.isPropagationStopped()); // true

provide(myevent);
});
```


## `Emitter`

С помощью псевдокласса можно генерировать события и осуществлять подписку на них. Это производится с помощью методов «класса».

```js
modules.define('event-test', ['events'], function(provide, events) {
    var myemmiter = new events.Emitter();

provide(myemmiter);
});
```


### Методы «класса»

Набор и сигнатуры статических методов идентичны набору и сигнатурам методов объекта, инстанцируемого классом.

#### `on`

Служит для подписки на событие определенного типа. Метод принимает следующие аргументы:

 * `{String}` `e` – тип события, на которое производится подписка.
 * `{Object}` `[data]` – дополнительные данные, доступные как значение поля `e.data` объекта события.
 * `{Function}` `fn` – функция-обработчик, вызываемая для события.
 * `{Object}` `[ctx]` – контекст функции-обработчика.

Метод возвращает ссылку на объект `this`.

```js
modules.define('event-test', ['events'], function(provide, events) {
    var myemmiter = new events.Emitter(),
        _shout = function(){ console.log('foo') },
        _yeld = function(){ console.log('bar') },
        subscribe = function(){
            // подписываемся на событие myevent с помощью статического метода «класса» Emitter
            events.Emitter.on('myevent', _shout); 
            // подписываемся на событие myevent с помощью собственного метода объекта
            myemmiter.on('myevent', _yeld);
        };

    subscribe();

    events.Emitter.emit('myevent'); // 'foo'
    myemmiter.emit('myevent'); // 'bar'
    
provide({ 'subscribe' : subscribe });
});
```

Кроме того, значением аргумента `e` могут быть:

* несколько типов событий, перечисленных через пробел – чтобы установить для них общую функцию-обработчик. 

```js
modules.require(['events'], function(events) {
    var _shout = function(){ console.log('foo') };

    events.Emitter.on('myevent event2', _shout); 

    events.Emitter.emit('myevent'); // 'foo'
    events.Emitter.emit('event2'); // 'foo'
});
```

* хеш вида `{ 'событие-1' : обработчик-1, ... , 'событие-n' : обработчик-n }` – чтобы установить сразу несколько обработчиков для разных типов событий.

```js
modules.require(['events'], function(events) {
    var _shout = function(){ console.log('foo') },
        _yeld = function(){ console.log('bar') };
    
    events.Emitter.on({ 'myevent': _shout, 'event2': _yeld });  

    events.Emitter.emit('myevent'); // 'foo'
    events.Emitter.emit('event2'); // 'bar'
});
```

Сказанное выше верно и для методов `once` и `un`.

#### `once`

Идентичен методу `on`, с той разницей, что выполняется единожды – после первого события подписка удаляется. Метод принимает следующие аргументы:

 * `{String}` `e` – тип события, на которое производится подписка.
 * `{Object}` `[data]` – дополнительные данные, доступные как значение поля `e.data` объекта события.
 * `{Function}` `fn` – функция-обработчик, вызываемая для события.
 * `{Object}` `[ctx]` – контекст функции-обработчика.

Метод возвращает ссылку на объект `this`.

```js
modules.define('event-test', ['events'], function(provide, events) {
    var _shout = function(){ console.log('foo') },
        runOnce = function(){
            // подписываемся на событие myevent с помощью статического метода «класса» Emitter
            events.Emitter.on('myevent', _shout); 
        };

    runOnce();

    events.Emitter.emit('myevent'); // 'foo'
    events.Emitter.emit('myevent'); // обработчик не вызывается

provide({ 'runOnce' : runOnce });
});
```


#### `un`

Служит для удаления установленной ранее функции-обработчика события. Метод принимает следующие аргументы:

 * `{String}` `e` – тип события у которого удаляется обработчик.
 * `{Function}` `fn` – удаляемый обработчик.
 * `{Object}` `[ctx]` – контекст обработчика.

Метод возвращает ссылку на объект `this`.

```js
modules.define('event-test', ['events'], function(provide, events) {
    var myemmiter = new events.Emitter(),
        _shout = function(){ console.log('foo') },
        _yeld = function(){ console.log('bar') },
        subscribe = function(){
            // подписываемся на событие myevent с помощью статического метода «класса» Emitter
            events.Emitter.on('myevent', _shout); 
            // подписываемся на событие myevent с помощью собственного метода объекта
            myemmiter.on('myevent', _yeld);
        },
        unsubscribe = function(){
            // отписываемся от события myevent с помощью статического метода «класса» Emitter
            events.Emitter.un('myevent', _shout); 
            // отписываемся от события myevent с помощью собственного метода объекта
            myemmiter.un('myevent', _yeld);
        };

    subscribe();

    events.Emitter.emit('myevent'); // 'foo'
    myemmiter.emit('myevent'); // 'bar'

    unsubscribe();
            
provide({
    'subscribe' : subscribe,
    'unsubscribe' : unsubscribe
});
});
```

### emit

Служит для генерации события. Другими словами, метод вызывает все функции-обработчики, заданные для события, переданного аргументом. Метод принимает следующие аргументы:

 * `{String|events:Event}` `e` – генерируемое событие в виде строки или ссылки на готовый объект события.
 * `{Object}` `[data]` – дополнительные данные, доступные как второй аргумент функции-обработчика.

Метод возвращает ссылку на объект `this`.

```js
modules.define('event-test', ['events'], function(provide, events) {
    var myevent = new events.Event('myevent'),
        _shout = function(e, data){ 
            data ?
            console.log(data) :
            console.log(e.data + ' from e.data'); 
        },
        subscribe = function(){
            events.Emitter.on('myevent', 'bar', _shout); 
        };

    subscribe();

    events.Emitter.emit(myevent);               // 'bar from e.data'
    events.Emitter.emit(myevent, 'foo');        // 'foo'
    events.Emitter.emit(myevent);               // 'bar from e.data'

    events.Emitter.emit('myevent');             // 'foo from e.data'
    events.Emitter.emit('myevent', 'ololo');    // 'ololo'
    events.Emitter.emit('myevent');             // 'ololo from e.data'
    
    events.Emitter.emit(myevent);               // 'bar from e.data'

provide({ 'subscribe' : subscribe });
});
```

Как видно из примера, при использовании готового объекта события, значение аргумента `[data]` не присваивается полю `e.data` объекта.

Результаты выполнения функции-обработчика сохраняются в поле `e.result` соответствующего объекта события.

```js
modules.require(['events'], function(events) {
    var myevent = new events.Event('myevent'),
        _tincle = function(){ 
            return 'hi-hi-hi' 
        };

    events.Emitter.on('myevent', _tincle); 

    events.Emitter.emit(myevent);
    console.log(myevent.result);    // 'hi-hi-hi'
});
```
