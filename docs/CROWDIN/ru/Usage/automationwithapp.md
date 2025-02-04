# Автоматизация при помощи стороннего приложения Android Automate

**This article has been written before AndroidAPS version 2.5. There is an [automation plugin in AndroidAPS](./Automation.md) itself with AndroidAPS version 2.5. For some, this here might be still useful, but should only be used by advanced users.**

Так как AndroidAPS-гибридная замкнутая система замкнутого цикла, необходимо некоторое взаимодействие с пользователем (например, подсказки алгоритму, что вы гуляли, ожидаете приема пищи, лежите на диване ...). Часто вводимые вручную пользовательские данные можно автоматизировать с помощью таких внешних инструментов, как Automate или IFTTT, что расширяет функциональность AndroidAPS.

## Приложение Android Automate

Бесплатное приложение Android™ Automate позволяет автоматизировать различные задачи на смартфоне. Создайте свои автоматизации с поточными диаграммами, чтобы ваше устройство автоматически изменяло такие настройки, как Bluetooth, Wi-Fi, NFC, или выполнять такие действия, как отправка SMS, электронной почты, на основе вашего положения, времени суток или любого другого "инициатора событий". Вы можете автоматизировать практически все на своем устройстве, Automate поддерживает даже вспомогательные модули, сделанные для Tasker и Locale.

С помощью этого инструмента вы можете легко создавать рабочие потоки для автоматического лечения диабета на основе нескольких условий в соответствии с принципом ' если это... и это ... не так ..., тогда сделать то ... и то ... Есть тысячи возможностей, которые можно сконфигурировать.

Пока что ** для цикла требуется профиль Nightscout **, так как Automate выполняет команды через HTTP-запрос непосредственно на веб-сайте NS, который впоследствии синхронизирует его с приложением AndroidAPS.

** Автономный цикл (прямое соединение между приложением Automate и AndroidAPS) еще не поддерживается **, но технически возможно. Возможно, в будущем появится решение. Если вы нашли способ сделать это, добавьте его в эту документацию или обратитесь к разработчику.

### Основные требования

#### Автоматизировать приложение

Загрузите Android Automate в Google Play Store или по адресу [ https://llamalab.com/automate/](https://llamalab.com/automate/) и установите его на смартфон с AndroidAPS.

В программе Automate коснитесь сендвич-меню в верхнем левом углу экрана > Настройки > и отметьте 'Выполнять при запуске системы'. Это позволит автоматически запускать рабочие процесс при запуске системы.

![Автоматический HTTP-запрос](../images/automate-app2.png)

#### AndroidAPS

В AndroidAPS коснитесь меню из трех точек в верхнем правом углу экране и перейдите в Настройки > NSClient > Параметры соединения > Снимите галочку с 'Использовать только соединение WiFi' и 'Только при зарядке', так как автоматизированное лечение работает только тогда, когда у AndroidAPS есть фактическое соединение с Nightscout.

![Параметры подключения Nightscout](../images/automate-aaps1.jpg)

В AndroidAPS коснитесь 3-точечного меню в верхней правой части экране и перейдите в меню Параметры > NSClient > Дополнительные параметры > Снимите галочки с 'Только загрузка в NS (без синхронизации)' и 'Без загрузки в NS'.

Be aware of the [security issues](Nightscout-security-considerations) that might occure and be very careful if you are using an [Insight pump](Accu-Chek-Insight-Pump-settings-in-aaps).

![Настройки загрузок Nightscout](../images/automate-aaps2.jpg)

### Примеры рабочих процессов

#### Пример 1: Если обнаружено действие (например, ходьба или бег), задать высокую временную цель TT. Если нагрузка прекращена, ждать 20 минут, а затем отменить TT

Этот рабочий процесс слушает датчики смартфона (шагомер, датчик гравитации ...), которые обнаруживают вашу активность. Если существует недавняя активность типа ходьбы, бега или катания на велосипеде, то Automate задаст высокую временную цель на вами указанное время. Если операция завершится, смартфон обнаружит это, подождет 20 минут, а затем вернет цель к обычному значению профиля.

Загрузите сценарий автоматизации [ https://llamalab.com/automate/community/flows/27808 ](https://llamalab.com/automate/community/flows/27808).

Редактируйте последовательность дейтвий нажав на символ карандаша редактирования > Блок-схема

![Automate sling](../images/automate-app3.png)

Настройте рабочий процесс по своему усмотрению следующим образом:

![Automate sling](../images/automate-app6.png)

1. = Задайте значение высокой временной цели TT
2. = Перейти к обычной цели через 20 минут по завершении нагрузки

1 ![Automate sling](../images/automate-app1.png)

2 ![Automate sling](../images/automate-app5.png)

Параметры требуемого URL : Ваш NS-URL с кончным адресом /api/v1/treatments.json (например, https://my-cgm.herokuapp.com/api/v1/treatments.json)

Требуемый контент:

* цельВерх/цельНиз: значение высокой временной цели TT (верх и низ должны иметь одно значение)
* продолжительность: длительность высокой TT (после чего цель вернется к обычному значению для профиля, если только активность не продолжится). 
* secret: хэш API SHA1. Это не ваш ключ API! Ключ API можно преобразовать в формат SHA1 здесь: [ http://www.sha1-online.com/](http://www.sha1-online.com/)

Сохранить: нажать на 'Готово' и на крючок

Запустить последовательность действий: нажать на кнопку Play

#### Пример 2: Если xDrip + оповещает о высокой ГК, то задается низкая временная цель TT на ... минут.

Этот сценарий прослушивает канал уведомлений xDrip +. Если срабатывает заданное пользователем xDrip+ оповещение о высокой ГК, Automate задает указанную пользователем низкую временную цель на указанное им время. После этого, возможно, еще одно оповещение продлит время работы низкой временной цели TT.

##### xDrip +

Сначала необходимо добавить оповещение о высокой ГК в xDrip + следующим образом:

![Настройки оповещения xDrip+](../images/automate-xdrip1.png)

Имя оповещения: (Обратите внимание на него!) Оно важно для запуска триггера. Оно не должно содержать ошибок и быть похожим на названия других оповещений. Пример: '180alarm' не должно существовать рядом с '80alarm'.

Порог: значение ГК, которое должно инициировать оповещение о высоком сахаре.

Значение кнопки Snooze (игнорировать) по умолчанию: Вставьте длительность, которую вы планируете задать для низкой временной цели TT, так как оповещение возникнет снова и, возможно, продлит время низкой TT.

![Настройки оповещения xDrip+](../images/automate-xdrip2.png)

##### Автоматизация

Во-вторых, загрузите сценарий Automate [ https://llamalab.com/automate/community/flows/27809 ](https://llamalab.com/automate/community/flows/27809).

Редактируйте последовательность дейтвий нажав на символ карандаша редактирования > Блок-схема

![Automate sling](../images/automate-app3.png)

Настройте рабочий процесс по своему усмотрению следующим образом:

В триггере 'Уведомление, размещено?' нужно задать 'TITLE'( имя оповещения) xDrip +, которое должно инициировать триггер, а также добавить переменную * до и после этого имени.

![Automate sling](../images/automate-app7.png)

![Automate sling](../images/automate-app4.png)

Параметры требуемого URL : Ваш NS-URL с кончным адресом /api/v1/treatments.json (например, https://my-cgm.herokuapp.com/api/v1/treatments.json)

Требуемый контент:

* targetTop/targetBottom: Низкое значение TT (верхнее и нижнее значения должны совпадать)
* duration: длительность низкой временной цели TT (после которой ГК будет возвращаться к обычному значению цели профиля). Рекомендуется использовать то же время, что и стандартное время в xDrip + ( 'Standard snooze')
* secret: хэш API SHA1. Это не ваш ключ API! Ключ API можно преобразовать в формат SHA1 здесь: [ http://www.sha1-online.com/](http://www.sha1-online.com/)

Сохранить: нажать на 'Готово' и на крючок

Запустить последовательность действий: нажать на кнопку Play

#### Пример 3: Добавьте сами!!!

Добавьте дополнительные автоматизированные рабочие потоки, загрузив файл .flo в группу Automate (под ключевым словом 'Nightscout') и опишите его здесь, выполнив [ Pull Request ](../make-a-PR.md) в репозитории AndroidAPSdocs.

## Если так, тогда (IFTTT)

Не стесняйтесь добавлять инструкции через Pull Request...