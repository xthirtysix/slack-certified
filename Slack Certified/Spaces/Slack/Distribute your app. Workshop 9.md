---
order: 7
---

⤴️ [[💬 Slack]]
🏷️ #slack/development 

# Distribute your app 
___

## Single workspace app
Любое приложение изначально создается как Single Workspace App. Оно привязано только к одному workspace (даже если это dev workspace), и доступно для любого плана Slack.

При установке такого приложения вы получите OAuth access token, который необходимо связать с соответствующим workspace id.

## Распространите свое приложение на другие workspace
Прежде чем вы сможете установить приложение на другие workspaces, нужно подготовить некоторые вещи:
- Убедитесь, что ваше приложение поддерживает OAuth 2.0 flow
- Включите SSL по всем направлениям.
  Для распределенных приложений требуется, чтобы все URL-адреса, включая URL-адреса перенаправления OAuth 2.0, запроса API событий, интерактивности, загрузки параметров и команды Slash, использовали защищенный протокол HTTP+SSL (HTTPS). Вам необходимо убедиться, что все URL-адреса, настроенные в вашем приложении, используют https://, прежде чем вы сможете активировать общедоступное распространение.

Если публичное распространение включено, вы получите доступ к сгенерированному коду кнопку *Add to Slack*, которая доступна и для single и для multiple workspace приложений

## Публикация приложения в Slack App Directory
Для публикации в Slack App Directory, должен быть включен Public Distribution.
Приложение будет проверено на соблюдение [гайдлайнов](https://api.slack.com/start/distributing/guidelines)
Убедитесь, что прошли [чеклист](https://api.slack.com/reference/slack-apps/directory-submission-checklist) на подготовку приложения к публикации.

Не забудьте предоставить информацию о безопасности и соответствии требованиям при подаче заявки. Администраторы Slack, которым поручено оценивать риски приложений для своей организации, часто принимают во внимание эту информацию.

Одно из решений, которое вы примете в процессе отправки, — решить, хотите ли вы, чтобы ваш список каталогов приложений ссылался на landing page или URL-адрес прямой установки:
- Landing page: кнопка «Установить» на странице со списком вашего приложения отправляет пользователя на страницу вашего сайта, где вы размещаете кнопку «Добавить в Slack» вашего приложения. Вам (разработчику) нужно будет добавить код, чтобы отобразить кнопку «Добавить в Slack» на вашем веб-сайте.
- Когда пользователь нажимает кнопку, он запускает поток OAuth 2.0.
- Прямая установка (обязательно): кнопка установки на странице вашего списка сразу отправляет пользователя к процессу установки OAuth 2.0. Вы (разработчик) должны предоставить URL-адрес, который перенаправляет на https://slack.com/oauth/authorize.

### После попадания в Slack App Directory
1. Вы несете ответственность перед своими пользователями за актуальность и безопасность вашего приложения.
2. Как и другие создаваемые вами приложения, приложения Slack следует тестировать в среде разработки, прежде чем вносить изменения в производственное развертывание и обновлять список каталогов.
3. Обратите внимание, что любые изменения в настройках приложения потребуют повторной подачи заявки.

### Тестовое приложение
При внесении изменений, стоит протестировать функциональность в тестовом приложении.
==Когда тестовое приложение **не нужно**:==
-   Обновление визуальной информации (имя приложения, описание, иконка)
-   Изменение цены и языков приложения
-   Добавление еще одного экземпляра существующей функциональности
==Когда тестовое приложение необходимо:==
- Включение новых функций в ваше приложение
- Переключение на вкладки «Сообщения» или «Главная страница приложения» в первый раз
- Добавление новых скоупов в ваше приложение

После того, как тестовое приложение тщательно протестировано, его нужно отправить на resubmit и approval.

### App Suggestion
Вы можете встроить html код в мета-тэг страницы, которая ассоциирована с вашим приложением. При отправки ссылок в чаты Slack, бот предложит установить это приложение.