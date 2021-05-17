### hw3.7
1. На лекции мы обсудили, что манипулировать размером окна необходимо для эффективного наполнения приемного буфера  
участников TCP сессии (Flow Control). Подобная проблема в полной мере возникает в сетях с высоким RTT. Например,  
если вы захотите передать 500 Гб бэкап из региона Юга-Восточной Азии на Восточное побережье США. Здесь вы можете  
увидеть и 200 и 400 мс вполне реального RTT. Подсчитайте, какого размера нужно окно TCP чтобы наполнить 1 Гбит/с  
канал при 300 мс RTT (берем простую ситуацию без потери пакетов). Можно воспользоваться готовым калькулятором.  
Ознакомиться с формулами, по которым работает калькулятор можно, например, на Wiki.  
```
TCP Window Size=(Link Speed in bits/sec * RTT)/8 = (1000000000*0.3)/8= 37500000 Байт
```
---

2. Во сколько раз упадет пропускная способность канала, если будет 1% потерь пакетов при передаче?  
```
Рассчитывается по уравнению Матиаса: 
Максимальная скорость в бит/с <= (MSS/RTT) × (1/квадратный корень (p)),
p - вероятность потери пакета.
если за p=0 взять p=0,000001 то корень квадратный из 1/(корень из 0,000001)=1000
если p(вероятность=0,01), то корень квадратный из 1/(корень из 0,001)=10
т.е. пропускная способность уменьшится в 100 раз
```
---

3. Какая максимальная реальная скорость передачи данных достижима при линке 100 Мбит/с? Вопрос про TCP payload, то есть цифры,  
которые вы реально увидите в операционной системе в тестах или в браузере при скачивании файлов. Повлияет ли размер фрейма на это?  
```
100Mbs=12500000byte\s
1 frame =1518 byte
IFG (Interframe gap) = 12 byte.

количество кадров за секунду = Скорость канала/(frame +IFG)=12500000byte\sec/(1518byte+12byte)=8170 frames in sec
MSS=Eth frame - Eht header - IP header - TCP header - Eth trailer= 1518-14-20-20-4=1460byte
8170*1460= 11 928 200 bytes in sec = 12MByte/sec
Размер фрейма повлияет, т.к. при заданном TCP windows size, кол-во кадров отправляемых без подтверждения уменьшится -> уменьшится 
количество служебной информации.
```
---

4. Что на самом деле происходит, когда вы открываете сайт? :) На прошлой лекции был приведен сокращенный вариант ответа на этот вопрос.  
Теперь вы знаете намного больше, в частности про IP адресацию, DNS и т.д. Опишите максимально подробно насколько вы это можете сделать,  
что происходит, когда вы делаете запрос curl -I http://netology.ru с вашей рабочей станции. Предположим, что arp кеш очищен, в локальном  
DNS нет закешированных записей.  
```
curl с запросом ip адреса обращается к резолверу. Резолвер в зависимости от настроек(в /etc/nsswitch.conf),  
ищет в /etc/hosts, системном кэше dns(systemd-resolver, libss). По условию, в системном кэше ничего нет.  
Резолвер обращается к серверу dns, указанному в настройках. Создается сокет(порт назначения 53),  
формируется UDP дейтаграмма, передаётся на уровень IP с IP адресом сервера DNS.  
IP протокол, по адресу сервера DNS, определяет находится ли он в той же подсети, создает пакет с IP адресом  
сервера DNS в поле IP адреса назначения. Если DNS сервер не в нашей подсети, на уровне 2 создается кадр с  
MAC адресом шлюза по умолчанию. Пакет передается на 2-й уровень. На втором уровне создается кадр с MAC шлюза  
по умолчанию, протокол ARP смотрит кэш arp, если там нет записи с нужным IP, отправляет broadcast запрос.  
После получения нужного MAC шлюза , создается Ethernet кадр и передается на физ. уровень.  
После получения ответа на DNS запрос, ответный кадр проходит цепочку в обратном направлении,  
curl получает нужный IP.  
Создается сетевой сокет, после создается сессия, на транспортном уровне(TCP) формируется сегмент для  
передачи, передается на IP уровень,  
формируется пакет с полученным IP адресом netology.ru.  
Создается кадр Ethernet, отправляется на шлюз по умолчанию.  
```
---
5. Сколько и каких итеративных запросов будет сделано при резолве домена www.google.co.uk?  
```
Первый запрос к корневому серверу имен. Он отдает адрес DNS сервера ответственного за зону uk.  
Второй запрос К TLD серверу с зоной uk. Он отдает адрес сервера ответственного за co.uk.  
Третий запрос к серверу с зоной co.uk. Он отдает адрес сервера ответственного за google.co.uk.  
Четвертый запрос к серверу google.co.uk. Он отдает адрес www.google.co.uk.  
```
---

6. Сколько доступно для назначения хостам адресов в подсети /25? А в подсети с маской 255.248.0.0. Постарайтесь потренироваться  
в ручных вычислениях чтобы немного набить руку, не пользоваться калькулятором сразу.  
```
Подсеть /25: Под адреса хостов остается 7 бит. 2 в 7 степени 128. Отнимаем 2 адреса под адрес сети и броадкаст, получаем 126.
Подсеть 255.248.0.0: Маска подсети /13. Под адреса хостов остается 19 бит. 2 в 19 степени 524288. Отнимаем 2 адреса под адрес сети и 
броадкаст, получаем 524286.
```
---

7. В какой подсети больше адресов, в /23 или /24?  
```
В сети /23 под адреса хостов оставлено больше битов -> количество хостов в подсети /23 больше.  
```
---
8. Получится ли разделить диапазон 10.0.0.0/8 на 128 подсетей по 131070 адресов в каждой? Какая маска будет у таких подсетей?  
```
Для того, чтобы разбить эту подсеть на 128 подсетей нужно будет под адресацию подсетей занять еще  7 бит, 
т.е. маска будет /15. Под адреса хостов в каждой подсети остается 17 бит. 2 в 17 степени = 131072. Отнимаем 2 адреса под адрес сети и броадкаст, получаем 131070.
Да, получится.
```
