# Раскройте себя с помощью ngrok

Очень часто в веб-разработке требуется показать своё веб-приложение другим
разработчиками или клиентам. О продакшн-версии сайта в этом случае беспокоиться не о
чем. Вы деплоите приложение на собственный сервер, садитесь в кресло и ждёте
предложения о покупке от Facebook. Но во время разработки вы пользуетесь локальным
сервером, и показать промежуточные результаты на котором является нетривиальной
задачей.

Это становится чуть больше, чем досадной неприятностью, когда вам требуется проверить
работу какого-то модуля в продакшн-окружении. Например GitHub поддерживает фичу под
названием _веб-хуки_. Вы говорите GitHub открывать определённый урл, при определённых
действиях с вашим репозиторием. Для того, чтобы веб-хуки правильно работали GitHub
нуждается в адресе, который он сможет открыть. Если вы укажете http://localhost или
http://127.0.0.1/ очевидно, что у вас ничего не заработает, так как в контексте
Github, localhost это он сам.

Знакомьтесь [ngrok][2], простой сервис который позволяет прокидывать локальный веб-
сервис (любого толка: Node.js, ColdFusion, PHP и так далее) в интернет. Он не только
позволяет не только просматривать ваши локальные сайты по по доступному для всех урлу,
но также тестировать входящие запросы также хорошо, как и повторять их (это может
стать решающим фактором, если вы тестируете сервис с ограничениями, вы можете
попросить ngrok повторить запрос без использования «настоящего» удалённого сервиса).

ngrok полностью бесплатен и также у него есть дополнительные услуги и для тех, кто
зарегистрируется и для тех, кто пожертвует деньги. Вам необязательно жертвовать какую-
то определённую сумму — достаточно будет какой-угодно суммы. Мне кажется, что это
*невероятно* богатый выбор для пользователя и поэтому ngrok может быть по достоинству
оценён.


## Установка и запуск

Сопротивляясь мейнстриму, ngrok один из немногих инструментов, который устанавливается без помощи npm. Невероятно, правда? Вместо этого вы просто скачиваете zip-архив, распаковываете его и копируете в папку для бинарников. Как только вы это сделали, запустите ваш сервер (ещё раз, ngrok непривиредлив до используемых вами технологий, поэтому не беспокойтесь даже об использовании Perl CGI) и напечатайте в терминале `ngrok 80` для начала работы ngrok. Спешу вас успокоить, конечно же, у ngrok тысячи полезных настроек. При старте ngrok берёт под контроль ваш терминал и постоянно рассказывает о происходящем.

![shot1][3]

Пропустим строчки о статусе и версии и обратим внимание на строчки о редиректах. Вы
заметите, что ngrok предоставляет доступ к вашему сайту по обоим типам соединения и
http и https по случайному поддомену (об этом чуть позже). Вы можете проверить работу
ngrok просто открыв этот адрес в браузере. Как только вы это сделаете вы увидите, что
ngrok обновил информацию в секции запросов.

![shot2][4]

Если вам интересно, то https работает замечательно даже если https не настроен на
вашем локальном сервере.

![shot3][5]

Теперь обратите внимание на URL веб-интерфеса. Он даёт прилично-выглядящий отчёт о
запросах на сервер.

![shot4][6]

В целом, ngrok очень мощный из коробки.


## Пример из реального мира

Давайте попробуем ngrok на реальном примере. Ранее я упоминал веб-хуки у GitHub,
давайте взглянем на них поближе. Я выбрал из своих проектов случайный
(<https://GitHub.com/cfjedimaster/BehanceAPI>), зашёл в настройки, раздел Webhooks &
Services и кликнул `Add webhook`.

В моём случае, я собираюсь использоваться простой ColdFusion-скрипт на моём локальном сервере. Я не буду делать ничего сложного, мне просто надо удостовериться, что он существует. Я создал файл `helloGitHub.cfm` и положил его в папку тестовую папку моего локального Apache сервера.

![shot5][7]

Заметьте — я выбрал опцию `Send me everything`, чтобы было легче тестировать. GitHub позволяет запускать вебхуки только на пуш в репозиторий и также хорошо на отдельные события. Нажмите `Add webhook` чтобы закончить.

Ок, что теперь? Теперь GitHub будет открывать URL когда что-то будет происходить. Но я
не представляю _что_ будет мне присылать GitHub. Уверен, что это будет куча полезной
информации и я могу найти информацию об этом в документации, но зачем скучать за
мануалами, когда мы можем протестировать наживую с ngrok.

Как я уже сказал, GitHub будет запускать вебхук на любое событие, поэтому я добавлю
пустой ишью к моему проекту и посмотрю, что произойдёт.

В тот же момент, как я добавил ишью, веб-интерфейс ngrok в реальном времени отразил
новое событие.

![shot6][8]

Как вы можете видеть, вы можете посмотреть краткую сводку и заголовки точно также как
необработанный и бинарный код. Заметьте, что информация отображается как есть — без
форматирования JSON-кода. На самом деле ngrok поддерживает форматирование JSON, но
только если тип данных `application/json`. Но так как GitHub не помечает данные этим
типом, то ngrok не может понимает данные как JSON.

С этого момента можно начать тестирование. Я поменю мой серверный код, чтобы правильно
обрабатывать вебхук и делать что-нибудь умное. Для тестирования мне необходимо каждый
раз что-нибудь делать с моим репозиторием. ngrok позволяет мне пропустить это, и
повторить запрос от GitHub локально. К несчастью, в этот момент я встретился с моим
первым багов ngrok. Я могу повторить первоначальный запрос от GitHub, при регистрации
вебхука, но не могу повторить запрос приходящий при создании ишью к моему проекту. Я
зарепортил это как [баг][9] и уверен, что это поведение скоро починят (прим.
переводчика — уже починили).

## Дополнительные возможности

В целом, это крутой сервис, особенно если учесть как просто его использовать и сколько
он стоит (бесплатно), но что можно получить за регистрацию? В первую очередь, _также
бесплатно_ вы получите именованные поддомены, TCP-туннелирование, HTTP-аутентификацию
и множественные тунели.

Регистрация безболезнена и по её окончании, вы получаете собственный токен. Вы
указываете ваш токен параметром в командной строке для использования новых
возможностей. К примеру, чтобы использовать именованый поддомен, я делаю так (токен я
поменял):

    ngrok -authtoken Ythisisrandomsecretcrap2 -subdomain mymilkshakeisbetterthanyours 80

Возможности следующего уровня требуют оплаты, но я опять напомню размер оплаты
свободный и зависит только от вас. Эти возможности включают в себя допонительную
поддержку туннелирования и резервирование поддоменов. Если честно, то мне кажется,
если вы чувствуете необходимость резервирование поддомена, то возможно вам стоит
задуматься о настоящем веб-сервере.


## Пара слов в заключение

В итоге, ngrok мощный инструмент, удовлетворяющий особенную потребность, что я думаю
будет безмерно полезно для веб-разработчиков.

 [1]: http://flippinawesome.org/authors/raymond-camden
 [2]: http://ngrok.com
 [3]: img/shot1.png
 [4]: img/shot2.png
 [5]: img/shot3.png
 [6]: img/shot4.png
 [7]: img/shot5.png
 [8]: img/shot6.png
 [9]: https://GitHub.com/inconshreveable/ngrok/issues/118
