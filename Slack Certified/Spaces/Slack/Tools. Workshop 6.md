---
order: 5
---

⤴️ [[💬 Slack]]
🏷️ #slack/development 

# Build your app: Tools
___

>[!info] Required links
>- 🔗 [SlackAPI · GitHub](https://github.com/slackapi)
>- 🔗 [Slack Platform Developer Tools](http://slack.dev/)
>- 🔗 [Bolt Tutorial](https://slack.dev/bolt/tutorial/getting-started)
>- 🔗 [Common ways to build Slack apps | Slack](https://api.slack.com/best-practices/blueprints)
>- 🔗 [Redirecting to https://slack.dev/bolt-js/tutorial/getting-started](https://slack.dev/bolt/tutorial/getting-started)
>- 🔗 [Hello World, Bolt ⚡️ - Building your very first app with Bolt | Slack](https://api.slack.com/tutorials/hello-world-bolt)
>- 🔗 [Block Kit | Slack](https://api.slack.com/block-kit)
>- 🔗 [**Take some time to explore Slack Platform Developer Tools**](https://slack.dev/)

## Инструменты
- Bolt Framework - существуют на 3 языках программирования: JavaScript/Java/Python
- First-party SDK - так же, как и bolt существует для JS/Java/Python и устанавливается через пакетные менеджеры этих языков
- Block Kit Builder - web приложения для создания интерфейсов (доступно только для desktop)
- Slack Developer Tools - slack приложение для дебага

## App Blueprints
Проекты приложений доступны на github и бесплатны. Каждый проект приложения включает подробные инструкции, образцы кода и полезную блок-схему.

## SDK
SDK - slack приложение для разработчиков.
- Предоставляет ==набор slash команд== `/sdk`
- Предоставляет возможность просмотреть код сообщений, используя ==message shortcuts==.
- Позволяет посмотреть статус Slack платформы, быстро искать API документацию, смотреть ID бесед и workspace'ов с помощью ==global shortcuts==

> [!warning] Доступ к SDK
> Все пользователи workspace будут иметь доступ к SDK приложению.

### Slash команды SDK
-    `/sdt docs [method name]` - короткий обзор метода
-   `/sdt test_webhook [webhook url]` - отправляет тестовый payload на Slack web hook URL
-   `/sdt` - набор команд и информация о поддержке

> [!tip]+ Key takeaways
> 1. Slack maintains first-party SDKs for three languages: Java, Javascript (via NodeJS) and Python. These tools let you more easily make requests to the Slack APIs and build Slack apps.
> 2. Bolt is Slack’s official development framework that lets you build Slack apps in a flash, and is available in Java and Javascript. You certainly don’t need to use Bolt, but it sure makes things easier.
> 3. Block Kit Builder and Slack Developer Tools can help you prototype Block Kit JSON and inspect those messages, respectively.
> 4. Slack publishes and maintains OpenAPI 2.0 specifications of the Slack Web and Events APIs to improve the platform’s accessibility and predictability.
> 5. Slack Developer Tools, or SDT for short, is a Slack application that helps developers look up useful information such as API documentation and platform status as well as inspect messages and look up user, workspace and Enterprise Grid IDs.

> [!tip]+ **Before moving on, confirm that you can:**
> - Locate and recommend when to use sample code, tools and resources.
> - Recommend when to use Bolt to build Slack apps.
> - Explain how Block Kit Builder and Slack Developer Tools (SDT) work.
