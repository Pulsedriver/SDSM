# GRE over IPSec

У начинающих тут часто случается конфуз \(_он и у автора случился_\): GRE over IPSec или IPSec over GRE. В чём разница, где применяются. Нельзя на этом не остановиться.  
Обычный режим, который мы рассматриваем тут и который применяется в подавляющем большинстве случаев, – это GRE over IPSec, то есть данные GRE инкапсулируются заголовками ESP или AH

![](http://img-fotki.yandex.ru/get/4136/83739833.24/0_ac120_38239cd3_XL.jpg)

А IPSec over GRE означает, наоборот, что внутри будут зашифрованные данные IPSec, а сверху заголовки GRE/IP. Они будут не зашифрованы:

![](http://img-fotki.yandex.ru/get/4134/83739833.24/0_ac122_7c031dee_XL.jpg)

Такой вариант возможен, например, если шифрование у вас происходит на отдельном устройстве перед туннелированием

![](http://img-fotki.yandex.ru/get/5641/83739833.24/0_ac121_d8dc78dd_XL.jpg)

Зачем такая пахабщина нужна, не очень понятно, поэтому обычно используется именно GRE over IPSec.
