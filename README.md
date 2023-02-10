# qa_guru_python_3_12 - Телеграм бот. Отправляем уведомления о результатах прохождения тестов

# Что понадобится

* Библиотека, позволяющая реализовать отправку уведомлений [ссылка](https://github.com/qa-guru/allure-notifications);
* Бот в Telegram;

# Как создать бота
Для создания бота надо обратиться к другому боту, найти которого можно в поиске Telegram по никнейму @BotFather. Важно быть внимательным, много нехороших людей создает фейковые боты, которые полностью копируют аватарку и описание бота, но во время создания бота просят сказать ему какие-то персональные данные или прислать уникальный токен. Запомните, что у официального бота есть верификационная галочка. Также никому не сообщайте никому уникальные токен, с ним любой человек может получить доступ к вашему боту.

![bot_father](https://raw.githubusercontent.com/qa-guru/knowledge-base/main/img/les12/les12-1.png)
Так выглядит официальный профиль @BotFather в Telegram

Теперь надо открыть чат с BotFather и написать ему команду /newbot. После этого система попросит задать имя бота, которое будет отображаться в чате. Это имя может быть любым. Далее надо будет прислать никнейм бота. В этом случае никнейм обязательно должен заканчиваться словом bot. Вместо пробелов можно использовать нижние подчеркивания. После BotFather пришлет в чат сообщение о том, что бот создан. Вместе с этим придет уникальные токен, который следует сохранить, и ссылка на работу с Bot API.

![bot_edit](https://raw.githubusercontent.com/qa-guru/knowledge-base/main/img/les12/les12-2.png)

Через BotFather можно задать аватарку бота, настроить его описание, подключить платежи и изменить имя пользователя.

# Как узнать Chat ID

Бот должен присылать сообщения в определенный чат. В Telegram у каждого чата есть уникальный идентификатор, с помощью которого можно настроить оправку сообщений. Для того, чтобы узнать Chat ID, надо добавить бота в целевой чат сделать его администратором, перейти по ссылке вида https://api.telegram.org/botИдентификаторВашегоБота/getUpdates, написать в чат и обновить страницу.


В результате мы увидим само сообщение, всю информацию о сообщении, ID отправителя и ID чата, который нам нужен. В стандартном виде страница будет рендериться без форматирования и нужные данные будет сложно найти. Ситуацию можно исправить, установив расширение JSONView.

![token_img](https://raw.githubusercontent.com/qa-guru/knowledge-base/main/img/les12/les12-3.png)


Теперь мы знаем все для отправки сообщений с помощью терминала. Для этого нам понадобится следующий набор команд:

```Python
curl -X POST \
     -H 'Content-Type: application/json' \
     -d '{"chat_id": "123456789", "text": "This is a test from curl", "disable_notification": true}' \
     https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage
```

Необходимо:

* Заменить содержимое chat_id на актуальный идентификатор чата;
* Указать в text текст желаемого сообщения;
* disable_notification отвечает за отправку сообщения с уведомлением или без;
* Заменить $TELEGRAM_BOT_TOKEN на актуальный токен.

После заполнения команда можно нажать на Enter и если все было сделано правильно, то бот отправит сообщение в чат.

# Как отправлять уведомления из проекта
> Важно: к проекту уже должен быть подключен Allure. Все взаимодействие идет с ним.

Первым делом необходимо по [ссылка](https://github.com/qa-guru/allure-notifications) скачать последнюю версию библиотеки и добавить jar-файл в проект. Далее создадим в проекте отдельную папку notifications и в ней файл конфигурации telegram.json. После в созданный файл вставим и заполним следующую конструкцию:

```Python
{
  "base": {
    "project": "some project",
    "environment": "some env",
    "comment": "some comment",
    "reportLink": "",
    "language": "en",
    "messenger": "telegram",
    "allureFolder": "build/allure-report/",
    "enableChart": true
  },
  "telegram": {
    "token": "asdhsdgfjsdfgFgjhg4831)@",
    "chat": "-1",
    "replyTo": ""
  }
}
```

Все поля интуитивно понятны из названия. Главное указать токен бота в поле token и идентификатор чата в chat. Также стоит убедиться, что путь в поле allureFolder ведет к папке с данными работы Allure.

После заполнения json-файла надо перейти в терминал и выполнить следующую команду:

```Python
java \
"-DconfigFile=${PATH_TO_FILE}" \
-jar allure-notifications-4.1.jar
```

${PATH_TO_FILE} надо заменить на путь к созданному json-файлу.

После выполнения команды бот пришлет в чат сообщения с результатами тестов.

# Как подключить все к Jenkins

В настройке сборки Jenkins необходимо выбрать Добавить шаг, в нем пункт Create/Update Text File.
![jk_file](https://raw.githubusercontent.com/qa-guru/knowledge-base/main/img/les12/les12-4.png)

В File Path указываем путь к json-файлу, в Text File Content вставляем содержимое файла. Ставим галочки на Create at Workspace и Overwrite file.
![general_options](https://raw.githubusercontent.com/qa-guru/knowledge-base/main/img/les12/les12-5.png)

Перед этим в json-файл надо добавить внутренние переменные Jenkins. Итоговый обновленный файл будет выглядеть так:

```Python
{
  "base": {
    "project": "${JOB_BASE_NAME}",
    "environment": "some env",
    "comment": "some comment",
    "reportLink": "${BUILD_URL}",
    "language": "en",
    "messenger": "telegram",
    "allureFolder": "build/allure-report/",
    "enableChart": true
  },
  "telegram": {
    "token": "asdhsdgfjsdfgFgjhg4831)@",
    "chat": "-1",
    "replyTo": ""
  }
}
```

Далее жмем Добавить шаг после сборки и выбираем Post build task.
![build_on](https://raw.githubusercontent.com/qa-guru/knowledge-base/main/img/les12/les12-6.png)

В поле Script указываем команду для отправки сообщения и сохраняем.
![lib_jar](https://raw.githubusercontent.com/qa-guru/knowledge-base/main/img/les12/les12-7.png)

Таким же образом можно создать несколько файлов конфигурационных файлов и отправлять уведомления в разные чаты.


# Allure-reports:
https://jenkins.autotests.cloud/job/avaoleg_qa_guru_python_3_12_jenkins_telegram_notofication/allure/

# Telegram notifiactions:


![screen](https://user-images.githubusercontent.com/49872564/213936015-ae8a1302-e92a-4de7-90cc-6cb50aaa411e.png)
