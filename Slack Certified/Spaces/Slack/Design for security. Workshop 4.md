---
order: 3
---

⤴️ [[💬 Slack]]
🏷️ #slack/development 

# Design for Security. Workshop 4
___
> [!info] Обязательно к прочтению
> - 🔗 [Basic app setup | Slack](https://api.slack.com/authentication/basics#scopes)
> - 🔗 [Slack Certified Developer Prep Course](https://www.slackcertified.com/certified-developer-prep-course/932655/scorm/8i3ugo85wizl)
> - 🔗 [Installing with OAuth | Slack](https://api.slack.com/authentication/oauth-v2)
> - 🔗 [Access tokens | Slack](https://api.slack.com/authentication/token-types)
> - 🔗 [Permission scopes | Slack](https://api.slack.com/scopes)
> - 🔗 [Verifying requests from Slack | Slack](https://api.slack.com/authentication/verifying-requests-from-slack)

## Виды токенов
Токен представляет собой авторизацию приложения, которая предоставляет доступ к данным (к каким именно данным - определяется token scope'ами).
Существует несколько видов токенов, подходящих для разного функционала и некоторые scopes уникальны для того или иного токена.

| Вид токена      | Описание                                                                                                                                                                                                                                                                                   |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Bot token       | Подходит для большинства приложений, с выборочной моделью доступа, чтобы запрашивать только те scopes, которые нужны. В отличии от **User token** не привязаны к идентификатору пользователя. Всегда начинаются с `xoxb-`                                                                  |
| User token      | Эти токены позволяют вам работать напрямую от имени пользователей, когда это необходимо. Действия записи с пользовательскими токенами выполняются как бы самим пользователем. Токены пользователя всегда начинаются с `xoxp-`                                                              |
| App-level token | Это специальные токены, используемые с определенными API, которые относятся к приложению во всех организациях, в которых оно установлено. Эти токены дают вам возможность обрабатывать вещи, относящиеся к вашему приложению в целом. Токены уровня приложения всегда начинаются с `xapp-` |
| Configuration token                | Специальные токены для workspace, используемые с Manifest App APIs для создания и настройки приложений Slack.                                                                                                                                                                                                                                                                                            |

## Scopes
Скоупы определяют набор доступов для вашего приложения. Всегда выбирайте необходимый минимум. Можно настраивать из вкладки **OAuth & Permissions** на странице приложения.
[Скоупы Slack приложения](https://api.slack.com/scopes)

## Установка приложения
Каждый раз, когда пользователь соглашается со скоупами вашего приложения и подтверждает установку, вы получаете токен доступа. ==Иными словами, каждый access token, который вы получаете представляет собой инстанс установленного Slack приложения==.

При разработке приложения можно выбрать 1 из 2 способов его установки:
1. Базовая установка
2. OAuth flow

### Базовая установка
- Выберете скоупы, которые нужны бот-пользователю.
- Установите приложение на single workspace
- Получите bot token для приложения на вкладке **OAuth & Permissions**

### OAuth flow
Если нужно установить приложение на более чем один workspace.
- OAuth позволяет пользователю любого Slack Workspace установить ваше приложение
- В конце OAuth flow ваше приложение получит access token. Этот токен откроет доступ к Slack API методам, events и другим фичам
- В процессе OAuth flow вы должны указать, какие скоупы нужны приложению

Ваше приложение получает acess token в 3 шага:
1. Запрашивает скоупы
2. Ожидает одобрение от пользователя на запрошенные скоупы
3. Обменивает временный авторизационный код на access token

#### Запрос скоупов
Чтобы запросить скоупы перенаправьте пользователя на `https://slack.com/oauth/v2/authorize`, добавьте в кач-ве параметров ClientId и скоупы в виде  `scope=incoming-webhook,command`. Конечная ссылка выглядит примерно так:
```ruby
https://slack.com/oauth/v2/authorize?scope=incoming-webhook,commands&client_id=3336676.569200954261
```
Параметр `scope` отвечает за скоупы bot-user'а. Если вам нужны user scopes замените этот параметр (или добавьте новый) на `user_scope`.

>[!warning] Progressive permissions
>Можно добавить новые скоупы, но нельзя удалить старые без отзыва токена

Вы так же должны сообщить Slack куда отправить авторизационный код. Добавьте в параметра `redirect_uri` ссылку с ==протоколом https==.
Так же можно включить необязательный параметр  `state` - передайте уникальное для пользователя значение, и проверьте его, когда получите ответ.

#### Ожидание одобрения
☕️ Просто ждите )

Распарсите ответ, который пришел на указанный `redirect_uri`, и отправьте авторизационный токен на `oauth.v2.acess`:
```sh
curl -F code=1234 -F client_id=3336676.569200954261 -F client_secret=ABCDEFGH https://slack.com/api/oauth.v2.access
```

После этого Slack пришлет ответ в котором содержится access token и другая информация. Если вы запрашивали user scopes, они будут находиться за ключом `authed_user`.


![[Pasted image 20230216211903.png]]

>[!tip] Ротация токенов
>Благодаря OAuth flow вы можете использовать ротацию токенов в качестве дополнительного уровня безопасности для своих токенов доступа. Без ротации токенов срок действия токена доступа никогда не истекает. При ротации токенов срок его действия истекает каждые 12 часов.

## Отзыв токена
Отзыв токена происходит через `auth.revoke` метод. После отзыва:
- Bot token больше не работает
- Пользователь бота удален из workspace
- Slash команды, связанные с  bot token, будут удалены из workspace, если нет пользовательских токенов для того же приложения, которые несут область действия команд
- Входящие веб-хуки, которые были установлены и связаны с bot token, будут удалены
- Если пользовательские токены для того же приложения не существуют, приложение будет удалено из workspace

## Refresh token
>[!warning] Ротацию токена нельзя отключить после включения

Для обновления токена, используется метод `oauth.v2.exchange` - он требует Client Id и Client secret. В ответе будет содержаться `access_token`, который теперь будет иметь префикс `xoxe-`, так же в ответе будет поле `expires_in` показывающее через сколько истечет действие токена.

>[!info] Когда использовать User token
> - Если вам нужно действовать от имени пользователя
> - Если вам нужны специфичные API, которые используют user token (Slack’s SCIM API, Audit Logs API, некоторые admin API)

## Ключевые преимущества точечных прав
-   **🧑🏻‍🤝‍🧑🏽** Повышена надежность приложения
-   **📶** Гибкость при растущей функциональности приложения
-   **🔐** Больше точности и меньше ограничений для администраторов, заботящихся о безопасности

## Уровни токена
- Workspace level - работает на 1 workspace внутри орга
- Org level - работает на множество workspace внутри орга

### Преимущества токенов на уровне организации

Токены уровня организации позволяют использовать такие API, как:
- **Slack’s Discovery API** (варианты использования для eDiscovery и DLP)(Обратитесь в службу поддержки Slack, чтобы включить эту функцию и добавить области действия в ваш экземпляр Slack.)
- **Audit Logs API** (вариант использования для мониторинга рабочей области)
- **API, связанные с администрированием** (случай создания приложений для администраторов Slack)
- **SCIM API** (вариант использования для изменения пользователей)

Скоупы для org-level токенов будут добавлены в user token и будут иметь корень  `admin:*` (или `discovery:*` или `auditlogs:*`).

==Для `admin:*` скоупов необходимо, чтобы OAuth flow был инициирован Enterprise Grid admin или owner'ом, и их установка возможна только на Enterprise Grid org, а не на индивидуальный workspace, используя workspace switcher в процессе установки==.

## Верификация Slack запросов

### Подписание запросов
Slack подписывает запросы специальным ключом (secret), уникальным для вашего приложения. ==В каждый запрос Slack добавляет `X-Slack-Signature` header==. Такая подпись генерируется комбинацией signing secret'а и тела запроса в дальнейшем зашифрованной 256-битным шифрованием.

#### Как подписать запрос в 4 простых шага:
- Возьмите `X-Slack-Request-Timestamp` и тело запроса
- Конкатинируйте строку из версии запроса, timestamp'а из заголовка и тела запроса разделенные символом `:`. Результат будет выглядеть примерно так:
  ```ruby
v0:123456789:command=/weather&text=94070
```
- С помощью HMAC SHA256 шифрования зашифруйте полученную строку, используя Slack `Signing Secret` в качестве ключа
- Сравните полученную строку с  `X-Slack-Signature` в запросе

## Mutual TLS

Запросы от Slack так же могут быть аутентифицированы через Mutual TLS. Отличия Mutual TLS от подписания запросов в том где и как это происходит. В отличии от верификации подписи в вашем приложении, вы настраиваете ваш TLS-сервер, чтобы аутентифицировать клиентские сертификаты Slack.

- Ваш TLS-сервер получает сертификат
- Достает определенный заголовок (1 из 2 по выбору)
- Вставляет содержимое заголовка в header запроса `X-Client-Certificate-SAN`
- Ваше приложение проверяет, что header - это `platform-tls-client.slack.com`


## 5 дополнительных Security соглашений
1. Шифрование
   ==Slack требует, использования HTTPS протокола и веб-сервер с установленным TLS 1.2 сертификатом==
2. Client ID и Client Secret
   Client ID может использоваться кодом и отправлен через email.
   Client Secret не должен быть передан в клиентский код, через email или содержаться в публичных репозиториях.
3. Ограничение токена по IP адресам
   ==Работает только для Single Workspace и Web API==
   Можно добавить пул IP адресов (до 10 единичных или набор в CIDR формате `101.101.101.106/32.`) запросы с которых будут одобрены.
   На остальные адреса будет приходить ответ:
   ```json
{
  "ok": false,
  "error": "invalid_auth"
}
```  
4. Безопасное хранение токенов
   - Подумайте, действительно ли вам нужен конкретный токен, чтобы ваше приложение работало. Если нет, будьте осторожны и не храните больше секретов клиентов, чем вам нужно.
   - Попросите наименьшее количество привилегий для вашего токена. Большинству приложений достаточно выполнить всего несколько действий с токеном. Убедитесь, что вы запрашиваете максимально ограниченные возможности для защиты ваших клиентов в случае взлома.
5. Все про URL
   - **URL входящих веб-хуков**
     Убедитесь, что не храните такие URL публично. Ограничения по IP не действуют на веб-хуки.
   - **Redirect URL**
     Ои безопасны для публикации в коде. Однако убедитесь, что redirect URL-адреса, определенные в общедоступном приложении, ограничены только доменными именами, находящимися под вашим непосредственным контролем.

___
> [!tip]+ Key takeaways
> 1. One of the ways we ensure data in Slack is secured for developers and users is through authentication.
> 2. OAuth is a protocol that lets your app request authorization to private details in a user's Slack account without getting their password and is used for installing apps on a workspace or org.
> 3. If you intend to install your app on two or more workspaces, you’ll need to build an OAuth installation flow, which includes three steps: asking for scopes; waiting for a user to approve your requested scopes; and exchanging a temporary code for an access token.
> 4. At the end of the OAuth flow, your app gains an access token, which opens the door to Slack API methods, events and other features.
> 5. During the OAuth flow, you specify which scopes your app needs. Those scopes determine exactly which methods, events and features your app can access.
> 6. The benefits of granular permissions include: more precision and fewer restrictions for security-conscious admins; improved app reliability; and flexibility to add incremental functionality to your app.
> 7. With the help of signing secrets and/or Mutual TLS, your app can more confidently verify whether requests sent from Slack are authentic.
> 8. Build confidence in the sanctity of your user tokens and bot tokens by storing them in a way that directly links them to the token owner (workspace or user).
> 9. Think about the security of your data, including different risks and mitigations, in the context of the layers of the OSI model.

> [!tip]+ **Before moving on, confirm that you can:**
> - Implement the OAuth installation flow.
> - Describe each step in the OAuth flow.
> - Describe how Client IDs and Client Secrets are used in the OAuth flow.
> - Explain the function and importance of app scopes.
> - Define the principle of least privilege.
> - List each token type that can be generated.
> - Explain how to generate tokens of various types and scopes.
> - Explain how to verify that a callback sent to your app has originated from Slack.
> - Identify the steps you could take to make your app communicate securely in Slack.
> - Explain how to restrict token use by IP address.
> - Describe how your application should safely store data.
> - Can identify basic do's and don'ts of the OSI security model.