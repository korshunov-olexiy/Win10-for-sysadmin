# Настройка политики безопасности в Windows 10 Pro, Enterprise или Education:
* ### __Ограничиваем запуск программ в системе__:
  * открываем __gpedit.msc__ под администратором
  * здесь открываем `"Конфигурация пользователя"`->`"Административные шаблоны"`->`"Система"`->`"Выполнять только указанные приложения Windows"`
  * указываем ВСЕ программы, которые должны запускаться в системе (например _notepad.exe_)
  * открываем `"Конфигурация компьютера"`->`"Конфигурация Windows"`->`"Параметры безопасности"`->`"Политики ограниченого использования программ"`- `"Назначенные типы файлов"` и дописываем туда расширения файлов: `VBS`, `VBA`, `JS`, `WSH` и т.п.
* ### __Указываем каталоги, в которых будут запускаться или блокироваться программы в системе__:
  * открываем __secpol.msc__ под администратором
  * переходим в `"Параметры безопасности"`->`"Политики ограниченого использования программ"`. Это делается первый раз в системе, то нужно нажать на этом пункте правую кнопку мыши и выбрать "Создать политики"
  * в разделе `"Уровни безопасности"` нажать правой кнопкой мыши на свойство `"Запрещено"` и выбрать `"По умолчанию"`
  * открыть раздел `"Дополнительные правила"` и добавить каталоги, в которых будет разрешено запускать программы:
    * %WINDIR%\explorer.exe - Неограниченный
    * %WINDIR%\system32\userinit.exe- Неограниченный
    * %WINDIR%\System32\winlogon.exe - Неограниченный
    * C:\ProgramData\Microsoft\Windows\Start Menu\Programs - Неограниченный
    * C:\Users - Запрещено
  * также можно добавить правило для хэша, которое будет срабатывать для указанного исполняемого файла в независимости от того где он расположен
  * переходим в `"Политики управления приложениями"`->`"AppLocker"`->`"Правила сценариев"` и разрешаем/запрещаем сценарии в определенных каталогах
* ### __Запрещаем установку съемных устройств__
  * открываем __gpedit.msc__ под администратором
  * переходим в `"Конфигурация компьютера"`->`"Административные шаблоны"`->`"Система"`->`"Установка устройств"`->`"Ограничения на установку устройств"` - `"Запретить установку съемных устройств"` - указать `"Включено"`
* ### __Ужесточаем требования к паролям__
  * открываем __secpol.msc__ под администратором
  * переходим в `"Параметры безопасности"`->`"Политика учетных записей"`->`"Политика паролей"` и устанавливаем:
	  * `"Минимальная длина пароля"` - __12__
	  * `"Пароль должен отвечать требованиям сложности"` - `"Включен"`

## Настраиваем политику выполнения скриптов PowerShell:
* будем использовать политику `AllSigned`, которая запрещает выполнение любых не подписанных скриптов в системе:
  * открываем `powershell` под администратором
  * пишем в консоли `Set-ExecutionPolicy AllSigned` и подтверждаем - `A`
### Создаем сертификат и подписываем им свои скрипты:
* #### Создаем собственный цифровой сертификат:
  * открываем `powershell` под администратором
  * создаем сертификат: `$cert = New-SelfSignedCertificate -CertStoreLocation cert:\currentuser\my -Subject "CN=Local Code Signing" -KeyAlgorithm RSA -KeyLength 2048 -Provider 'Microsoft Enhanced RSA and AES Cryptographic Provider' -KeyExportPolicy Exportable -KeyUsage DigitalSignature -Type CodeSigningCert`
  * открываем консоль управления сертификатами - `certmgr /s my`
  * копируем созданный сертификат из `"Личное"`->`"Сертификаты"` в `"Доверенные корневые центры сертификации"`->`"Сертификаты"`
  * Экспортируем сертификат в формат `*.pfx` - жмем правой кнопкой мышки на созданном сертификате и выбираем экспорт (обязательно указываем пароль для доступа к сертификату)
* #### Подписываем наши скрипты ps1 своим сертификатом:
  * укажем сертификат для подписания: `$cert = Get-PfxCertificate -FilePath C:\Cert\localhost.pfx`
  * какой скрипт нужно подписать: `$file = "C:\Scripts\superscript.ps1"`
  * подписываем: `Set-AuthenticodeSignature $file $cert`
