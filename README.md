# Bird4Static
Здесь выложены файлы для работы bird4 с сервисом antifilter.download

Без BGP, с обычным обновлением раз в час, с возможностью внесения пользовательских правил.

Есть возможность настройки с один впн, так и с двумя (один основной, второй резервный + пользовательское перенаправление в определенный)

Предназначено для роутеров Keenetic с установленным на них entware

## Установка
1) Зайти по ssh в среду entware
2) Выполнить:
```
  opkg install git git-http
  git clone https://github.com/DennoN-RUS/Bird4Static.git
  chmod +x ./Bird4Static/install.sh
  ./Bird4Static/install.sh 
```
3) Далее последует вопрос о количестве впн тунеллей. Если у вас их 2 и хотите, что бы они оба использовались, то введите 1. Если второго впн нет, то введите 0 или любое другое значение (приравняется к 0)
4) Во время выполнения скрипта потребуется ввести имя интерфейса провайдера и интерфейса/ов VPN. Все данные будут выводиться в консоль перед вводом, так что необходимо только скопировать нужные имена и вставить в консоль
5) После выполнения установки роутер получит маршрутизацию через впн до нужных ресурсов

### ОПЦИОНАЛЬНО:

6) Так же можно принудительно указать ресурсы которые надо пустить через VPN или провайдера. Для этого нужно отредактировать файлы:

      `Bird4Static/lists/user-isp.list` - для перенаправления трафика через провайдера
  
      `Bird4Static/lists/user-vpn.list` - для перенаправления трафика через VPN (в случе использования двух впн маршруты будут добавлены для обоих)

      `Bird4Static/lists/user-vpn1.list` и `Bird4Static/lists/user-vpn2.list` - появляется только при использвоании двух впн, указывает к какому адресу идти с определенного впн (по умолчанию 2ip.ru отрвыается через первый, yoip.ru через второй)
  
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

7) Если нужно пустить трафик только до ресурсов добавленных в файлы, которые указаны выше, без общего списка c antifilter.download, то нужно закомментровать, т.е. поставить знак # в начале строки URL0=https://antifilter.download/list/allyouneed.lst в файле /opt/etc/cron.daily/add-bird4_routes.sh. Потом нужно заполнить хотя бы один файл из п.6 и выполнить

    `./Bird4Static/scripts/add-bird4_routes.sh`

8) Так же можно включить режим отладки, для этого в скрипте add-bird4_routes.sh нужно установить переменную DEBUG=1. Информация будет выводится на экран консоли. Выводится информация о том, какой шаг выполняется, и более детальная работа команд diff (выводит изменения, которые накладываются на текущие файлы с маршрутами) и iprange (выводит информацию о суммиризации списов и резолв доменов из пользоваательских списков). После отладки рекомандуется установить обратно переменную в 0

## Обновление

1) Выполнить в папке /opt/root/Bird4Static/ команду git pull
2) Запустить скрипт установки, главное во время выполения не соглашаться с перезаписью файлов `user-*.list`
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

  - Если у вас при заполнении файла user-isp.list перестают открываться ресурсы указанные в нем, то надо изменить переменную в скрипте add-bird4_routes.sh с ISP=eht3 (где eth3 - это интерфейс провайдера) на ISP=10.0.0.1 (где 10.0.0.1 - это шлюз провайдера). Узнать шлюз можно командой `ip route | grep default` ВНИМАНИЕ! Сам скрипт не отслеживает какой сейчас шлюз. Если вы указали один, а потом он изменился, то надо снова менять в файле значение переменной ISP и перезапускатть скрипт.

Более подробно что и как расписано [здесь](https://forum.keenetic.com/topic/8577-%D0%BE%D0%B1%D1%85%D0%BE%D0%B4-%D0%B1%D0%BB%D0%BE%D0%BA%D0%B8%D1%80%D0%BE%D0%B2%D0%BE%D0%BA-%D1%81-%D0%B8%D1%81%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5%D0%BC-bird4/)

