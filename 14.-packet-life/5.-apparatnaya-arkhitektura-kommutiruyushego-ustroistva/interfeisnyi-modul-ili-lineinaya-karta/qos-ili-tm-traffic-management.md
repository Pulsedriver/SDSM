# QoS или TM — Traffic Management

Иногда в самом FE, иногда как отдельный чип, дальше идёт чип QoS, совмещённый с очередью, вместе обычно носящие название Traffic Management.  
Входная очередь \(очередь на входном тракте\) нужна для того, чтобы не переполнить выходную \(очередь на выходном тракте\).  
Выходная очередь предназначена для избежания явления, известного, как Back Pressure — когда на чип FE пакеты поступают быстрее, чем он в состоянии обработать. Такая ситуация невозможна с Ingress FE, потому что к нему подключено такое количество интерфейсов, что трафик от них он в состоянии переварить, либо Ethernet через Flow Control возьмёт ситуацию под свой Control.  
А вот на Egress FE трафик может сливаться со многих разных плат \(читай Ingress FE\) — и ему захлебнуться — это как два байта переслать.  
Задача очереди не только сгладить всплески трафика, но и управляемо дропать пакеты, когда это становится неизбежным. А именно — выкидывать из очереди низкоприоритетные пакеты с бо́льшей вероятностью, чем высокоприоритетные. Причём отслеживать перегрузку желательно на уровне интерфейсов — ведь если через дестятигигабитный интерфейс нужно отправить 13 Гб/с трафика, то 3 из них однозначно будет отброшено, а четырёхсот-гигабитный FE при этом даже близок к перегрузке не будет.  
  
Схема достаточно усложняется — две очереди, а значит, двойная буферизация, более того как-то надо по интерфейсам их подробить, встаёт ещё вопрос такой: а если один интерфейс перегружен, то вся входная очередь встанет?  
Эти сложности никак не разрешались ранее, однако сегодня они адресованы механизму VOQ — Virtual Output Queue. VOQ прекрасно описан вот [в этой заметке](https://www.juniper.net/documentation/en_US/junos/topics/concept/cos-qfx-series-voq-understanding.html).  
В двух словах — это виртуализация всех очередей между различными FE. Имеется один физический чип памяти DRAM на входном тракте, который внутри разбит на виртуальные очереди. Количество входных очередей — по общему числу выходных. Выходная очередь больше не распологается реально на выходном модуле — она в том же самом DRAM — только виртуальная.  
  
Таким образом \(возьмём пример Juniper\), если есть 72 выходных интерфейса по 8 очередей на каждом, итого получается 576 входных очередей на каждом интерфейсном модуле \(читай TM\). Если на устройстве 6 модулей, то оно должно поддерживать 3456 VOQ.  
Это элегантно снимает вопрос двойной буферизации и проблем [Head of Line Blocking](https://en.wikipedia.org/wiki/Head-of-line_blocking), когда одна выходная очередь в момент перегрузки блокирует всю физическую входную — теперь с VOQ только ту виртуальную, которая с ней связана.

![](../../../.gitbook/assets/image%20%2853%29.png)

![](../../../.gitbook/assets/image%20%28172%29.png)

Кроме того пакет теперь отбрасывается при необходимости на входной очереди, и не нужно его отправлять на фабрику и забивать выходные очереди.  
  
Что ещё важно знать про очереди, так это то, что даже те пакеты, которые предназначены на другой интерфейс этого же FE, должны пройти через входную и выходную очереди.  
Это нужно для той же самой борьбы с Back Pressure. Только очереди могут защитить FE от перегрузок и отбрасывать лишний трафик согласно приоритетам, поэтому никакого прямого мостика для транзитного трафика между Ingress FE и Egress FE не предусмотрено.  
На фабрику однако такой «локальный» трафик попадать не должен.  
Но про QoS мы ещё поговорим в следующей части.
