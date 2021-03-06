### настройка l3fwd

l3fwd занимается маршрутизацией пакетов, но это лишь тестовый пример, поэтому у него даже близко нет нужного фукнционала.
Для его работы нужно в каждый порт подключить по единственному клиенту. Он хоть и маршрутизирует пакеты по IP на основе 
вкомпилированной таблицы, но не умеет arp, поэтому при запуске нужно задать MAC адреса клиентов, с которыми он будет 
общаться на каждом порту (один мак на порт, поэтому на каждом порту один получатель пакетов):
```
./l3fwd -l 1-3 -n 2 -- -p 0x3 -L --config="(0,0,1),(1,0,2)" --eth-dest=0,00:18:7d:ff:33:ac --eth-dest=1,00:02:c9:08:a8:d0
```
Таким образом, l3fwd, получив пакет на свой порт, определит на какой порт его маршрутизировать и установит мак назначения
клиента, MAC источника - свой и выдаст пакет в порт. Поэтому на каждом клиенте нужно прописать маршрутизацию на сеть за 
l3fwd (на другом его интерфейсе) и сделать arp запись (на клиенте) с маком и IP l3fwd. В реальности, этот IP можно брать 
любой, потому что на l3fwd IP адреса нет и на arp он не ответит. Этот IP и его мак нужны клиенту, чтобы он отправил пакет
в сторону порта l3fwd.
Итого, на клиенте нужно
  повесить на интерфейс сеть, которую хочет видеть на данном порту l3fwd,
  прописать маршрутизацию на сеть на другом порту l3fwd, через любой IP в своей подсети, которая висит между клиентом и l3fwd,
  прописать arp запись для этого IP шлюза (потому что на шлюзе он не висит и на arp шлюз не ответит, по сути клиент отправляет
  пакет на mac адрес),
  сделать то же на другом клиенте.

Можно проверять роутинг. Клиент, захотев отправить пакет в другую сеть, будет знать, что его надо отправлять на IP шлюза, а MAC
для этого IP он найдет в статической arp таблице. В результате пакет будет отправлен в порт l3fwd, хотя там нет никакого IP.
l3fwd отмаршрутизирует пакет на другой порт, поменяв там MAC адреса. MAC другого клиента ему известен из командной строки
(причем опять же l3fwd не имеет своей arp таблицы, он будет слать все пакеты для того порта на единственный MAC адрес).

В обратную сторону все аналогично.

