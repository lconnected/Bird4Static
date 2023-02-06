# Bird4Static
Здесь выложены файлы для работы bird4 с сервисами antifilter.download или antifilter.network

Без BGP, с обычным обновлением раз в час, с возможностью внесения пользовательских правил.

Есть возможность настройки с одним впн, так и с двумя (один основной, второй резервный + пользовательское перенаправление в определенный)

Предназначено для роутеров Keenetic с установленным на них entware, а так же для любой системы с opkg пакетами, и у которых система расположена в каталоге */opt/

## Установка
1) Зайти по ssh в среду entware

2) Выполнить:
```
  opkg install git git-http
  git clone https://github.com/DennoN-RUS/Bird4Static.git
  chmod +x ./Bird4Static/install.sh
  ./Bird4Static/install.sh 
```

3) Далее последует вопрос о количестве впн туннелей. Если у вас их 2 и хотите, что бы они оба использовались, то введите 2. Если второго впн нет, то введите 1 или любое другое значение (приравняется к 1)

4) Потом нужно выбрать режим работы, где 1 это скачивание файла со списком адресов, 2 это работа через BGP, 3 это просто использование пользовательских листов. Все так же выбирается через цифру

5) В зависимости от выбора выше скрипт предложит настройки. Для режима 1 нужно выбрать список, на основе которого будут строится маршруты в впн, но можно указать и свой урл. Для режима 2 нужно выбрать сервис с которым будет устанавливаться BGP соединение (Внимание на третий вариант, там требуется установить [vpn соединение](https://antifilter.network/vpn)). Все так же выбирается через цифру

6) Во время выполнения скрипта потребуется ввести имя интерфейса провайдера и интерфейса/ов VPN. Все данные будут выводиться в консоль перед вводом, так что необходимо только скопировать нужные имена и вставить в консоль

7) После выполнения установки роутер получит маршрутизацию через впн до нужных ресурсов

### ОПЦИОНАЛЬНО:

1) Так же можно принудительно указать ресурсы которые надо пустить через VPN или провайдера. Для этого нужно отредактировать файлы:

      `Bird4Static/lists/user-isp.list` - для перенаправления трафика через провайдера
  
      `Bird4Static/lists/user-vpn.list` - для перенаправления трафика через VPN (в случе использования двух впн маршруты будут добавлены для обоих)

      `Bird4Static/lists/user-vpn1.list` и `Bird4Static/lists/user-vpn2.list` - появляется только при использовании двух впн, указывает к какому адресу идти с определенного впн (по умолчанию 2ip.ru открывается через первый, yoip.ru через второй)
  
      После редактирования надо запустить скрипт обновления
  
      `./Bird4Static/scripts/add-bird4_routes.sh`

      Для разбора файлов с пользовательскими данными используется [iprange](https://manpages.org/iprange) и кастомный скрипт для разбора AS

      Подсказка по правилам заполнения файлов:
  
    - комментарии начинаются с решётки (#) или точки с запятой (;);

    - один IP на строку (без маски);
    
    - CIDR на строку (A.A.A.A/B);

    - диапазон IP-адресов на строку (A.A.A.A - B.B.B.B);
    
    - диапазон CIDR на строку (A.A.A.A/B - C.C.C.C/D); диапазон рассчитывается как сетевой адрес A.A.A.A/B до широковещательного адреса C.C.C.C/D;
    
    - CIDR могут быть заданы либо в формате префикса, либо в формате сетевой маски во всех случаях (включая диапазоны);
    
    - одно имя хоста в строке, разрешаемое с помощью DNS (если IP-адрес разрешается в несколько IP-адресов, все они будут добавлены) имена хостов не могут быть указаны в виде диапазонов;

    - один Autonomous system на строку в виде AS13414 (регистр важен, указывать только большими буквами!);

    - пробелы и пустые строки игнорируются.

2) Так же можно указать несколько урлов задав в переменной `URLS` в файле add-bird4_routes.sh. Пример (кавычки "" важны!): `URLS="https://antifilter.download/list/allyouneed.lst https://community.antifilter.download/list/community.lst"`

3) Так же можно включить режим отладки, для этого в скрипте add-bird4_routes.sh нужно установить переменную DEBUG=1. Информация будет выводится на экран консоли. Выводится информация о том, какой шаг выполняется, и более детальная работа команд diff (выводит изменения, которые накладываются на текущие файлы с маршрутами) и iprange (выводит информацию о суммировании списков и резолв доменов из пользовательских списков). После отладки рекомендуется установить обратно переменную в 0

## Обновление

1) Перейти в папку `Bird4Static`

2) Выполнить
    ```
    git restore install.sh
    git pull
    chmod +x install.sh
    ```
3) Запустить скрипт установки `./install.sh`, главное во время выполнения не соглашаться с перезаписью файлов `user-*.list`
    ```
    cp: overwrite '/opt/root/Bird4Static/lists/user-isp.list'? n
    cp: overwrite '/opt/root/Bird4Static/lists/user-vpn.list'? n
    cp: overwrite '/opt/root/Bird4Static/lists/user-vpn1.list'? n
    cp: overwrite '/opt/root/Bird4Static/lists/user-vpn2.list'? n
    ```
    Если согласится на это, то ваши списки будут заменены дефолтными из репозитория

## Удаление
Для удаления нужно запустить:
```
  chmod +x ./Bird4Static/uninstall.sh
  ./Bird4Static/uninstall.sh 
```
Далее ответить на вопросы, что удалять, а что оставить из дополнительных пакетов

## Известные проблемы

  - Если у вас при заполнении файла user-isp.list перестают открываться ресурсы указанные в нем, то надо изменить переменную в скрипте add-bird4_routes.sh с ISP=eht3 (где eth3 - это интерфейс провайдера) на ISP=10.0.0.1 (где 10.0.0.1 - это шлюз провайдера). Узнать шлюз можно командой `ip route | grep default`

    ВНИМАНИЕ! Сам скрипт не отслеживает какой сейчас шлюз. Если вы указали один, а потом он изменился, то надо снова менять в файле значение переменной ISP и перезапускать скрипт. Так же можно автоматизировать указание шлюза, указав в add-bird4_routes.sh ```ISP=`ip route | grep default | cut -f 3 -d' '` ``` Но при каждой смене шлюза нужно будет все равно запускать скрипт
    
  - После установки во время выполнения скрипта add-bird4_routes.sh вываливается ошибка
    ```
    sh: =~: unknown operand
    BusyBox v1.31.0 () built-in shell (ash)
    ```
    Что бы исправить это нужно обновить busybox командами (добавлено в скрипт установки начиная с версии v3.4.1)
    ```
    opkg update
    opkg upgrade busybox
    ```

Более подробно что и как расписано [здесь](https://forum.keenetic.com/topic/8577-%D0%BE%D0%B1%D1%85%D0%BE%D0%B4-%D0%B1%D0%BB%D0%BE%D0%BA%D0%B8%D1%80%D0%BE%D0%B2%D0%BE%D0%BA-%D1%81-%D0%B8%D1%81%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5%D0%BC-bird4/)

