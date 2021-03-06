# Идиоматический Redux: Дао Redux, Часть 1 - Реализация и намерения
*Мысли о том, что Redux требует, как его использовать и какие возможности он предоставляет*

*Перевод статьи [Mark Erikson](https://twitter.com/acemarke): [Idiomatic Redux: The Tao of Redux, Part 1 - Implementation and Intent](http://blog.isquaredsoftware.com/2017/05/idiomatic-redux-tao-of-redux-part-1/). Опубликовано с разрешения автора.*

![](tao.jpg)

## Вступление

> Я потратил много времени на обсуждения шаблонов использования Redux в Интернете, будь то помощь в ответах на вопросы от учащихся в каналах Reactiflux, дискуссии о возможных изменениях в API Redux на Github или обсуждения различных аспектов Redux в комментариях на Reddit и HN. Со временем я выработал своё собственное мнение о том, что представляет собой хороший, идиоматический код Redux, и я хотел бы поделиться некоторыми из этих мыслей. Несмотря на мой статус мантейнера Redux, это всего лишь мои взгляды, но я думаю, что их хорошо придерживаться:)

Redux, по своей сути, невероятно прост. Он сохраняет текущее значение, запускает, когда это необходимо, функцию для обновления этого значения и уведомляет всех подписчиков о том, что что-то изменилось.

Несмотря на эту простоту или, возможно, из-за этого, существует множество подходов, мнений и взглядов о том, как использовать Redux. Многие из этих подходов широко расходятся с концепциями и примерами документации.

В то же время постоянно поступают жалобы на то, как Redux «заставляет» вас делать определенные вещи. Многие жалобы на самом деле связаны с концепциями, связанными с тем, как обычно используется Redux, а не с каким-либо фактическим ограничением, налагаемым самой библиотекой Redux. (Например, всего лишь в одном потоке [HN](https://news.ycombinator.com/) я видел жалобы на «слишком много шаблонов», «константы экшенов (*action constants*) и ActionCreators (*action creators*) не нужны», «мне нужно отредактировать слишком много файлов, чтобы добавить функциональность», «почему мне нужно переключать файлы, чтобы писать собственную логику?», «термины и имена слишком сложны для изучения или запутанные», - и это далеко не всё.)

Поскольку я искал, читал, обсуждал и изучал различные способы использования Redux и идеи, которыми обмениваются в сообществе, я пришел к выводу, что **важно различать, как *действительно* работает Redux, пути использования, для которых он *предназначен* концептуально, и почти бесконечное количество *возможных* способов использования Redux**. Я хотел бы затронуть несколько аспектов использования Redux и обсудить, как они вписываются в эти категории. В целом, **я надеюсь объяснить, почему специфичные шаблоны использования и практики Redux существуют, философию и намерения Redux, а также то, что я считаю «идиоматическим» и «неидиоматическим» использованием Redux**.

Этот пост будет разделен на две части. В **Часть 1 - Реализация и намерения** мы рассмотрим фактическую реализацию Redux, какие ограничения он накладывает и почему эти ограничения существуют. Затем мы рассмотрим первоначальные намерения и цели разработки Redux на основе обсуждений и заявлений авторов (особенно в ходе раннего процесса разработки).

В [Часть 2 - Практика и философия](http://blog.isquaredsoftware.com/2017/05/idiomatic-redux-tao-of-redux-part-2/) мы исследуем общие практики, которые широко используются в приложениях Redux, и опишем, почему эти практики существуют в качестве первоочередных. Наконец, мы рассмотрим ряд «альтернативных» подходов к использованию Redux и обсудим, почему многие из них возможны, но не обязательно «идиоматичны».

## Закладываем фундамент

### Изучаем три принципа

Начнем с рассмотрения широко известных [трёх принципов Redux](http://redux.js.org/docs/introduction/ThreePrinciples.html):

* **Единственный источник правды**: стейт (*state*) всего вашего приложения хранится объектным деревом в одном сторе (*store*).
* **Стейт доступен только для чтения**: единственный способ изменить стейт - эмитить экшен (*action*) - объект, описывающий, что произошло.
* **Изменения производятся чистыми функциями**: чтобы указать, как экшен трансформирует дерево стейтов, вы описываете чистые редьюсеры (*reducers*).

В действительности **каждое из этих заявлений - ложь**! (Или можно использовать классическую фразу из «Возвращения джедая»: «**они верны... с определенной точки зрения**»).

* «Единственный источник правды» лукавит, потому что (основываясь на FAQ Redux) [вам не нужно выносить всё в Redux](http://redux.js.org/docs/faq/OrganizingState.html#organizing-state-only-redux-state), стор не обязательно должен быть объектом, и вам даже [не нужно иметь один стор](http://redux.js.org/docs/faq/StoreSetup.html#store-setup-multiple-stores).
* «Стейт доступен только для чтения» неверно, потому что нет ничего, что фактически мешает остальной части приложения изменять существующее дерево стейтов.
* И «Изменения производятся чистыми функциями» неверны, потому что функции редьюсера могут также напрямую мутировать дерево стейтов или порождать другие побочные эффекты.

Итак, если эти утверждения не совсем верны, зачем они вообще нужны? **Эти принципы не являются фиксированными правилами или буквальными утверждениями о реализации Redux**. Скорее, **они декларируют то, как должен использоваться Redux**.

Эта тема будет продолжена на протяжении всей нашей дискуссии. Поскольку Redux является такой маленькой библиотекой, он очень мало требует и обеспечивает на техническом уровне. Это вызывает ценную дискуссию, на которую стоит обратить внимание.

### «Язык» и «Метаязык»

В выступлении Cheng Lou на ReactConf 2017 «[Укрощение метаязыка](https://www.youtube.com/watch?v=_0T5OSSzxms&list=PLb0IAmt7-GS3fZ46IGFirdqKTIxlws7e0&index=34)» он описал, что **только исходный код является «языком», а всё остальное (комментарии, тесты, документация, учебные пособия, сообщения в блогах и конференции) - «метаязыки»**. Другими словами, сам по себе исходный код может сообщить только определенное количество информации. Необходимо много дополнительных уровней передачи информации на уровне человека, чтобы помочь ему понять «язык».

Затем Cheng Lou продолжил рассуждать о том, как закреплять дополнительные концепции в самом языке программирования, позволяя выражать больше информации с помощью исходного кода, не прибегая к использованию «метаязыков» для этого. С этой точки зрения **Redux - это крошечный «язык», и почти вся информация о том, как его использовать, на самом деле является «метаязыком»**.

«Язык» (в данном случае, основная библиотека Redux) имеет минимальную выразительность, поэтому концепции, нормы и идеи, окружающие Redux, находятся на уровне «метаязыков» (фактически, статья «[Понимание "Укрощения языка"](http://frantic.im/meta-language)», раскрывающая идеи доклада Cheng Lou, берёт Redux как конкретный пример этих идей). В конечном счете, это означает, что **понимание, почему существуют некоторые практики в Redux, и принятие решения, что является и не является «идиоматическим», будут включать в себя мнения и обсуждения, а не просто определение на основе исходного кода**.

## Как работает Redux на самом деле

Прежде чем мы действительно углубимся в философскую сторону вопроса, важно понять, какие технические ожидания у Redux действительно есть. Взгляд на внутренности и реализацию довольно информативен.

### Ядро Redux: `createStore`

Функция `createStore` - ядром функциональности Redux. Если мы исключим комментарии, проверку ошибок и код дополнительной функциональности (усилителей (*enhancers*) и наблюдателей (*observables*)), то получим вот такую `createStore` (пример кода, заимствованный из учебника под названием [Hacking Redux](http://paulserraino.com/javascript/2016/02/16/hacking-redux.html)):

```js
function createStore(reducer) {
    var state;
    var listeners = []

    function getState() {
        return state
    }

    function subscribe(listener) {
        listeners.push(listener)
        return unsubscribe() {
            var index = listeners.indexOf(listener)
            listeners.splice(index, 1)
        }
    }

    function dispatch(action) {
        state = reducer(state, action)
        listeners.forEach(listener => listener())
    }

    dispatch({})

    return { dispatch, subscribe, getState }
}
```

Это примерно 25 строк кода, но в нём присутствуют ключевые функции. Он отслеживает текущее значение стейта и несколько подписчиков, обновляет значение и уведомляет подписчиков при диспатче (*отправе*) экшена, а также предоставляет API стора.

Рассмотрим всё, что этот фрагмент не включает:

* Неизменность
* «Чистые функции»
* Промежуточные обработчики (*middleware*)
* Нормализацию
* Селекторы
* Thunks
* Саги (*sagas*)
* Должны ли типы экшенов быть строками или символами, и должны ли они определяться в качестве констант или встроенных строк
* Должны ли вы использовать ActionCreators для создания экшенов
* Должен ли стор содержать несериализуемые элементы, такие как обещания или экземпляры классов
* Должны ли данные помещаться в стор нормализованными или вложенными
* Где должна размещаться асинхронная логика

В этом ключе стоит процитировать [пулл-реквест Дэна Абрамова на пример «Counter Vanilla»](https://github.com/reactjs/redux/pull/1289):

> Новый пример Counter Vanilla направлен на то, чтобы развеять миф о том, что Redux требует Webpack, React, горячую перезагрузку, саги, ActionCreators, константы, Babel, npm, модули CSS, декораторы, свободное владение латынью, подписки Egghead, докторскую степень или уровень Превышения ожидания О.У.М. (*некие шутки из фандома Гарри Поттера, - прим. пер.*). Нет, это всего лишь HTML, несколько тегов script и старые-добрые DOM-манипуляции. Наслаждайтесь!

Функция `dispatch` внутри `createStore` вызывает функцию редьюсера и сохраняет **любое возвращаемое им значение**. Не смотря на это, идеи из вышеуказанного списка имеют широкое признание и любое хорошее Redux-приложение должно быть основанно на них.

Перечислив всё, о чём `createStore` не беспокоится, важно отметить, что она всё-таки требует. Реальная функция `createStore` принуждает к соблюдению двух ограничений: **экшены, достигающие стора, должны быть простыми объектами** и **экшены должны иметь свойство `type` со значением, отличным от `undefined`**.

Оба эти ограничения берут своё начало в первоначальной концепции Flux-архитектуры. Процитируем раздел [Flux Actions and the Dispatcher](Flux Actions and the Dispatcher) документации Flux:

> Когда новые данные поступают в систему, будь то от человека, взаимодействующего с приложением, или через вызов web API, эти данные упаковываются в экшен - литерал объекта, содержащий новые поля данных и конкретный тип экшена. Мы часто создаем библиотеку вспомогательных методов, называемых ActionCreators, которые не только создают объект экшена, но и передают его диспатчеру (*dispatcher*).

> Различные экшены идентифицируются атрибутом типа. Когда все сторы получают экшены, они обычно используют этот атрибут, чтобы определить, должны ли и как реагировать на него. В Flux-приложениях и сторы, и представления контролируют себя; они не подвластны воздействию внешних объектов. Экшены поступают в стор через функции обратного вызова, которые они определяют и фиксируют, а не через сеттеры (*методы настройки*).

Для Redux первоначально не требовалось специальное свойство `type`, но позже проверка была добавлена, чтобы помочь уловить возможные опечатки или неправильный импорт констант экшенов и избежать [закона тривиальности](https://ru.wikipedia.org/wiki/Закон_тривиальности) относительно базовой структуры объектов экшенов.

### Встроенная утилита: `combineReducers`

Здесь мы наконец-то начинаем видеть ограничения, знакомые большинству. `combineReducers` ожидает, что каждый редьюсер «правильно» отработает неизвестный экшен, вернув стейт по умолчанию и никогда не вернет `undefined`. Он также ожидает, что текущее значение состояния является простым JS-объектом и что существует точное соответствие между ключами в текущем объекте состояния и объекте функций редуктора. Наконец, он ссылается на сравнения сравнений, чтобы увидеть, вернули ли все редукторы срезов прежние значения. Если все возвращаемые значения кажутся одинаковыми, предполагается, что ничего не изменилось нигде, и оно возвращает исходный объект состояния root как потенциальную оптимизацию.

---

*Слушайте наш подкаст в [iTunes](https://itunes.apple.com/ru/podcast/девшахта/id1226773343) и [SoundCloud](https://soundcloud.com/devschacht), читайте нас на [Medium](https://medium.com/devschacht), контрибьютьте на [GitHub](https://github.com/devSchacht), общайтесь в [группе Telegram](https://t.me/devSchacht), следите в [Twitter](https://twitter.com/DevSchacht) и [канале Telegram](https://t.me/devSchachtChannel), рекомендуйте в [VK](https://vk.com/devschacht) и [Facebook](https://www.facebook.com/devSchacht).*

[Статья на Medium](https://medium.com/devschacht/ire-aderinokun-asynchronous-functions-101-7bc145afe930)
