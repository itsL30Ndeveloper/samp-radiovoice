# **RADIOVOICE**
Indonesia | [English](https://github.com/itsL30Ndeveloper/samp-radiovoice/blob/main/README.md) | [Русский](https://github.com/itsL30Ndeveloper/samp-radiovoice/blob/main/README.ru.md)

## Описание
---------------------------------
**RADIOVOICE** - является дополнительной функцией для реализации системы голосового общения на языке _Pawno_ для сервера SA:MP системы **SAMPVOICE**.

#### Поддержка версий
----------------------------------
* Client: SA:MP 0.3.7 (R1, R3)
* Server: SA:MP 0.3.7 (R2)

## Funktsiya
---------------------------------
* Контролируемая передача голоса
* Управление микрофоном игрока
* Привязка звукового потока к игровому объекту
* Удаленный голос
* Звуковая частота

## Установка
---------------------------------
Для работы плагина он должен быть установлен игроком и на сервере. Для этого в плагине есть клиентская и серверная части.

#### Для игроков
---------------------------------
Игрокам доступно 2 варианта установки: автоматическая (через установщик) и ручная (через архив).
##### Secara otomatis
---------------------------------
1. Чтобы загрузить установщик, перейдите на [страницу выпуска] (https://github.com/CyberMor/sampvoice/releases) и выберите нужную версию плагина.
2. После загрузки запустите установщик и выберите нужный язык для вашей установки, после чего установщик автоматически найдет вашу папку GTA San Andreas.
3. Если каталог правильный, нажмите «ОК» и дождитесь завершения установки. После завершения установки установщик завершит работу.

##### Вручную
----------------------------------
1. Перейдите на [страницу релизов](https://github.com/CyberMor/sampvoice/releases) и скачайте архив с нужной версией клиента.
2. Распакуйте архив в папку с GTA San Andreas.

#### Для разработчиков
----------------------------------
1. Загрузите со [страницы выпуска] (https://github.com/CyberMor/sampvoice/releases) нужную версию плагина для вашей платформы.
2. Распаковать архив в корневой каталог сервера.
3. Добавьте в файл конфигурации сервера *server.cfg* строки *"plugins sampvoice"* для *Win32* и *"plugins sampvoice.so"* для *Linux x86*. **(Если у вас есть плагин Pawn.RakNet, обязательно поставьте после него SampVoice)**

## Использовать
----------------------------------
Чтобы начать использовать плагин, прочтите документацию, которая идет вместе с серверной частью. Для этого откройте файл *sampvoice.chm*, используя справочник Windows. **(Если документация не открывается, щелкните правой кнопкой мыши файл документации, затем Свойства -> Разблокировать -> ОК)**

Чтобы начать использовать функциональность плагина, подключите заголовочный файл:
```php
#include <sampvoice>
#include <sscanf2>
#include <a_samp>
#include <zcmd>
```
#### Краткий справочник
----------------------------------
Вы должны знать, что плагин использует свой собственный тип констант и систему. Несмотря на то, что это всего лишь оболочка для базового типа Pawn, она помогает ориентироваться в самом типе плагина и не путает указатели.

Для перенаправления аудиотрафика с проигрывателя А на проигрыватель Б необходимо создать аудиопоток (например, глобальный поток, используя **SvCreateGStream**), а затем прикрепить его к потоковому проигрывателю А в качестве динамика (используя **SvAttachSpeakerToStream*). *), затем присоедините его к потоку игрока B в качестве слушателя (используя **SvAttachListenerToStream**). Законченный. Теперь при активации микрофона игрока А (например, с помощью функции **SvStartRecord**) его аудиотрафик будет передаваться и затем прослушиваться игроком Б.

Звуковой поток весьма полезен. Их можно визуализировать на примере Discord:
* Стрим - аналог комнаты (или канала).
* Говорящий — это участник в комнате, у которого отключен звук, но включен микрофон.
* Слушатели — это участники в комнате, микрофоны которых отключены, но отключены.

Игроки могут быть говорящими и слушающими одновременно. В этом случае аудиотрафик игрока не будет ему переадресовываться.

#### Пример
----------------------------------
Давайте рассмотрим некоторые функции плагина на практическом примере. Ниже мы создадим сервер, который будет привязывать всех подключенных игроков к глобальному потоку, а также создадим локальный поток для каждого игрока. Таким образом, игроки смогут общаться через глобальный (звук одинаково слышен в любой точке карты) и локальный чат (слышен только рядом с игроком).
```php
#include <a_samp>
#include <zcmd>
#include <sampvoice>
#include <sscanf2>

#define MAX_RADIOS 9999 //maximum radio frequency channel

new SV_GSTREAM:RadioChannel[MAX_RADIOS] = SV_NULL; 
new FreqNumber[MAX_PLAYERS]; //frequency number
new SV_GSTREAM:gstream = SV_NULL;
new SV_LSTREAM:lstream[MAX_PLAYERS] = { SV_NULL, ... };

/*
    The public OnPlayerActivationKeyPress and OnPlayerActivationKeyRelease
    are needed in order to redirect the player's audio traffic to the
    corresponding streams when the corresponding keys are pressed.
*/

public SV_VOID:OnPlayerActivationKeyPress(SV_UINT:playerid, SV_UINT:keyid) 
{
    // Attach player to local stream as speaker if 'B' key is pressed
    if (keyid == 0x42 && lstream[playerid]) SvAttachSpeakerToStream(lstream[playerid], playerid);
    // Attach the player to the global stream as a speaker if the 'Z' key is pressed
    if (keyid == 0x5A && gstream) SvAttachSpeakerToStream(gstream, playerid);
    // Attach player to radio stream as speaker when 'B' button is pressed
    if(keyid == 0x42 && FreqNumber[playerid] >= 1) SvAttachSpeakerToStream(RadioChannel[FreqNumber[playerid]], playerid);
    if(keyid == 0x42 && FreqNumber[playerid] == 0) return;
}

public SV_VOID:OnPlayerActivationKeyRelease(SV_UINT:playerid, SV_UINT:keyid)
{
    // Detach the player from the local stream if the 'B' key is released
    if (keyid == 0x42 && lstream[playerid]) SvDetachSpeakerFromStream(lstream[playerid], playerid);
    // Detach the player from the global stream if the 'Z' key is released
    if (keyid == 0x5A && gstream) SvDetachSpeakerFromStream(gstream, playerid);
    // Detach the player from the radio stream if the 'B' key is released
    if(keyid == 0x42 && FreqNumber[playerid] >= 1) SvDetachSpeakerFromStream(RadioChannel[FreqNumber[playerid]], playerid);
    if(keyid == 0x42 && FreqNumber[playerid] == 0) return;
}

public OnPlayerConnect(playerid)
{
    // Checking for plugin availability
    if (SvGetVersion(playerid) == SV_NULL)
    {
        SendClientMessage(playerid, -1, "Could not find plugin sampvoice.");
    }
    // Checking for a microphone
    else if (SvHasMicro(playerid) == SV_FALSE)
    {
        SendClientMessage(playerid, -1, "The microphone could not be found.");
    }
    // Create a local stream with an audibility distance of 40.0, an unlimited number of listeners
    // and the name 'Local' (the name 'Local' will be displayed in red in the players' speakerlist)
    else if ((lstream[playerid] = SvCreateDLStreamAtPlayer(40.0, SV_INFINITY, playerid, 0xff0000ff, "Local")))
    {
        SendClientMessage(playerid, -1, "Press Z to talk to global chat and B to talk to local chat.");

        // Attach the player to the global stream as a listener
        if (gstream) SvAttachListenerToStream(gstream, playerid);

        // Assign microphone activation keys to the player
        SvAddKey(playerid, 0x42);
        SvAddKey(playerid, 0x5A);
    }
}

public OnPlayerDisconnect(playerid, reason)
{
    // Removing the player's local stream after disconnecting
    if (lstream[playerid])
    {
        SvDeleteStream(lstream[playerid]);
        lstream[playerid] = SV_NULL;
    }
}

public OnGameModeInit()
{
    // Uncomment the line to enable debug mode
    // SvDebug(SV_TRUE);

    gstream = SvCreateGStream(0xffff0000, "Global");
}

public OnGameModeExit()
{
    if (gstream) SvDeleteStream(gstream);
}

CMD:setfreq(playerid, params[])
{
	new id;
	if(sscanf(params, "d", id)) return SendClientMessage(playerid, -1, "Use: /setfreq [frequensi] | 0 = turn off");

	FreqNumber[playerid] = id;

	if(RadioChannel[FreqNumber[id]] == SV_NULL)
	{
		RadioChannel[FreqNumber[id]] = SvCreateGStream(0xFF00FFFF, "Radio");
		SvAttachListenerToStream(RadioChannel[FreqNumber[id]], playerid);
 	}
 	else
 	{
    	SvAttachListenerToStream(RadioChannel[FreqNumber[id]], playerid);
	}
	return 1;
}

```

## компилировать
----------------------------------
Скомпилируйте плагины для платформ *Win32* и *Linux x86*.

Ниже приведены дальнейшие инструкции:

Клонируйте репозиторий на свой компьютер и перейдите в каталог плагинов:
```sh
git clone https://github.com/CyberMor/sampvoice.git
cd sampvoice
```

### Windows (клиент/сервер)
----------------------------------
Чтобы скомпилировать клиентскую часть плагина, вам понадобится *DirectX SDK*. По умолчанию клиентская часть компилируется для версии **SA:MP 0.3.7 (R1)**, но вы также можете явно указать компилятору версию для сборки с помощью макросов **SAMP_R1** и **SAMP_R3** . Чтобы собрать клиентскую и серверную части плагина для платформы *Win32*, откройте проект *sampvoice.sln* в MS Visual Studio 2019 и скомпилируйте:
> Построить -> Построить решение (F7)

### Linux (серверы)
----------------------------------
Чтобы собрать серверную часть плагина для платформы *Linux x86*, следуйте этим инструкциям:
```sh
cd server
make
```
