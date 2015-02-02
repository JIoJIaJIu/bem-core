# events__channels

Элемент `channels` блока `events` предназначен для работы с именными каналами событий. Именные каналы позволяют организовать работу с событиями, используя шаблон проектирования «наблюдатель» (также известный как Publisher-subscriber).

Элемент реализует функцию, позволяющую:

* получить ссылку на именной канал по `id`.
* получить ссылку на стандартный канал.
* удалить канал – стандартный или по `id`.

Элемент `events` реализован в технологии `vanila.js` и подходит для использования как на клиенте, так и на сервере.

Функция принимает аргументы:

* `{String}` `[id]` – Идентификатор канала. Если не задан будет использоваться канал по умолчанию (`'default'`).
* `{Boolean}` `[drop]` – Логический флаг, указывающий (в значении `true`) на необходимость удалить канал. По умолчанию `false`.

Функция возвращает:
* ссылку на объект «класса» `Emitter` – именной канал.
* `undefined` если функция была вызвана с параметром `drop` в значении `true`.

```js
modules.define('channels-test', ['events__channels'], function(provide, channels) {
    var _shout = function(e){ e.data && console.log(e.data) },
        getDef = function(){
            // получаем ссылку на стандартный канал
            return channels();
        },
        dropDef = function(){
            // удаляем стандартный канал
            return channels(true);
        },
        def = getDef();

    def.on('test', _shout);
    def.emit('test', 'my message'); // 'my message'

provide({ 
    'getDef': getDef,
    'dropDef': dropDef
});
});
```
