Yii-phpList
===========

Integration phpList and Yii

Общие сведения
--------------
Модуль представляет собой полноценную систему phpList для управления рассылками и компонент для фреймворка Yii,
предоставляющий другим частям системы стандартный API для доступа к функциям рассылка.

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

Необходимо отметить, что:
- предполагается, что используется стандартный файл yii без функций слияния (CMap::mergeArray).
- предполагается, что приложение на yii использует mysql как сервер базы данных. В ином случае phpList работать не будет.
Если внутри приложения используется другой движок базы данных, то для работы phpList необходимо создать новый компонент
приложения, который будет работать с mysql и в настройках phpList использовать имя этого компонента.

4. Для создания таблиц phpList надо запустить стандартный установщик http://site.name/lists/admin/
