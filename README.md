# Библиотека интеграции 1С с сайтом "ВКонтакте" (VK SDK for 1C)
Библиотека интеграции с сайтом ВКонтакте выполнена в виде расширения и содержит базовый набор функций для обмена с сайтом. Так же в ней реализованы механизмы авторизации пользователя, отключения его аккаунта и ввода Captcha. Помимо самой библиотеки присутствует демонстрационная база с примерами размещения записей на стене пользователя и сообществ, добавления фотографий в альбом и добавления товаров на страницу ВКонтакте в раздел "Товары". Используя данную библиотеку и примеры реализации из демо-базы можно просто и быстро реализовать требуемый вам функционал по интеграции вашей учётной системы 1С с сайтом ВКонтакте.

**ext** - библиотека в виде расширения.

**demo** - конфгурация, демонстрирующая использование библиотеки.

## Авторизация ВКонтакте из 1С.

Для выполнения запросов к API ВКонтакте необходимо выполнить авторизацию и получить ключ доступа (access_token), который будет использоваться при вызове методов API. При выполнении авторизации необходимо указать ID вашего приложения (приложение создаётся на сайте ВКонтакте в разделе разработчики по ссылке у https://vk.com/editapp?act=create) и необходимые права доступа к аккаунту ВКонтакте. Права доступа являются строковыми значениями и перечисляются через запятую. Например, если необходимо получить доступ к фотографиям и группам пользователя, то значение прав доступа будет равно "**photos,wall**". Полный список прав доступа и их значения можно получить в разделе разработчика ВКонтакте по адресу https://vk.com/dev/permissions.

Подробную информацию о получении ключа доступа можно получить в разделе разработчика ВКонтакте по адресу https://vk.com/dev/access_token.

Для выполнения авторизации используется процедура **АвторизацияВКонтакте** общего модуля **vk_ИнтеграцияВККлиент**.

**Синтаксис:**

АвторизацияВКонтакте(<ИдентификаторПриложения>, <ПраваДоступа>, <ОповещениеОЗавершении>)

**Параметры:**

*<ИдентификаторПриложения> (обязательный)* - Строка - ID вашего приложения, которому будет представлен доступ.

*<ПраваДоступа> (обязательный)* - Строка - список прав доступа, перечисленных через запятую.

*<ОповещениеОЗавершении> (обязательный)* - ОписаниеОповещения - описание процедуры, которая будет вызвана при завершении авторизации. Если авторизация не была выполнена, значение результата закрытия будет Неопределено, иначе Структура:

+ ИдентификаторПользователя - Строка - идентификатор авторизованного пользователя (используется для запросов к API);
+ КлючДоступа - Строка - ключ доступа (access_token), используется для запросов к API;
+ СрокДействияКлюча - Дата - дата окончания срока действия ключа. По истечению этого срока необходимо заново выполнить процедуру авторизации для получения нового ключа. Если установлено право доступа offline (доступ в любое время), то значение равно Неопределено.

**Доступность**: Клиент.

**Описание:**

Выполняет открытие формы авторизации пользователя на сайте ВКонтакте. При успешном выполнении авторизации результат выполнения в параметре **ОповещениеОЗавершении** содержит полученные параметры доступа (идентификатор пользователя, ключ доступа и срок его действия).

## Вызов методов API ВКонтакте.

Для вызовов методов API используется функция **ВызватьМетодAPI** общего модуля **vk_ИнтеграцияВККлиентСервер**.

**Синтаксис:**

ВызватьМетодAPI(<ИмяМетода>, <Параметры>, <КлючДоступа>, <ИнформацияОбОшибке>)

**Параметры:**

*<ИмяМетода> (обязательный)* - Строка - имя вызываемого метода API.

*<Параметры> (обязательный)* - Структура - параметры вызываемого метода.

*<КлючДоступа> (обязательный)* - Строка - ключ доступа, полученный при авторизации (см. раздел Авторизация ВКонтакте из 1С).

*<ИнформацияОбОшибке> (необязательный)* - Структура - при возникновении ошибки содержит подробную информацию о ней. Если метод выполнен успешно - содержит значение Неопределено.

+ КодОшибки - Строка - код ошибки.
+ ТекстОшибки - Строка - текст ошибки.
+ ОшибкаHTTP - Булево - содержит значение **Истина**, если ошибка произошла при вызове HTTP-метода (404, 500 и т.п.), например, если сервер недоступен. Если ошибка возникла при выполнении метода ВК API, то будет содержать значение **Ложь**. Подробнее про коды ошибки API см. https://vk.com/dev/errors.
+ Представление - Строка - представление текста ошибки для отображения пользователю.
+ ТребуетсяCaptcha - Булево - Истина, если ошибка вызвана требованием ввести Captcha (подробнее см. https://vk.com/dev/captcha_error).
+ ПараметрыCaptcha - Структура - если *ТребуетсяCaptcha = Истина*, то содержит параметры Captcha, используемые для ввода Captcha пользователем:
 
*Идентификатор* - Строка - идентификатор captcha;
 
*АдресИзображения* - Строка - URL адрес изображения captcha;
 
*КлючДоступа* - Строка - ключ доступа к API, используется для повторной отправки запроса после ввода пользователем Captcha;
 
*ПереданныеПараметры* - Массив - содержит массив переданных параметров вызванного метода. Используется для повторной отправки запроса после ввода пользователем Captcha.

**Возвращаемое значение:**

Структура, Массив - структура или массив со значениями, возвращаемые вызываемым методом. Если во время выполнения метода произошла ошибка, то возвращает значение **Неопределено**.

**Доступность:** Клиент, Сервер.

**Описание:**

Вызывает метод API ВКонтакте и возвращает результат его выполнения. Полную информацию о методах API, их параметрах и возвращаемых значениях см. в руководстве разработчика ВКонтакте по адресу https://vk.com/dev/methods.

Подробнее информацию об обработке ошибки, связанную с требованием ввести captcha см. в разделе Запрос Captcha.

## Проверка авторизации пользователя перед вызовом метода API.

Если используется ключ доступа с ограниченным сроком действия (при авторизации в правах доступа отсутствует право **offline**), то может возникнуть ситуация, когда при отправке запроса срок действия ключа уже закончен. В таком случае метод ВК вернёт соответствующую ошибку. Для упрощения обработки такой ситуации предназначена процедура **ПроверитьАвторизациюПользователя** общего модуля **vk_ИнтеграцияВККлиент**.

**Синтаксис:**

ПроверитьАвторизациюПользователя(<ОповещениеОЗавершении>, <ИдентификаторПриложения>, <ПраваДоступа>, <СрокДействияКлюча>, <АвтообновлениеКлюча>)

**Параметры:**

*<ОповещениеОЗавершении> (обязательный) - ОписаниеОповещения* - описание процедуры, которая будет вызвана при завершении проверки. Значение результата проверки - Структура:
+ Результат - Строка - результат проверки ключа. Принимает следующие значения:
+ КлючДействителен - ключ доступа является действующим;
+ КлючНедействителен - срок действия ключа доступа истёк;
+ КлючОбновлен - срок действия ключа истёк и был обновлён пользователем.
+ ПараметрыДоступа - Структура - если ключ доступа был обновлён пользователем, то содержит обновлённые параметры доступа.

*<ИдентификаторПриложения> (обязательный)* - Строка - идентификатор приложения, используется для обновления ключа доступа при завершении его срока действия.

*<ПраваДоступа> (обязательный)* - Строка - права доступа к аккаунту, используется для обновления ключа доступа при завершении его срока действия.

*<СрокДействияКлюча> (обязательный)* - Дата - проверяемый срок действия ключа.

*<АвтообновлениеКлюча> (необязательный)* - Булево - если Истина, то пользователю будет предложено обновить ключ доступа в случае истечения его срока действия. Значение по умолчанию: Истина.

**Доступность:** Клиент.

**Описание:**

Выполняет проверку наличия действующего ключа доступа к API ВК в клиентском сеансе. Если срок действия ключа истёк, то пользователю будет предложено обновить ключ. По завершении выполнения результат в параметре ОповещениеОЗавершении будет содержать результат проверки. Если значения параметра АвтообновлениеКлюча равно Истина и срок действия ключа истёк, то пользователю будет предложено обновить ключ.

## Запрос Captcha.

В API ВКонтакте существует ограничение на вызов однотипных методов. После превышения количественного лимита доступ к конкретному методу может потребовать ввода captcha. Для ввода captcha пользователем предназначена процедура **ВвестиCaptchaИПовторитьВызовМетодаAPI** общего модуля **vk_ИнтеграцияВККлиент**.

**Синтаксис:**

ВвестиCaptchaИПовторитьВызовМетодаAPI(<ПараметрыCaptcha>, <ОповещениеОЗавершении>)

**Параметры:**

*<ПараметрыCaptcha> (обязательный)* - Структура - параметры captcha, возвращённые с ошибкой при вызове метода API. Подробнее см. раздел Вызов методов API ВКонтакте.

*<ОповещениеОЗавершении> (обязательный)* - ОписаниеОповещения - описание процедуры, которая будет вызвана при завершении ввода captcha. Если captcha не была введена пользователем, то результат выполнения будет **Неопределено**, иначе Структура:
+ Ответ - Структура - результат выполнения метода API. Если во время выполнения метода возникла ошибка, то содержит значение Неопределено.
+ ИнформацияОбОшибке - Структура - если вызов метода выполнен успешно, то содержит значение Неопределено, иначе содержит сведения о возникшей ошибке.

**Доступность:** Клиент

**Описание:**

Открывает форму для ввода captcha. После ввода captcha выполняет повторный запрос, приведший к возникновению необходимости ввода captcha.

## Вспомогательные методы для работы с API ВКонтакте.

Помимо основного метода ВыполнитьМетодAPI в библиотеке существуют дополнительные методы для облегчения загрузки файлов на сайт ВКонтакте, находящиеся в общем модуле vk_ИнтеграцияВККлиентСервер.

### ЗагрузитьФотографиюВАльбом

**Синтаксис:**

ЗагрузитьФотографиюВАльбом(<КлючДоступа>, <ИдентификаторАльбома>, <ИдентификаторСообщества>, <Изображение>, <Описание>, <Широта>, <Долгота>, <ИнформацияОбОшибке>)

**Параметры:**

*<КлючДоступа> (обязательный)* - Строка - ключ доступа к API.

*<ИдентификаторАльбома> (обязательный)* - Строка - идентификатор альбома, в который необходимо сохранить фотографии.

*<ИдентификаторСообщества> (необязательный)* - Число - идентификатор сообщества, которому принадлежит альбом (если необходимо загрузить фотографию в альбом сообщества). Если параметр не указан, то фотография загружается на стену текущего пользователя.

*<Изображение> (обязательный)* - ДвоичныеДанные - двоичные данные загружаемого изображения.

*<Описание> (необязательный)* - Строка - текст описания фотографии (максимум 2048 символов).

*<Широта> (необязательный)* - Число - географическая широта, заданная в градусах (от -90 до 90).

*<Долгота> (необязательный)* - Число - географическая долгота, заданная в градусах (от -180 до 180).

*<ИнформацияОбОшибке> (необязательный)* - Структура - при возникновении ошибки содержит подробную информацию о ней.


**Возвращаемое значение:**

Массив - загруженные объекты фотографий, см. https://vk.com/dev/objects/photo

**Доступность:** Клиент, Сервер.

**Описание:**

Загружает фотографию в альбом пользователя или сообщества. Подробнее см. в руководстве разработчика ВКонтакте по адресу: https://vk.com/dev/upload_files

### ЗагрузитьФотографиюНаСтену

**Синтаксис:**

ЗагрузитьФотографиюНаСтену(<КлючДоступа>, <ИдентификаторПользователя>, <ИдентификаторСообщества>, <Изображение>, <Описание>, <Широта>, <Долгота>, <ИнформацияОбОшибке>)

**Параметры:**

*<КлючДоступа> (обязательный)* - Строка - ключ доступа к API.

*<ИдентификаторПользователя> (обязательный)* - Число - идентификатор пользователя, на стену которого нужно сохранить фотографию.

*<ИдентификаторСообщества> (необязательный)* - Число - идентификатор сообщества, на стену которого нужно загрузить фото (без знака "минус").
*<Изображение> (обязательный)* - ДвоичныеДанные - двоичные данные загружаемого изображения.

*<Описание> (необязательный)* - Строка - текст описания фотографии (максимум 2048 символов).

*<Широта> (необязательный)* - Число - географическая широта, заданная в градусах (от -90 до 90).

*<Долгота> (необязательный)* - географическая долгота, заданная в градусах (от -180 до 180).

*<ИнформацияОбОшибке> (необязательный)* - Структура - при возникновении ошибки содержит подробную информацию о ней.

**Возвращаемое значение:**

Массив - загруженные объекты фотографий, см. https://vk.com/dev/objects/photo

**Доступность:** Клиент, Сервер.

**Описание:**

Загружает фотографию на стену пользователя или сообщества. Подробнее см. в руководстве разработчика ВКонтакте по адресу: https://vk.com/dev/upload_files

### ЗагрузитьФотографиюТовара

**Синтаксис:**

ЗагрузитьФотографиюТовара(<КлючДоступа>, <ИдентификаторСообщества>, <Изображение>, <ДляОбложки>, <КоординатаДляОбрезкиX>, <КоординатаДляОбрезкиY>, <ШиринаПослеОбрезки>, <ИнформацияОбОшибке>)

**Параметры:**

*<КлючДоступа> (обязательный)* - Строка - ключ доступа к API.

*<ИдентификаторСообщества> (обязательный)* - Число - идентификатор сообщества, на стену которого нужно загрузить фото (без знака "минус").

*<Изображение> (обязательный)* - ДвоичныеДанные - двоичные данные загружаемого изображения.

*<ДляОбложки> (необязательный)* - Булево - является ли фотография обложкой товара (**Истина** — фотография для обложки, **Ложь** — дополнительная фотография). Значение по умолчанию - *Истина*.

*<КоординатаДляОбрезкиX> (необязательный)* - Число - координата x для обрезки фотографии (верхний правый угол).

*<КоординатаДляОбрезкиY> (необязательный)* - Число - координата y для обрезки фотографии (верхний правый угол).

*<ШиринаПослеОбрезки> (необязательный)* - Число - ширина фотографии после обрезки в px.

*<ИнформацияОбОшибке> (необязательный)* - Структура - при возникновении ошибки содержит подробную информацию о ней.

**Возвращаемое значение:**

Массив - загруженные объекты фотографий, см. https://vk.com/dev/objects/photo

**Доступность:** Клиент, Сервер.

**Описание:**

Загружает фотографию для товара. Подробнее см. в руководстве разработчика ВКонтакте по адресу: https://vk.com/dev/upload_files

### ЗагрузитьФотографиюДляПодборкиТоваров

**Синтаксис:**

ЗагрузитьФотографиюДляПодборкиТоваров(<КлючДоступа>, <ИдентификаторСообщества>, <Изображение>, <ИнформацияОбОшибке>)

**Параметры:**

*<КлючДоступа> (обязательный)* - Строка - ключ доступа к API.

*<ИдентификаторСообщества> (обязательный)* - Число - идентификатор сообщества, на стену которого нужно загрузить фото (без знака "минус").

*<Изображение> (обязательный)* - ДвоичныеДанные - двоичные данные загружаемого изображения.

*<ИнформацияОбОшибке> (обязательный)* - Структура - при возникновении ошибки содержит подробную информацию о ней.

**Возвращаемое значение:**

Массив - загруженные объекты фотографий, см. https://vk.com/dev/objects/photo

**Доступность:** Клиент, Сервер.

**Описание:**

Загружает фотографию для подборки товаров. Подробнее см. в руководстве разработчика ВКонтакте по адресу: https://vk.com/dev/upload_files
