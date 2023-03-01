---
order: 1
---

⤴️ [[💬 Slack]]
🏷️ #slack/development #slack/in-progress

# Design your workflows: Workshop 2
___
## Опишите свои варианты использования
Варианты вопросов для сбора фидбэка и идеи:
- *Есть ли сообщения, которые постят регулярно?*
- *Попросите члена команды найти пару e-mail сообщений, которые они получали за последнюю неделю, и они были важны для их работы*
- *Задумайтесь об интерактивности приложения*
- *Подумайте об основных препятствиях, с которыми люди сталкиваются при использовании Slack в вашей команде*, и опишите их. Может ли приложение решить эти проблемы?
- *Как часто вы используете внешние сервисы в работе?* - если у них нет своей интеграции, и они предлагают web API, разработайте приложение чтобы интегрировать сервис в Slack

## Планируя взаимодействия
### Флоу взаимодействий
Как происходят взаимодействия между пользователем и приложением?
- Что-то триггерит взаимодействие
- Приложение генерирует ответ на этот триггер
Эти триггеры могут варьироваться от более пассивных ивентов, как schedules или внешних сервисов, до прямых действий пользователя.
Пользователи могут вызывать приложения, используя одну из нескольких точек входа.
### Источники взаимодействия
- *Shortcuts* добавляют действия в интерфейс Slack
- Интерактивные компоненты *Block Kit* внедряют интерактивность в Surfaces приложения
- *Слэш команды* - команды, которые пользователь может вызвать из поля ввода чата

Какую же точку входа выбрать? Если вы знаете что ваша аудитория плохо знакома с интерактивностью,  *слэш команды* могут быть сложны для них. В большинстве случаев ответ будет - "*все из них*" - ваше приложение решает конкретные задачи через наиболее подходящую точку входа. Также следует учитывать какую информацию нужно передать приложению; если необходим полный контекст сообщения пользователя, *message shortcuts* - идеальный вариант.

## Способы взаимодействия
1. Shortcuts
   Бывают global и message. Приложение может иметь максимум 5 shortcuts каждого вида. ==Для их использования, у приложения должен быть `commands` скоуп==
2. Slash commands
   Набираются из инпута сообщений, после ввода команды  `/`. Если при создании включить опцию **Escape channels, users, and links sent to your app** ==то в payload'е будут приходить id соответствующих сущностей, обернутые в `<>`==
3. Взаимодействия сгенерированные внешними сервисами
4. Взаимодействия по таймеру или отложенные взаимодействия
5. Slack [[Events API]]
6. Chained взаимодействия

> [!warning] У slash-команд нет неймспейсов
   > Если происходит конфликт имен, сработает команда из приложения, установленного позже
   
>[!info] Тип response сообщений
>По умолчанию, сообщения имеют тип `ephemeral`, но лучше всегда указывать его явно, записав свойство `response_type`

>[!info] Тип взаимодействия
>В каждом payload от взаимодействия содержится поле `type`. Возможные значения `type`:
>-   [`block_actions` payloads](https://api.slack.com/reference/interaction-payloads/block-actions) are received when a user clicks a [Block Kit interactive component](https://api.slack.com/interactivity/components)
>-   [`shortcut` and `message_actions` payloads](https://api.slack.com/reference/interaction-payloads/shortcuts) are received when [global and message shortcuts](https://api.slack.com/interactivity/shortcuts) are used
>-   [`view_submission` payloads](https://api.slack.com/reference/interaction-payloads/views) are received when a [modal is submitted](https://api.slack.com/surfaces/modals/using#handling_submissions)
>-   [`view_closed` payloads](https://api.slack.com/reference/interaction-payloads/views) are received when a [modal is cancelled](https://api.slack.com/surfaces/modals/using#modal_cancellations).

### Ответы на взаимодействие
> [!warning] Время на ответ - 3 секунды
> Необходимо отправить хотя-бы пустой ответ со статусом `200` в течении 3 секунд, иначе взаимодействие выкинет ошибку

-   [**Acknowledgment response**](https://api.slack.com/interactivity/handling#acknowledgment_response) - a required, immediate response that confirms your app received the payload.
-   [**Message responses**](https://api.slack.com/interactivity/handling#message_responses) - optional responses that use a special URL from the payload to publish messages.
-   [**Modal responses**](https://api.slack.com/interactivity/handling#modal_responses) - optional responses that use a short-lived code from the payload to invoke a modal.
-   [**Asynchronous responses**](https://api.slack.com/interactivity/handling#async_responses) - optional responses that are inspired by the info in the payload, but not directly related.

## Link Unfurl
Для link unfurling'а нужно зарегистрировать домен (лимит - 5 доменов). Владельцем домена должна быть компания/разработчик.
Необходимо подписаться на `link_shared` event, и включить `link:read`, `link:write` скоупы.

## Выбираем правильный API

### Event API
С этим API вы выбираете событие, которое хотите получить и Slack отправляет его на ваш эндпоинт.

### Web API
Если Event API пингует ваш эндпоинт, то [[Slack Web API|Web API]] - это обратный процесс - запрос от приложения к Slack. Используется когда нужно сделать что-то в Slack workspace из вашего приложения - например, создать беседу или отправить сообщение, или когда нужно запросить что-то напрямую - например, список пользователей

### Conversation API
Это подмножество [[Slack Web API|Web API]]. Эти функции работают с channel-like объектами - чаты, беседы, DM и так далее. В большинстве своем эти объекты очень похожи, но будут иметь свои значения атрибутов - насколько они приватны, шарятся ли между workspace'ами  и т.д.

>[!IMPORTANT]+ Типы каналов
>- *Public каналы*
> 	 - открыты всем членам workspace'а
> 	 - все что публикуется здесь, доступно для поиска даже не членам канала
> 	 - по умолчанию все члены workspace'а, кроме гостей, могут создавать public каналы
> 	 - должны быть местом для работы по умолчанию
>- *Private каналы*
>	-  члены канала должны быть приглашены в такие каналы
>	- по умолчанию все члены workspace'а (кроме Single-Channel Guests) могут создавать приватные каналы
>	- приватные каналы не могут быть сконвертированы в публичные
>- *Каналы расшареные с внешними оргами*
>	- вы можете пригласить до 19 внешних организаций в канал
>	- члены одного расшареного канала могут
>		- читать и писать сообщения
>		- обмениваться DM
>		- пользоваться приложениями
>		- загружать файлы
>	- могут быть public или private
>	- все, кроме гостей, могут отправлять инвайты. чтобы инвайт сработал, нужен approve от админа с правами менеджера канала
>	- приглашенные орги могут просмотреть историю сообщений, членов канала и ботов
>-  *Multi-workspace каналы* (Только для Enterprise Grid плана)
>	- соединяют несколько workspace'ов внутри Enterprise Grid организации
>	- могут быть public или private
>	- org owners и admins могут создавать такие каналы, и разрешить членам орга делать это
>	- добавляйте и двигайте канал в любой workspace (кроме general канала)
>	- могут быть org-wide (доступны всем пользователям в орге), или настроены как дефолтный канал 
>- *Приватные сообщения (DM) и групповые приватные сообщения*
>	- Приватные сообщения - тет-а-тет беседы, доступные всем пользователям
>	- Групповые приватные сообщения - беседы вне канала между максимум 9 пользователями
>		- когда вы приглашаете кого-то, новая беседа будет создана, без доступа к истории сообщений
>		- чтобы сохранить историю сообщений перед приглашением, вы можете конвертировать такую беседу в приватный канал
>		- невозможно покинуть такую беседу - вам нужно попросить его членов создать новую беседу без вас
>		- Single-Channel Guests могут быть членами таких бесед, но не могут создавать новые

### Socket Mode
Используйте *Event API* и интерактивные возможности платформы без открытия статичных HTTP эндпоинтов - используя WebSocket протокол

### Real Time Messaging
RTM API - устаревший способ подключения приложения к Slack. Если не подходит статичный HTTP эндпоинт, предпочтительнее использовать WebSocket

## Коммуникация с пользователями
Сообщения - основной способ коммуникации в Slack. Приложения тоже используют сообщения.
Но сообщения не должны быть обычным текстом. Они могут содержать Rich Text или сложные интерфейсные решения. За реализацию таких сообщений отвечает UI фреймворк *Block Kit*. С помощью блоков можно создавать *workflows* прямо в Slack, которые одинаково хорошо работают как на десктопе так и в мобильном приложении.

Существует 2 способа отправить сообщение в Slack:
1. Используя web-hook - может только отправлять сообщения в определенный канал, определенный при установке приложения
2. Испльзуя `chat.postMessage` API метод
  - может постить в любой канал к которому имеет доступ, включая треды индивидуальных сообщений
  - отправлять сложные интерфейсные сообщения, построенные с помощью *Block Kit*
  - получать ответ, чтобы построить интерактивность в приложении

**Block Kit** - позволяет строить сложные UI. Можно создавать интерфейсы, форматируя JSON, или используя *Block Kit Builder*, который генерирует JSON для вас.

## Лимиты
Web API имеет 4 тира лимитов: 1, 20, 50 и 100 запросов в минуту, и *особый* тир, уникальный для метода. ==По исчерпанию лимитов вы получите response `HTTP 429 Too Many Requests` и `Retry-After` header==, содержащий в себе кол-во секунд, через который можно повторить запрос.

Events API лимитирован в 30'000 ивентов на workspace в час.

### Key takeaways
-   Before you design your app workflows, identify how it will solve a need and for whom. This will help you avoid duplicating existing apps and anchor your interactivity flows to specific use cases.
    
-  The interactive flow of your app is generally structured by a trigger and your app’s response to that action.
    
-   Your app’s interactions can be triggered by various origin or entry points, including user action, external services, the Events API, a schedule/timer or other interactions.
    
-    Your app's response to an interactive payload can be anything from a simple acknowledgement that a message has been received to consuming an API.
    
-    The Slack platform provides a range of component features for different use cases, including surfaces such as App Home, modals and messages; shortcuts, slash commands; Block Kit interactive components; incoming webhooks; bots; and app unfurling.
    
-    Slack's core APIs underpin most of Slack's functionality and are the APIs you will use in most cases. These include the Web API and the Events API.
    
-    In the Web API suite, the Slack Conversations API provides your app with a unified interface to work with the different types of channel-like objects as conversations.
    
-    Slack provides a range of other APIs (Admin APIs, SCIM APIs, Audit Log APIs and Status API) for niche use cases related to automating the administration of workspaces and orgs.
    
-   The RTM is an outmoded API that should only be used if your app's hosting infrastructure cannot connect to Slack over HTTP via the public Internet and instead needs to leverage a WebSockets connection.
    
-  Rate limits are different depending on which API endpoint you use. In all cases, if your app receives an HTTP 429 from Slack, it will indicate how long to wait before trying again.

### Before moving on, confirm that you can:

-   List common workflows and use cases for Slack apps.
    
-   Explain how Slack interacts with your application architecture.
    
-   Describe the flow of data in the interactivity lifecycle.
    
-    List and define the possible user entry points for an app's workflow.
    
-    Recommend when to use different Slack components and features (such as incoming webhooks, slash commands, shortcuts, interactive components, unfurling and bots) for common use cases.
    
-   Implement link unfurling in apps.
    
-   Build a slash command.
    
-   Configure a message shortcut.
    
-    Add a bot.
    
-   Integrate support for interactive components.
    
-   List and define the use cases for Slack different APIs.  
    
-   Demonstrate the ability to subscribe to an event using the Events API.
    
-    Describe the use case and limitations of using the RTM API.
    
-   Summarize how rate limits are handled for the different APIs.