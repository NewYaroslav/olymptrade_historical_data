# Исторические данные брокера OlympTrade

### Описание
Данный репозиторий содержит исторические данные котировок и процентов выплат брокера OlympTrade

* Файлы обновляются в репозитории каждый торговый день в 00:00 GMT
* Один файл соответствует одному дню, название файла формируется из даты
* Файлы содержат информацию о следующих валютных парах или индексов (см. *parameters.json*)

### Структура файлов исторических данных по процентам выплат брокера

Папка proposal_data содержит файлы с процентами выплат для нескольких валютных пар.
Однин сэмпл файла соответствует одной секунде. 
Один файл соответствует одному дню, название файла формируется из даты.
Проценты выплат хранятся в двоичном формате для уменьшения размера файлов. В начале файла есть заголовок-строка, оканчивающееся символом *\n* и содержащая в формате JSON информацию.
Пример заголовка:

```json
{"sample_len":84,"symbols":["EURRUB","CHFJPY","HG","GBPCHF","GBPAUD","USDRUB","AUDUSD","TSLA","ETCUSD","BTGUSD","CADCHF","FCE","TF","DASHUSD","EURCAD","ES","XAGUSD","LTCUSD","CADJPY","BMW","NZDJPY","USDTRY","AUDJPY","Z","USDNOK","EURUSD","LTCBTC","ZECUSD","USDCAD","XMRUSD","MSFT","NINTENDO_JP","BA","USDCHF","NZDUSD","FB","USDJPY","AUDCHF","EURCHF","V","ETHUSD","NKD","GBPNZD","GBPUSD","FDAX","_BRN","AUDNZD","HSI","MCD","SBUX","USDMXN","EURJPY","EURAUD","NZDCHF","Bitcoin","KO","XAUUSD","IBM","NG","AAPL","FESX","EURGBP","ETHBTC","BCHUSD","XRPUSD","GBPCAD","NQ","USDCLP","PL","EURNZD","GBPJPY","NZDCAD","GOOGL","YM","USDSGD","AUDCAD"]}
```

* *sample_len* - длина одного сэмпла соответствующего одной секунде дня, в байтах
* *symbols* - массив валютных пар

После заголовка идет массив бинарных данных, состоящий из сэмплов. Каждый сэмпл содержит значения процентов выплат для всех валютных пар в порядке их чередования, указанном в заголовке, а также содержит *timestamp* длиной 8 байт в конце сэмпла.
Для сокращения данных проценты выплат хранятся в виде одного байта данных. Чтобы получить значение типа float или double, необходимо разделить считанное из файла значение на 100. Пример:

```C++
unsigned char raw_proposal;
// читаем данные в raw_proposal
file.read(reinterpret_cast<char *>(&raw_proposal), sizeof (raw_proposal));

//...

// восстанавливаем данные
float proposal = (float)raw_proposal / 100.0f;
```

Пример кода, который читает файл целиком:

```C++
	// загружаем настройки
	std::ifstream fin(file_name);
	std::string _s;
	std::getline(fin, _s);
	// парсим JSON строку
	json j_pp = json::parse(_s);
	// запоминаем смещение в файле (нужно так как бинарные данные расположены после заголовка)
	unsigned long start_pos = _s.size();
	fin.close();

	unsigned long sample_size = j_pp["sample_len"]; // длина одного сэмпла
	// читаем бинарные данные
	std::ifstream i(file_name, std::ios_base::binary);

	i.seekg (0, std::ios::end); // смещаемся в конец файла
	unsigned long data_size = i.tellg(); // получаем размер файла
	// 2- это магическое число появляется из-за символов переноса строки
	data_size = data_size - start_pos - 2; // получаем размер файла без заголовка
	i.seekg (start_pos + 2, std::ios::beg); // смещаемся в начало бинарных данных
	i.clear(); // очищаем флаги, навсякий случай

	// проверяем кратность размера данных размеру сэмпла
	if(data_size % sample_size != 0) {
		// если кратности нет, то дела плохи
		return false;
	}
	// получаем количество эсмплов
	unsigned long sample_num = data_size / sample_size;
	// получаем количество валютных пар
	unsigned long symbols_size = j_pp["symbols"].size();

	for(unsigned long n = 0; n < sample_num; ++n) { // читаем все сэмплы
		for(unsigned long s = 0; s < symbols_size; ++s) { // в каждом сэмпле читаем все валютные пары
			unsigned char temp = 0;
			i.read(reinterpret_cast<char *>(&temp), sizeof (temp));
			double proposal_data = (double)temp / 100.0d;
			/* делаем что хотим с данными процентов выплат (proposal_data)
			 * данные представлены от 0.0 (0%) до 1.0 (100%)
			 */
		}
		unsigned long long timestamp = 0;
		// читаем временную метку
		i.read(reinterpret_cast<char *>(&timestamp), sizeof (timestamp));
		// в переменной timestamp находится время, когда данные процентов выплат были актуальны
	}
	i.close();
	// конец
```
