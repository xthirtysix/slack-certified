---
order: 4
---

⤴️ [[💬 Slack]]
🏷️ #slack/development 

# Design for Scale
___

>[!info] Required links
>- 🔗 [Slack for Enterprises | Slack](https://slack.com/help/articles/360004150931-What-is-Slack-Enterprise-Grid)
>- 🔗 [Building apps in Enterprise Grid | Slack](https://api.slack.com/enterprise/grid)
>- 🔗 [Supporting and developing apps in Enterprise Grid | Slack](https://api.slack.com/enterprise/grid/developing)
>- 🔗 [Developing and Testing on Enterprise Grid | Slack](https://api.slack.com/enterprise/grid/testing)
>- 🔗 [Slack Connect: working with channels between organizations | Slack](https://api.slack.com/enterprise/channels-between-orgs)

## Enterprise Grid
Slack Enterprise Grid - это сеть из 2 и более workspace, работающих на одном орге. Такие workspaces связаны через поиск, личные сообщения и директорию, которая шарится между всеми workspace внутри Enterprise Grid.

### Глоссарий
| Термин                       | Определение                                                                                                                                           |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Enterprise org               | Сущность, содержащая несколько workspace. Когда клиент использует Enterprise Grid, все пользователи и личные сообщения хранятся на уровне организации |
| Enterprise organization user | Пользователи Enterprise Grid. У них один ID и профиль во всех workspace                                                                               |
| Enterprise User ID           | Пользовательский ID, начинающийся с `W` или `U`, представляет пользователя во всех workspace внутри Enterprise Grid                                   |
| Legacy User ID               | Обычные пользоватьельские ID. Начинаются с `U`                                                                                                        |
| Shared channel               | Канал, который шарится между workspace внутри орга                                                                                                    |
| Translation layer            | Прозрачно трансформирует organization-based пользовательские ID в Legacy user ID. Нужен для миграции данных                                           |

## Поддержка Enterprise Grid в вашем приложении
| API                      | Стратегия поддержки                                                                                                                              |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| Все Slack приложения     | - Поддерживайте global ID и используйте метод `migration.exchange`                                                                               |
|                          | - Поддерживайте пользователей из других workspace внутри одного Enterprise Grid                                                                  |
| Events API               | - Поддерживайте ивенты и сообщения, содержащие глобальные ID                                                                                     |
|                          | - Поддерживайте пользователей из Shared Channels                                                                                                 |
| Web API                  | - Поддерживайте global ID и используйте метод `migration.exchange`                                                                               |
|                          | - Поддерживайте пользователей из Shared Channels                                                                                                 |
|                          | - Используйте метод `users.info` чтобы получить дополнительную информацию о пользователях по cross-workspace User ID, не найденым в `users.list` |
| Slash команды            | - Включите entity resolution                                                                                                                     |
|                          | - Поддерживайте global ID и используйте метод `migration.exchange`                                                                               |
|                          | - Запускаются только пользователями, которые состоят в workspace с установленным приложением                                                     |
| Интерактивные компоненты | - Обрабатывайте запуск действий пользователями из других workspace                                                                               |
|                          | - Поддерживайте global ID и используйте метод `migration.exchange`                                                                                                                                                 |

### Как подготовить приложение к Enterprise Grid
1. Сохраняйте значения поля `enterprise_id`, которое можно посмотреть методах `auth.test` и `channels.list`
2. Подготовьте интеграцию или бот для использования в Shared Channels. Используйте `users.info` метод, чтобы получить информацию о пользователях не принадлежащих к workspace
3. Если ваш бот установлен в 3 workspace на орге, вы приглашены на shared channel с участниками из всех 3 workspace, вы получите сообщение три раза, по одному разу для каждого workspace. Ваш бот должен отвечать только на одно из доставленных сообщений.
4. Если ваше приложение является частью каталога, сообщите нам, что вы подготовили рабочие пространства Enterprise Grid. Мы проведем дополнительное тестирование и предоставим вам отзывы о том, что можно улучшить.

> [!warning] Дубликаты сообщений
>Избегайте дубликатов сообщения (в том числе приватных) - они имеют одинаковое значение в поле `ts`.
>Поле `source_team` содержит workspace из Enterprise Grid, из которого сообщение было отправлено

Не нужно отслеживать дубликаты в `Events API` - мы получим один ивент для всех workspace, для которых он предназначен, а ID этих workspace будут хранится в поле  `authed_teams`.

## Миграция данных
Во время миграции Web API, RTM API, Events API и некоторые другие взаимодействия со Slack приложением могут быть недоступны. Например, взаимодействие через Web API может вернуть ошибку `team_added_to_org`.
Чтобы спланировать миграцию, подпишитесь на события:
	- `grid_migration_started`
	- `grid_migration_finished`
Используя эти события, приложение может остановить и возобновить свою активность.

## User в Enterprise Grid
Объект пользователя содержит дополнительную информацию в Enterprise Grid в поле `enterprise_user`:
-   `id` - ID пользователя, начинающийся с `U` или `W`
-   `enterprise_id` - уникальный ID Enterprise Grid
-   `enterprise_name` - имя орга
-   `is_admin` 
-   `is_owner` 
-   `teams` - массив workspace в которых состоит пользователь

 > [!important] Префиксы в глобальном ID
 > Между `U` и `W` нет ни какой разницы

## Тестирование для  Enterprise Grid
Разработчику нужно запросить создание sandbox для Enterprise Grid в специальной форме. Сообщения в таком sandbox хранятся до 3 дней. Поскольку Enterprise Grid требует Single Sign On, необходимо использовать либо настроенный IdP, либо Slack приложение Simple IdP для автоматической генерации юзера. Можно создать и своих пользователей, но это займет немного больше времени и потребует дополнительных шагов.

## Slack Connect
Slack Connect позволяет шарить каналы с людьми, которые не состоят в вашей организации.
В большинстве случаев расшареные каналы будут работать так же как и обычные, но есть и некоторые соглашения.

- Сообщения и файлы
  Все workspace вовлеченные в connected channel могут читать и отправлять сообщения, а так же получать доступ к истории shared channels
- Настройки канала
  Каналы внутри организации могут иметь разные настройки, в зависимости от сценариев:
  1. Имя канала может отличаться в зависимости от workspace. Всегда сверяйте каналы по ID а не по имени
  2. Доступ к каналам может отличаться в зависимости от workspace (private/public)
  3. Настройки хранения данных даже для одного и того же канала могут отличаться в разных командах. Например, одна организация может не ограничивать срок хранения данных, а другая — 30 днями. Это означает, что одна команда увидит историю канала, а другая нет.

## Поддержка externally shared каналов
1. Определяйте, когда в каналах состоят пользователи из внешних оргов
   Приложение может подписаться на события `channel_shared` и `channel_unshared`. Для этого приложению нужны  `сhannels:read` и `group:read` скоупы.
   Эти ивенты будут содержать ID канала и ID команды с которой канал был или перестал быть расшарен
2. Общение с незнакомцами
   Есть два типа пользователей, которые не состоят в орге, на котором установлено приложение:
   1. External Users
   2. Strangers (`user.info` вернет флаг `is_stranger: true` )
   Для обоих типов пользователей данные профиля не вернут email, даже если у вас есть нужные скоупы, и не вернут информацию о локали даже с переданным  `include_locale` флагом.
3. Разные настройки для одного канала
   В зависимости от настроек, префикс канала может отличаться. Поскольку каждая команда внутри канала может решить что канал будет публичным или приватным, это влияет на API:
   - Методы группы `conversations.*` принимают любые каналы
   - Объект канала теперь включает инфо о нем (private/public etc)
   - `converstaions.info` будет предоставлять инфо о workspace подключенных к каналу и team ID хостящего workspace
   - Объект канала содержит дополнительные поля:
	   - `is_ext_shared`
	   - `is_private`
	   - `is_mpim`
4. Каналы, которые конвертируются обратно в single-workspace channels
   ID сохранится только для host-workspace, в остальных workspace ID каналов изменится, и каналы останутся, предоставляя доступ к истории

> [!tip] Slash команды в externally shared каналах
>  - Работают только в workspace, где установлено приложение
>  - Могут быть запущены только членами оригинального канала, хотя остальные пользователи смогут видеть вывод slash команд

## Стратегии поддержки в зависимости от функционала
| Фича                 | Стратегия                                                                                                                                                             |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Events API           | -   Support events originated from external users in the shared channels.                                                                                             |
|                      | -   No duplicated event triggering between shared channels                                                                                                            |
|                      | -   To see which teams the event is delivered, look for the values of the `authed_teams` property for the response.                                                   |
| Web API              | -   Support external users in shared channels. Some user-related actions will not be permissible due to their external nature                                         |
|                      | -   Use `users.info` to retrieve additional information on cross-team user ID not found in `users.list`.                                                              |
| Incoming webhooks    | -   Messages from incoming webhooks are visible to all members of a shared channel.                                                                                   |
|                      | -   Incoming webhook can send DM only to the users on installed teams.                                                                                                |
| Slash commands       | -   Slash commands can be only invoked by users belonging to the workspace your app is installed.                                                                     |
|                      | -   Turn on entity resolution for mentioned users, allowing you to identity them by id, including on foreign teams.                                                   |
| Message actions      | -   Message actions can be only invoked by users belonging to the workspace your app is installed.                                                                    |
| Interactive messages | -   Handle action invocation by users from other teams, and letting them know if an action is not permissible due to their external nature.                           |
| Unfurls              | -   When a user on the installing workspace posts a link in the shared channel. The link should be unfurled for the entire channel unless there is a privacy concern. |
| Bot users 🤖         | -   Bot users can DM all local users in the workspace they are installed in, and external users with a common shared channel.                                         |
___
> [!tip]+ Key takeaways
> 1. Slack’s Enterprise Grid plan, in addition to having security and compliance features, allows an organization to build a “network” of workspaces all of which exist within a Grid org.
> 2. Multi-workspace channels are channels within an Enterprise Grid that are shared and available to multiple workspaces within the same Enterprise Grid.
> 3. In Enterprise Grid, users have both global and local user IDs. Slack applications installed into an Enterprise Grid will default to receiving the global ID however developers can toggle the **Translate Global IDs** setting off from their [application configuration page](https://api.slack.com/apps).
> 4. Developers can request Enterprise Grid sandbox orgs  for the purpose of testing their apps.
> 5. Organizations will occasionally migrate a workspace (or several) into an Enterprise Grid. There are events types developers may want to subscribe to and considerations to make in terms of shared channels.
> 6. Channels shared with two or more organizations, also known as Slack Connect, are distinctively different from Enterprise Grid multi-workspace channels, and allow multiple organizations on any paid Slack plan to collaborate without leaving Slack.

> [!tip]+ **Before moving on, confirm that you can:**
> - Explain how the features of Enterprise Grid impact your app's design and development.
> - Use appropriate terminology to refer to the fundamental Slack platform concepts.
> - Describe how your application supports multi-workspace channels on an Enterprise Grid org vs channels shared via Slack Connect.
