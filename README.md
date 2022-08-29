
# Sber ID SDK

IOS SDK помогает реализовать получение кода авторизации (Auth Code) Сбер ID минимальными усилиями со стороны разработчика и в соответствии с утвержденными гайдами по отображению кнопки. Чтобы добавить поддержку Сбер ID в свое приложение, следуйте инструкциям ниже. 

Для выполнения успешных запросов вам необходимо зарегистрировать ваше приложение в банке и подписать договор. Заявку можно оставить по [ссылке](https://developers.sber.ru/portal/tools/sber-id)

### Содержание:

[Авторизация:](#Подключение)

[- Подключение SDK к вашему проекту](#Подключение)

[- Настройка SDK](#Настройка)

[- Добавление кнопки Sber ID](#Добавление)

[- Запуск процесса авторизации по Сбер ID](#Запуск)

[- Обработка ответа после авторизации](#Обработка)

[Ошибки](#Ошибки)

[Кастомизация кнопки](#Кастомизация)


## Подключение SDK к проекту <a name="Подключение"></a> 
 
Перетащите ```SberbankSDK.xcframework``` и ```MPAnalytics.xcframework``` в *Frameworks, Libraries, and Embedded Content*, а также выставите *Embed & Sign* и *Do Not Embed* соответствено

<img src="ReadMeImages/AddFramework.png" width="600">

Отдельно добавьте ```MPAnalyticsDataModel.xcdatamodeld```, этот файл предоставляется в исходном виде и собирается вместе с приложением.
Важно убрать галочку с *Copy items if needed*

<img src="ReadMeImages/image2021-7-23_18-31-47.png" width="550" height="200"> <img src="ReadMeImages/image2021-7-23_18-32-14.png" height="200">
<img src="ReadMeImages/image2021-7-23_18-33-16.png" height="200"> <img src="ReadMeImages/image2020-12-9_16-55-50.png" height="200">

Во вкладке *Build Phases*, в параметрах *Embed Frameworks* проверьте, что добавлен только ```SberbankSDK.xcframework```

<img src="ReadMeImages/image2020-12-16_11-57-7.png" width="600">

## Настройка SDK <a name="Настройка"></a> 

## Добавление кнопки Sber ID <a name="Добавление"></a> 

Импортируйте модуль SberbankSDK.

###### Swift
```swift
import SberbankSDK
```
###### Objective C
```Objective-C
@import SberbankSDK;
```

Создайте кнопку и добавьте её на view.

###### Swift
```swift
/// Инициализатор создаст кнопку по стилистическому гайду Сбербанка с заданными размерами и выбранным заголовком
/// - Parameters:
///   - type: стиль кнопки
///   - textType: вариант текста
///   - clientId: идентификатор клиента
///   - desiredSize: желаемые высота и ширина
///   - observer: наблюдатель состояния кнопки
init(type: LoginButtonStyle,
     textType: LoginButtonTextType,
     clientId: String,
     desiredSize: CGSize,
     observer: LoginButtonObserverProtocol? = nil) {}
     
/// Инициализатор создаст кнопку только по выбранному стилю
/// - Parameters:
///   - type: стиль кнопки
///   - clientId: идентификатор клиента
///   - observer: наблюдатель состояния кнопки
init(type: LoginButtonStyle,
     clientId: String,
     observer: LoginButtonObserverProtocol? = nil) {}
     
// Пример:
let loginButton = SBKLoginButton(type: .white,
                                 clientId: "clientId")
loginButton.addTarget(self, action: #selector(loginButtonDidTap(_:)), for: .touchUpInside)
view.addSubview(loginButton)
```

###### Objective C
```Objective-C
/// Инициализатор создаст кнопку по стилистическому гайду Сбербанка с заданными размерами и выбранным заголовком
/// params:
///   type: стиль кнопки
///   textType: вариант текста
///   clientId: идентификатор клиента
///   desiredSize: желаемые высота и ширина
///   observer: наблюдатель состояния кнопки
- (nonnull instancetype)initWithType:(enum LoginButtonStyle) 
                            textType:(enum LoginButtonTextType)
                            clientId:(NSString * _Nonnull)
                         desiredSize:(CGSize)
                            observer:(id<LoginButtonObserverProtocol> _Nullable) {}

/// Инициализатор создаст кнопку только по выбранному стилю
///   type: стиль кнопки
///   clientId: идентификатор клиента
///   observer: наблюдатель состояния кнопки
- (nonnull instancetype)initWithType:(enum LoginButtonStyle)
                            clientId:(NSString * _Nonnull)
                            observer:(id<LoginButtonObserverProtocol> _Nullable) {}

// Пример:
SBKLoginButton *loginButton = [[SBKLoginButton alloc] initWithType:LoginButtonStyleGreen clientId:"clientId" observer:nil];
[loginButton addTarget:self action:@selector(loginButtonDidTap:) forControlEvents:UIControlEventTouchUpInside];
[self.view addSubview:loginButton];
```

## Запуск процесса авторизации по Сбер ID <a name="Запуск"></a> 

Для того чтобы ваше приложение могло проверить возможность запуска приложения Сбербанк Онлайн в *Info.plist* необходимо добавить следующий параметр:

<img src="ReadMeImages/image2020-12-9_17-18-49.png" width="600">

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
    <string>sberbankidexternallogin</string>
</array>
```

Для успешного запроса авторизации создайте и заполните объект ```SBKAuthRequest``` параметрами. Их описание можно найти в [1.1.2.1. Параметры запроса](https://api.developer.sber.ru/product/SberbankID/doc/v1/reqmobile).

Методы ```auth``` и ```soleLoginWebPageAuth``` создадут уникальную ссылку и откроют мобильное приложение Сбербанк Онлайн (при наличии) либо веб окно для авторизации:

###### Swift
```swift
// Параметры для поддержки PKCE
let verifier = SBKUtils.createVerifier()
let challenge = SBKUtils.createChallenge(verifier)
 
let request = SBKAuthRequest()
request.clientId = "your client id"
request.nonce = "your nonce"
request.scope = "your scope" //Перечесление scope через пробел
request.state = "your state"
request.redirectUri = "myapp://sberidauth"
request.codeChallenge = challenge //Необязательный параметр
request.codeChallengeMethod = SBKAuthRequest.challengeMethod //Необязательный параметр
 
// Запуск авторизации
let loginViewController = UIViewController()
SBKAuthManager.auth(withSberId: request, loginViewController) // Авторизоваться с помощью Сбербанк Онлайн, 
                                                              // если Сбербанк онлайн не установлен открывается
                                                              // веб окно авторизации.
// или
SBKAuthManager.soleLoginWebPageAuth(sberIdRequest: request,           
                                    svcRedirectUrlString: "URL для возврата в safariViewController",
                                    viewController: loginViewController) // Авторизация с помощью веб окна.
```

###### Objective C
```Objective-C
// Параметры для поддержки PKCE
NSString *verifier = [SBKUtils createVerifier];
NSString *challenge = [SBKUtils createChallenge:verifier];
 
SBKAuthRequest *request = [SBKAuthRequest new];
request.clientId = @"your cliend id";
request.nonce = @"your nonce";
request.scope = @"your scope"; //Перечесление scope через пробел
request.state = @"your state";
request.redirectUri = @"myapp://sberidauth";
request.codeChallenge = challenge; //Необязательный параметр
request.codeChallengeMethod = SBKAuthRequest.challengeMethod; //Необязательный параметр
 
// Запуск авторизации
UIViewController *loginViewController = [UIViewController new];
[SBKAuthManager authWithSberId:request viewController:loginViewController]; // Авторизоваться с помощью Сбербанк Онлайн, 
                                                                            // если Сбербанк онлайн не установлен открывается
                                                                            // веб окно авторизации.
// или
[SBKAuthManager soleLoginWebPageAuthWithSberIdRequest:request 
                                 svcRedirectUrlString:@"URL для возврата в safariViewController 
                                       viewController:yourLoginViewController]; // Авторизация с помощью веб окна.
```
*в версиях до 1.3.1 открывается внешний браузер Safari - на данный момент это запрещено Apple.

*в версиях до 2.0.0 возможна установка свойства ```SBKAuthManager.navigationController```, через который будет открыт SafariViewController c web страницей, если приложение Сбербанк Онлайн не может быть запущено (```UIApplication.shared.canOpenURL``` для Сбербанк Онлайн возвращает ```false```)

## Обработка ответа после авторизации <a name="Обработка"></a> 

После авторизации Сбербанк Онлайн перенаправит вас обратно в ваше приложение по адресу, указанному в параметре ```redirectUri``` объекта ```SBKAuthRequest```. Для того чтобы при переходе открылось ваше приложение, необходимо зарегистрировать deeplink(адрес) вышего приложения.

Откройте параметры проекта и перейдите на вкладку *Info*. В нижней части добавьте свой *URL Type*.

<img src="ReadMeImages/Screenshot 2022-08-29 at 15.39.31.png" height="300">


Далее в файле *AppDelegate* прописываем свою логику (см. пример ниже), метод ```application``` вызывается при открытии вашего приложения по deeplink(ссылке). ```SBKAuthManager.getResponseFrom(_ url: URL, completion: (SBKAuthResponse) -> Void)``` вернет объект ```SBKAuthResponse``` с полученными параметрами. 

###### Swift

```swift
func application(_ app: UIApplication, 
		 open url: URL, 
		 options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {     
    if url.scheme == "myapp" && url.host == "sberidauth" {
        SBKAuthManager.getResponseFrom(url) { response in
	    //do something         
        }
    }
    return true
}
```

###### Objective C

```objc
- (BOOL)application:(UIApplication *)app 
	    openURL:(NSURL *)url 
	    options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options
{
    if ([url.scheme isEqualToString:@"myapp"] && [url.host isEqualToString:@"sberidauth"])
    {
        [SBKAuthManager getResponseFrom:url completion:^(SBKAuthResponse *response) {
	    //do something
        }];
    }
    return YES;
}
```

Модель ответа:

###### Swift

```swift
class SBKAuthResponse : NSObject {

    /// Значение, сгенерированное внешней АС для предотвращения атак повторения
    var nonce: String { get }

    /// Значение для предотвращения подделки межсайтовых запросов, случайно сгенерированное
    var state: String? { get }

    /// Код авторизации клиента
    var authCode: String? { get }

    /// Текст ошибки
    var error: String? { get }

    /// Статус операции
    var isSuccess: Bool { get }
}
```
###### Objective C

```objc
@interface SBKAuthResponse : NSObject

/// Значение, сгенерированное внешней АС для предотвращения атак повторения
@property (nonatomic, readonly, copy) NSString * _Nonnull nonce;

/// Значение для предотвращения подделки межсайтовых запросов, случайно сгенерированное
@property (nonatomic, readonly, copy) NSString * _Nullable state;

/// Код авторизации клиента
@property (nonatomic, readonly, copy) NSString * _Nullable authCode;

/// Текст ошибки
@property (nonatomic, readonly, copy) NSString * _Nullable error;

/// Статус операции
@property (nonatomic, readonly) BOOL isSuccess;

- (instancetype)init NS_UNAVAILABLE;
+ (instancetype)new NS_UNAVAILABLE;

@end
```

## Ошибки <a name="Ошибки"></a> 

Пример ответа с ошибкой

```cpp
appScheme://redirect?status=fail&error=invalid_request
```

|№|типы возвращаемых ошибок|описание ошибки|
|:-------:|:-------:|:-------:|
|1|invalid_request|В запросе отсутствуют обязательные атрибуты.|
|2|unauthorized_client|АС-источник запроса не зарегистрирована в банке.|
|3|unauthorized_client|АС-источник запроса заблокирована в банке.|
|4|unauthorized_client|Значение атрибута client_id не соответствует формату.|
|5|unsupported_response_type|Значение атрибута response_type не равно «code».|
|6|invalid_scope|Запрошенный scope содержит значения, недоступные для АС-источника запроса.|
|7|invalid_request|Значение code_challenge_method не соответствуют допустимым значениям.|



## Кастомизация кнопки <a name="Кастомизация"></a> 

Возможные вариации кнопки, а также рекомендуемые размеры можно найти в [гайдбуке по кнопке авторизации](https://www.figma.com/file/S7nPyGmpZFuk0oTOOAYQrQ/%5BBrand%5D-%D0%93%D0%B0%D0%B8%CC%86%D0%B4%D0%B1%D1%83%D0%BA-%D0%BF%D0%BE-%D0%BA%D0%BD%D0%BE%D0%BF%D0%BA%D0%B5-%D0%B0%D0%B2%D1%82%D0%BE%D1%80%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D0%B8?node-id=0%3A1)

Доступные стили кнопки, стиль устанавливается при инициализации кнопки Sber ID:
```swift
LoginButtonStyle.green  // Зеленая стандартная кнопка
LoginButtonStyle.white  // Белая кнопка с прозрачным фоном и обводкой
```

Доступные варианты текста, можно установить во время инициализации:
```swift
LoginButtonTextType.short    // "Сбер ID"
LoginButtonTextType.general  // "Войти по Сбер ID"
LoginButtonTextType.filling  // "Заполнить со Сбер ID"
LoginButtonTextType.pursue   // "Продолжить со Сбер ID"
```

Степень скругления кнопки можно установить с помощью метода:
```swift
func setCornerRadius(_ radiusStyle: CornerRadiusStyle)
CornerRadiusStyle.no // отсутствует
CornerRadiusStyle.normal // радиус 4
CornerRadiusStyle.max // радиус равен половине высоты кнопки
```
*при необходимости можно установить свои параметры через ```SBKLoginButton().layer.cornerRadius```

Установить цвет обводки кнопки Sber ID (используется для белого стиля кнопки), поможет метод:
```swift
func setBorderColor(_ color: UIColor)
```
### Кастомизация кнопки

**Тип**

По умолчанию используется стандартный вид кнопки:

![Скриншот кнопки 2](ReadMeImages/buttonsecond.png ':size=600')

Для изменения стиля кнопки, укажите параметр type: ."значение из списка" кнопки в SBKLoginButton.

![Скриншот кода кнопки](ReadMeImages/codd.png ':size=600')

Значение **white** соответствует белой кнопке с серой обводкой:

![Скриншот кнопки 3](ReadMeImages/white.png ':size=600')

**Обводка**

В случае установки типа кнопки "white" есть возможность указать значение цвета обводки для соответствия вашему дизайну, если на экране присутствует несколько однотипных кнопок. Для этого установите конкретный цвет через атрибут кнопки setBorderColor(. “значение из списка”)

![Скриншот выбора обводки](ReadMeImages/second.png ':size=600')

![Скриншот выбора обводки 2](ReadMeImages/third.png ':size=600')

При установке цвета обводки для стандартной зеленой кнопки значение будет проигнорировано.

**Скругление**

Для изменения скругления углов кнопки, укажите параметр layer.cornerRadius кнопки layer.cornerRadius="значение".

![Скриншот обводки 2](ReadMeImages/screen.png ':size=600')

Для установки стандартных значений скругления, укажите параметр setCornerRadius кнопки. Значения max, normal и no соответствуют значениям: половина высоты кнопки, 4.0 и 0 соответственно.

![Скриншот обводки 3](ReadMeImages/code.png ':size=600')

![Скриншот скругления](ReadMeImages/buttonnew.png ':size=600')


**Текст**

Для изменения текста кнопки, укажите параметр **textType** кнопки SBKLoginButton(type: ..., textType: значение, desiredHeight: ..., desiredWidth: ...).

*** Соответствие **значения** и текста:
- .short      => "Сбер ID"
- .general  => "Войти по Сбер ID"
- .filling      => "Заполнить со Сбер ID"
- .pursue   => "Продолжить со Сбер ID"

**Значение по умолчанию** - Войти по Сбер ID. Установка собственного текста, шрифта, цвета текста не поддерживается.

``Тексты поддерживают английскую локализацию на устройстве. При выборе локализации, отличной от русской и английской, тексты будут на русском.``


![Скриншот языков](ReadMeImages/button.png ':size=600')
![Скриншот языков 2](ReadMeImages/seven.png ':size=600')


_В примере кнопки с текстом "Сбер ID" и “Sber ID” отрисована с минимально допустимой шириной_

### Поддержка бесшовной авторизации

Начиная с версии SDK 1.4.0 реализована поддержка бесшовной авторизации по Сбер ID, когда после перехода из приложений СберБанка в ваше приложение необходимо запустить авторизацию без показа кнопки и необходимости пользователю выполнять лишнее действие по нажатию на нее.
В диплинке, который придет в ваше приложение для старта процесса авторизации пользователя, помимо параметров, которые вы заложите в него, приходит дополнительный параметр, содержащий строку со схемой и хостом, которую необходимо передать в объект SBKAuthRequest(при инициализации или после, присвоив значение свойству **ssoBaseUrl**) перед запуском процесса авторизации

``Для стандартной (не бесшовной) авторизации по Сбер ID по кнопке выполнять указанные в этом пункте действия не требуется``


Чтобы получить значение этого параметра, необходимо воспользоваться методом:

func getSSOUrlStringFrom(_ url: URL?) -> String?

класса **SBKUtils**, передав в него исходный **uri**, полученный при переходе в ваше приложение в сценарии бесшовной авторизации.Полученное значение необходимо передать в свойство **ssoBaseUrl** при построении объекта **SBKAuthRequest**.

**Swift**

```cpp
/// Получение параметра ssoBaseUrl
let ssoBaseUrl = SBKUtils.getSSOUrlStringFrom(receivedUrl)

/// Присваивание параметра ssoBaseUrl свойству объекта SBKAuthRequest
let request = SBKAuthRequest(clientId: "client-ID",
                             scope: "scope",
                             state: "state",
                             nonce: "nonce",
							 ssoBaseUrl: ssoBaseUrl,
							 redirectUri: "https://testRedirect.url",
							 codeChallenge: "challenge",
							 codeChallengeMethod: SBKAuthRequest.challengeMethod)

/// ИЛИ:
let request = SBKAuthRequest()
/// …
request.ssoBaseUrl = ssoBaseUrl

/// Запуск авторизации
SBKAuthManager.auth(withSberId: request)
```

Все дальнейшие действия по подготовке диплинка и старте авторизации аналогичны описанным в разделе «Запуск процесса авторизации по Сбер ID».


### Авторизация через единый web-портал авторизации по Сбер ID

В версии SDK 1.4.0 была добавлена возможность авторизации пользователя по Сбер ID, используя единое веб окно авторизации.

**Как это работает:**

- Необходимо направить запрос на support@ecom.sberbank.ru на добавление deeplink в список доверенных. В запросе указывается client_id и список deeplink, по которым будет производиться возврат в мобильное приложение партнера. Сотрудник банка добавит домен в список разрешенных.
- Создайте запрос, как при обычном входе.
- Передайте в SBKAuthManager.navigationController необходимый контроллер навигации, через который будет открыт SafariViewController.
- Запустите OIDC авторизацию через метод soleLoginWebPageAuth с дополнительным параметром svcRedirectUrlString.

**Swift**

```cpp
SBKAuthManager.navigationController = navigationController
let request = SBKAuthRequest()
/// Наполнение request данными …

let result = SBKAuthManager.soleLoginWebPageAuth(sberIdRequest: request,
svcRedirectUrlString: "https://yourApp.url/redirectFromSVCExample")

```

Если запуск сценария невозможен, вернется false.

- Если все сделано верно, будет открыт портал Сбер ID в SafariViewController  с различными способами идентификации входа по Сбер ID:

![Скриншот кнопки](ReadMeImages/image2021-7-23_18-18-2.png ':size=600')

Новый параметр **svcRedirectUrlString** используется для передачи «активности» в ваше приложение из SafariViewControllera-a. После прохождения авторизации на портале и возврата в ваше приложение(по диплинку из **svcRedirectUrlString**) процесс авторизации продолжит работу по стандартному сценарию OIDC. В ваше приложение вернется AuthCode и другие параметры через диплинк, переданный в SBKAuthRequest().redirectUri. Вам необходимо закрыть SafariViewController самостоятельно.
Дальнейшие шаги процесса авторизации описаны в разделе «Запуск процесса авторизации по Сбер ID»

### Персонализация кнопки

Начиная с версии SDK 1.2.0 кнопка входа по Сбер ID автоматически поддерживает персонализацию. Персонализация заключается в автоматическом изменении текста в кнопке на текст, содержащий информацию об имени и фамилии пользователя. Все остальные параметры, которые вы указали при добавлении кнопки, остаются без изменений.

Условия для работы функционала персонализации кнопки:
- на устройстве установлено МП СБОЛ версии от 11.9 и выше
- в МП СБОЛ включен функционал персонализации
- ваше приложение входит в группу разработчика сбербанк
- Проведены настройки в вашем приложении

Если персонализация на стороне МП СБОЛ отключена либо на устройстве не установлено МП СБОЛ, текст на кнопке будет отображаться тот, который вы указали при создании кнопки.

_Если кнопка была добавлена с коротким текстом "Сбер ID", персонализация кнопки не будет запущена в любом случае. Если дизайн экрана предполагает наличие широкой кнопки входа по Сбер ID, рекомендуется использовать один из длинных вариантов текста._

При персонализации кнопка выглядит следующим образом:

![Скриншот перскнопки](ReadMeImages/noname.png ':size=600')
![Скриншот перскнопки 2](ReadMeImages/eight.png ':size=600')

Если имя пользователя слишком длинное, чтобы уместиться в отведенное размерами кнопки место, итоговый текст будет автоматически обрезан с правой стороны

![Скриншот перскнопки 3](ReadMeImages/nine.png ':size=600')

**Настройки для персонализации кнопки**

Для включения возможности персонализации кнопки необходимо:
- ваше приложение должно публиковаться в AppStore от группы разработчика "Сбербанк России"
- в настройках проекта, targets → <название основного таргета> → Signing&Capabilities должен быть включён функционал Keychain Sharing

![Скриншот перскнопки 4](ReadMeImages/image2021-3-9_15-44-43.png ':size=600')

- в настройках Keychain Sharing необходимо добавить keychain group c именем $(AppIdentifierPrefix)ru.sberbank.onlineiphone.shared, должен появиться файл с расширением .entitlements(если такового не было), в котором, среди прочих, будет данный блок:

```xml
	<key>keychain-access-groups</key>
	<array>
		<string>$(AppIdentifierPrefix)ru.sberbank.onlineiphone.shared</string>
	</array>
```

![Скриншот перскнопки 6](ReadMeImages/image2021-3-9_15-47-29.png ':size=600')

- в info.plist проекта необходимо добавить пару ключ-параметр, в RunTime он будет заменен на TeamId: **AppIdentifierPrefix ↔ $(AppIdentifierPrefix)**

![Скриншот перскнопки 7](ReadMeImages/image2021-5-28_18-10-33.png ':size=600')

```xml
	<key>AppIdentifierPrefix</key>
	<string>$(AppIdentifierPrefix)</string>
```



