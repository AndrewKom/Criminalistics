# Volatility Framework

# Криминалистический анализ оперативной памяти

Платформа Volatility Framework представляет собой полностью открытый
набор инструментов, реализованный на Python под лицензией GNU General
Public License для извлечения цифровых артефактов. В данной работе она
будет использована для анализа дампов оперативной памяти с вредоносной
нагрузкой.

## Установка Volatility Framework

Необходимо произвести установку необходимой версии python для правильной
работы фреймворка.

    wget https://bootstrap.pypa.io/pip/2.7/get-pip.py

![](images/clipboard-480825925.png)

Произведем установку pip для python2

    sudo python2 get-pip.py

![](images/clipboard-3217783170.png)

Произведем обновление инструментов установки а также установим
python2-dev.

    pip2 install --upgrade setuptools 

    sudo apt-get install python2-dev

![](images/clipboard-2742129718.png)

![](images/clipboard-817953072.png)

Производим установку зависимостей для дальнейшей установки самого
фреймворка.

    pip2 install pycrypto
    pip2 install distorm3

![](images/clipboard-3092615861.png)

Производим копирования git для последующей установки и переходим в папку
с скопированным репозиторием

    git clone https://github.com/volatilityfoundation/volatility
    cd volatility

![](images/clipboard-1022345405.png)

Запускаем установку фреймворка

    sudo python2 setup.py install

![](images/clipboard-596062415.png)

Проверяем его работоспособность

![](images/clipboard-1234823303.png)

## Анализ дампов

### Cridex

Получим первичную информацию о доставшемся дампе памяти

    vol.py -f cridex.vmem imageinfo

![](images/clipboard-3119893713.png)

Больше всего нас интересует название ОС с которой был произведен дамп –
WinXPSP2x86. В дальнейшем будем использовать любые плагины только под
данную платформу.

Для начала посмотрим какие процессы были запущены в момент взятия дампа.
Для наглядности отобразим деревом.

    vol.py -f cridex.vmem --profile=WinXPSP2x86 pstree

![](images/clipboard-1106381786.png)

Из интересного можно отметить только процесс с pid=1640, reader_sl.exe,
который был вызван pid=1484, explorer.exe. Остальные представляют на
первый взгляд базовые процессы системы Windows.

Попробуем посмотреть скрытые процессы в системе. Может есть не менее
подозрительные процессы.

    vol.py -f cridex.vmem --profile=WinXPSP2x86 psxview

![](images/clipboard-4056058647.png)

Однако ничего интересного в нашем дампе не удалось найти.

Посмотрим сетевой статус на нашем дампе, посмотрим TCP-соединения

    vol.py -f cridex.vmem --profile=WinXPSP2x86 connscan

![](images/clipboard-119830200.png)

Сокеты:

    vol.py -f cridex.vmem --profile=WinXPSP2x86 sockets

![](images/clipboard-4158249574.png)

Активные соединения(Не работает для нашего профиля)

    vol.py -f cridex.vmem --profile=WinXPSP2x86 netscan

![](images/clipboard-1023208647.png)

Наконец стоит получить данные исполняемый файл, чтобы удостовериться,
что он является вредоносным.

    vol.py -f cridex.vmem --profile=WinXPSP2x86 procdump -p 1640 --dump-dir volatility

Отправляем файл на проверку в VirusTotal и подтверждаем свои опасения.
Это вредоносная программа, притворяющаяся настоящим процессом Adobe
Acrobat.

![](images/clipboard-207534393.png)

![](images/clipboard-1329403455.png)

![](images/clipboard-4005301943.png)

![](images/clipboard-797310221.png)

![](images/clipboard-27504675.png)

### WannaCry

Получим первичную информацию о доставшемся дампе памяти.

    vol.py -f cridex.wcry.raw imageinfo

![](images/clipboard-1224309370.png)

Больше всего нас интересует название ОС с которой был произведен дамп –
WinXPSP2x86. В дальнейшем будем использовать любые плагины только под
данную платформу.

Для начала посмотрим какие процессы были запущены в момент взятия дампа.
Для наглядности отобразим деревом.

    vol.py -f wcry.raw --profile=WinXPSP2x86 pstree

![](images/clipboard-59651358.png)

Из интересного можно отметить только процесс с pid=740, WanaDecryptor,
который был вызван pid=1940, explorer.exe. Остальные представляют на
первый взгляд базовые процессы системы Windows.

Посмотрим сетевой статус на нашем дампе:

TCP-соединения:

    vol.py -f wcry.raw --profile=WinXPSP2x86 connscan

![](images/clipboard-2032212201.png)

Сокеты:

![](images/clipboard-3359977194.png)

    vol.py -f wcry.raw --profile=WinXPSP2x86 sockets

Соединений нет и чего-то интересного для нас тоже.

Наконец стоит получить данные исполняемый файл, чтобы удостовериться,
что он является вредоносным

    vol.py -f wcry.raw --profile=WinXPSP2x86 procdump -p 740 --dump-dir volatility

![](images/clipboard-64116687.png)

Отправляем файл на проверку в VirusTotal и подтверждаем свои опасения.
Это вредоносная программа, WannaCry — вредоносная программа, сетевой
червь и программа-вымогатель денежных средств, поражающая компьютеры под
управлением операционной системы Microsoft Windows.

![](images/clipboard-1850581324.png)

![](images/clipboard-2968915157.png)
