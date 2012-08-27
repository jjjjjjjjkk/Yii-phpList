Yii-phpList
===========

Integration phpList and Yii

Общие сведения
--------------
Модуль представляет собой полноценную систему phpList для управления рассылками и компонент для фреймворка Yii,
предоставляющий другим частям системы стандартный API для доступа к функциям рассылка.

Системные требования
--------------------
- предполагается, что используется конфигурация yii - стандартный файл, возвращающий массив и не использующий внутри функций слияния yii
(CMap::mergeArray).
- предполагается, что приложение на yii использует mysql как сервер базы данных. В ином случае phpList работать не будет.
Если внутри приложения используется другой движок базы данных, то для работы phpList необходимо создать новый компонент
приложения, который будет работать с mysql и в настройках phpList использовать имя этого компонента.


Установка
---------
Модуль состоит из 2х частей:

1. Компонент mailing - компонент фреймворка yii. Его надо распаковать в папку protected/components.

2. Модуль рассылки phpList - его необходимо сделать доступным через веб браузер. Для этого после выполнения пункта 1
необходимо _переместить_ папку lists из директории /protected/components/mailing/vendors/phplist-2.10.18/public_html
в www_root папку сервера. Чтобы доступ к ней из браузера был возможен по ссылке вида http://site.name/lists/

3. Код phpList изменён, чтобы он мог брать настройки из настроек фрреймворка yii. Для корректного функционирования надо
настроить путь к конфигурационному файлу yii в файле /lists/config/config.php. В случае, если имя компонента базы
данных в конфигурации yii отличается - то его необходимо также настроить в указанном ранее файле.

4. Для создания таблиц phpList надо запустить стандартный установщик http://site.name/lists/admin/

5. Для просмотра полного руководства по настройке phpList пользуйтесь http://docs.phplist.com/PhpListConfiguration.html

6. Настройте консольное приложение для выполнения команд компонента. Для этого в конфигурационном файле console.php добавьте
```php
'commandMap' => array(
    'mailing' => array(
        'class' => 'application.components.mailing.MailingCommand',
        'phpListPath' => 'http://yiiphplist.local/lists', //url к текущей установке phpList
        'cronUserName' => 'cron', //имя пользователя, используемое для рассылки писем (будет создан автоматически при установке)
        'cronUserPassword' => 'sjdfh12e12nr23r5@#$!@fd', //пароль пользователя для отправки писем
))
```

7. Запустите консольный инсталлятор компонента *yiic mailing install phplist_*, где *phplist_* - префикс таблиц в базе данных
для phpList (в файле config.php у phpList этот параметр называется *table_prefix*). Это добавит необходимые данные в таблицы
phpList.

8. Настроить компонент веб приложения в файле config.php:
```php
'mailing'=>array(
    'class'=>'application.components.mailing.MailingComponent',
    'phpListPrefix' => 'phplist_', //префикс таблиц phplist
    'validateEmails' => true //валидировать ли email при выполнении операций
)
```

9. Для интеграции административной части phpList и yii в плане добавления подписчиков необходимо в любом контроллере
прописать пути к action компонента, отвечающим за вывод контрола "Список пользователей для добавления" и "Добавление
подписчика":
```php
public function actions()
{
    return array(
        'subscribeList'=>array( //вывод контрола "Список пользователей для добавления"
            'class'=>'application.components.mailing.GetUsersAction',

            'yiiUserModel' => 'Users', //имя класса модели с пользователями
            'yiiUserIdColumn' => 'id', //имя столбца, по которому связываем пользователей
            'yiiUserEmailColumn' => 'email', //поле email
            'yiiUserSortColumn' => 'email', //поле, по которому сортируется список
            'yiiUserNameColumn' => 'username'; //поле используемое для визуального отображения имени пользователя

            'phpListPrefix' => 'phplist_'; //префикс таблиц phpList

            'formMethod' => 'post'; //метод сабмита формы
            'formAction'=>Yii::app()->createUrl('/site/addSubscriber') //адрес, куда должна сабмититься форма (см. далее)
            'paramName'=>'subscriberIds', //имя параметра для сабмита
        ),
        'addSubscriber'=>array( //"Добавление подписчика"
            'class'=>'application.components.mailing.AddSubscriberAction',

            'yiiUserModel' => 'Users', //имя класса модели с пользователями
            'yiiUsedIdColumn' => 'id', //имя столбца, по которому связываем пользователей
            'yiiUserEmailColumn' => 'email', //поле email

            'method' => 'post', //метод сабмита формы
            'paramName'=>'subscriberIds', //имя параметра для сабмита

            'mailingComponent' => 'mailing', //имя компонента, используемого для добавления
            'autoConfirm' => true //считать ли адреса подписчиков автоматически подтверждёнными
        )
    );
}
```
Действия сделаны автономными от компонента mailing и универсальными и именно поэтому принимают на вход много параметров.

10. Для корректной работы рассылки добавьте команду 'yiic mailing' в crontab с периодичностью выполнения в полчаса. После
вызова этой команды все созданные рассылки рассылаются автоматически средствами phpList. Настройки рассылки также
производятся через административный интерфейс phpList.