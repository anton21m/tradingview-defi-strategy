import os, requests


# Устанавливаем прокси для HTTP и HTTPS
```
os.environ['HTTP_PROXY'] = 'http://localhost:8080'
os.environ['HTTPS_PROXY'] = 'http://localhost:8080'

response = requests.get('http://example.com')
print(response.text)
```

# Hint Функций
`self.transport.fetch_candles_by_pair_ids`
Загрузка OHLCV данных без примесей (данные в формате parquet)
```
{"ts":1639670400,"o":86.40313072279048,"h":104.5949100674392,"l":71.51368925215993,"c":94.85657706579647,"v":440.80824199999995,"xr":1.0,"b":6,"s":10,"tc":16,"bv":207.982826,"sv":232.82541599999996,"p":2601348,"sb":8323378,"eb":8326874}
```

Все это мапится `.venv/src/trade-executor/deps/trading-strategy/tradingstrategy/transport/jsonl.py`
```
CANDLE_MAPPINGS = {
    "ci": None,  # chain_id discarded
    "ei": None,  # exchange_id discarded
    "p": "pair_id",
    "ts": "timestamp",
    "o": "open",
    "h": "high",
    "l": "low",
    "c": "close",
    "xr": "exchange_rate",
    "b": "buys",
    "s": "sells",
    "bv": "buy_volume",
    "sv": "sell_volume",
    "sb": "start_block",
    "eb": "end_block",
    "tc": None,  # Currently not available for Uni v3 exchanges
    "v": "volume",  # TODO deprecate
}
```
Актуальный сейчас
```
{'ts': 1639670400, 'o': 86.40313072279048, 'h': 104.5949100674392, 'l': 71.51368925215993, 'c': 94.85657706579647, 'v': 440.80824199999995, 'xr': 1.0, 'b': 6, 's': 10, 'tc': 16, 'bv': 207.982826, 'sv': 232.82541599999996, 'p': 2601348, 'sb': 8323378, 'eb' : 8326874}
```
конечная форма в DataFrame
```
values: array(['timestamp', 'open', 'high', 'low', 'close', 'volume','exchange_rate', 'buys', 'sells', 'buy_volume', 'sell_volume','pair_id', 'start_block', 'end_block', 'avg'], dtype=object)
```

## формат json для получения свеч
Через API: method /candles-jsonl 

для образца pair_ids=9261 (https://tradingstrategy.ai/api/explorer/#/Trading%20pair/web_candles_jsonl)

Формат json для свеч - JSONL содержит такие данные:
```
{"ts":1641081600,"o":3757.9228102550887,"h":3766.728760405581,"l":3752.4468305293567,"c":3765.383031825666,"v":781734.9429940002,"xr":1.0,"b":32,"s":25,"tc":57,"bv":463456.8956050001,"sv":318278.047389,"p":9261,"sb":13922672,"eb":13922932}
```
> Расшисшифровка: OHLCV, xr (сплит, раскол, 1.0 не было раскола), b, s (количество покупателей/продавцов), tc (общее количество сделок, b + s), bv, sv (сумма всех покупок/продаж), p (id пары), sb, eb (начальный/конечный блок в блокчейне, который был при прохождении этой свечи)


# Патчи и заглушки для быстрого внедрения pinescript кода

PATH
```
отключаем кеш
/home/anton/Documents/my-test/Trading/new-trading-view-strategy/tradingview-defi-strategy/.venv/src/trade-executor/deps/trading-strategy/tradingstrategy/transport/cache.py # 700 
```



```
.venv/src/trade-executor/deps/trading-strategy/tradingstrategy/transport/jsonl.py # 126 подменяем данные на pinescript sample
# sample tradingview data 
    import io
    import json
    import jsonlines
    import io
    import random
    from datetime import datetime, timedelta

    # Устанавливаем начальную временную метку и количество записей
    start_datetime = datetime(2022, 3, 1)  # Начальная временная метка (пример)
    end_datetime = datetime(2022, 10, 1)   # Конечная дата

    time_step = timedelta(hours=4)  # Шаг в 4 часа
    num_records = int((end_datetime - start_datetime) / time_step) + 1  # Количество записей

    # Формируем JSONL с использованием шаблона
    jsonl_output = io.StringIO()
    with jsonlines.Writer(jsonl_output) as writer:
        for i in range(num_records):
            # Вычисляем текущую временную метку
            current_timestamp = int((start_datetime + time_step * i).timestamp())
            
            # Генерируем случайные значения для o, h, l, c, v
            open_price = random.uniform(70, 100)
            high_price = random.uniform(open_price, open_price + 20)  # h должен быть больше o
            low_price = random.uniform(open_price - 20, open_price)    # l должен быть меньше o
            close_price = random.uniform(low_price, high_price)         # c должен быть между l и h
            volume = random.uniform(100, 500)

            # Создаем запись в формате JSON
            record = {
                "ts": current_timestamp,
                "o": open_price,
                "h": high_price,
                "l": low_price,
                "c": close_price,
                "v": volume,
                "xr": 0,
                "b": 0,
                "s": 0,
                "tc": 0,
                "bv": 0,
                "sv": 0,
                "p": 2601348,
                "sb": 0,
                "eb": 0
            }
            
            # Записываем запись в JSONL
            writer.write(record)

    # Выводим результат в формате JSONL
    jsonl_output.seek(0)  # Возвращаемся в начало потока для чтения
    # Исправленный формат resp2: объединяем JSON-объекты в список
    resp = jsonl_output.read()

    reader = jsonlines.Reader(io.StringIO(resp))

    # resp = session.get(api_url, params=params, stream=True)

    print("Debug")


    # reader = jsonlines.Reader(resp.raw)

    candle_data = defaultdict(list)
```