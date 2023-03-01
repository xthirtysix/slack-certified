---
order: 2
---

⤴️ [[💬 Slack]]
🏷️ #slack/development 

# Design a great UX. Workshop 3
___
Вещи о которых стоит помнить при разработке UX:
1. Временные зоны
   >[!info] Обработка временных зон
   >Чтобы получить временную зону пользователя, приложение может обратиться на эндпоинт `user.info`. В полях `tz` и `tz_offset` содержится нужная информация
2. Языки
   Локаль назначается пользователю и каналу. Убедитесь, что отправляете сообщение в наиболее привычной для канала/пользователя локали.
   > [!info] Обработка локали
   > Установив параметр `include_locale` в значение `true` для методов `users.info` и `converstions.info` вы получите ответ с полем `locale` в IETF формате (напрмер  `en-US`)
3. Размер команды
   Обратите внимание как приложение работает со списками пользователей/каналов. Возможно такие списки стоит разбить на несколько частей.
4. Роли и доступы
   Помимо админов, владельцев и членов каналов есть еще и гости, чьи доступы ограничены.
   Типы гостей:
   - **Single-Channel Guests** - ==в ответе `users.info` для такие гостей будет поле `is_ultra_restricted=true`.== Могут быть добавлены только в 1 канал, не используют слэш-команды и shortcuts
   - **Multi-Channel Guests** - ==в ответе `users.info` для такие гостей будет поле `is_restricted=true`.== В зависимости от workspace, могут быть лишены возможности использовать слэш-команды и shortcuts

## UX дизайн для Surfaces
### Home Tab
1. Отображайте персонализированную информацию
2. Создавайте визуальную иерархию (самый важный контент должен находиться вверху)
3. Предоставьте меню настроек (лучше прятать их за кнопкой)
4. Призывайте к действию (самый важный контент должен быть заметен и доступен)
### Modals
1. Не перегружайте модальные окна (используйте несколько view вместо одного, не создавайте более 6 инпутов в 1 view)
2. Обозначьте результат (пользователь должен понимать что произойдет по submit'у)
3. Показывайте прогресс
4. Не спрашивайте логин информацию
### Shortcuts
1. Начинайте имя shotcut'а с глагола
2. Показывайте модальные окна при запуске shortcut
3. Кастомизируйте shortcut в зависимости от контекста (message/global)
4. Включайте пользовательское подтверждение результата (предоставьте пользователю подтверждение и подробные сведения о том, что он только что сделал. А для стороннего сервиса включите соответствующие ссылки на ваш сервис)

>[!tip] Укажите `default_to_current_conversation` (опционально) параметр в conversation list select menu чтобы по-умолчанию отправлять сообщение в беседу из которой был вызван shortcut

 >[!tip] Доступность пользователя
> Методы `users.getPresence` и `dnd.info` позволят отследить доступность пользователя и Do Not Disturb часы

> [!tip]+ Key takeaways
> 1. Prioritizing user experience (UX) helps you design Slack apps that will be not just functional, but intuitive to use.
> 2. Design with empathy by keeping in mind how time zones, languages, workspace size, screen dimensions, workspace/user preferences and user roles shape how your app is experienced.
> 3. Your app's tone and voice are a representation of your brand.
> 4. Create UX-friendly surfaces through which your app communicates with users.
> 5. Use your App Home’s home tab to display personalized information, create visual hierarchies, present app settings and curate call-to-actions.
> 6. Keep modals glanceable and use them to indicate outcome and progress.  
> 7. Start your shortcuts with a verb (make them action-oriented) and customize based on context.
> 8. Never prompt users for confidential information and always be clear when a paid feature is available.

> [!tip]+ **Before moving on, confirm that you can:**
> - Design a user experience for your app that is empathetic to your audience.
> - Identify common factors that affect Slack users’ experience of apps.
> - Summarize best practices for communicating clearly and concisely with end users.
> - Explain best practices for designing user-friendly surfaces in Slack apps.
> - Describe best practices for onboarding new users to your app.
> - Plan for guest users that have differing access to your app's functionality.
> - Identify common factors that affect Slack users’ experience of apps.