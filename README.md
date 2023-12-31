# Использование фильтра Калмана для обработки полученных данных с гироскопа и акселерометра.
## Общие сведения
### Цель проекта
Улучшение точности и стабильности данных с гироскопа и акселерометра для более эффективного использования в навигации.

### Команда исполнителей
- Земцовская Ольга;
- Зиуатдинов Исламбек;
- Храпов Олег;
- Чухрай Екатерина.

## Технические требования
### Требования к функциональным характеристикам

Возможность работы в реальном времени.
Устойчивость к шумам и возможность компенсации ошибок измерений.
Возможность интеграции с другими датчиками (например, магнитометром).
Получение данных с устройства для фильтрации.
Простота использования и настройки фильтра Калмана.
Работоспособность при температуре от -40°C до +85°C.
### Требования к надежности
Стабильность работы при изменении температуры и влажности.
Устройство должно быть надежным и прочным, способным выдерживать длительное использование.
Возможность замены неисправных элементов без необходимости замены всего устройства.
### Условия эксплуатации
Данные по условиям эксплуатации взяты из технических характеристик устройства MPU-6050.
Воздействие условий с абсолютными максимальными номинальными значениями в течение длительного времени может повлиять на надежность устройства.

|Параметр|Оценка|
|--------|-------|
|Напряжение питания, VDD|-0.5V to +6V|
|Уровень входного напряжения VLOGIC (MPU-6050)|-0.5V to VDD + 0.5V|
|REGOUT|-0.5V to 2V|
|Input Voltage Level (CLKIN, AUX_DA, AD0, FSYNC, INT, SCL, SDA)|-0.5V to VDD + 0.5V|
|CPOUT (2.5V ≤ VDD ≤ 3.6V )|-0.5V to 30V|
|Ускорение (любая ось, без питания)|10,000g for 0.2ms|
|Диапазон рабочих температур|-40°C to +105°C|
|Диапазон температур хранения|-40°C to +125°C|
|Защита от электростатического разряда (ESD)|2kV (HBM);  250V (MM)|
|Защита от короткого замыкания|JEDEC Class II (2),125°C ±100mA|


### Требования к составу и параметрам технических средств
Микроконтроллер STM32 <br/>
Микросхема MPU-6050 <br/>

### Требования к информационной и программной совместимости
MPU-6050 подключается к микроконтроллеру через интерфейс I2C. <br/>
Напряжение питания 3.3 В. <br/>
Программное обеспечение, которое конвертирует данные и обеспечивает калибровку. <br/>

## Проектирование системы
### Описание 
В работе используется MPU-6050 – это инерционный измерительный модуль, который используется для измерения ускорения и угловой скорости в трех осях. Модуль обычно используется в дронах для определения ориентации и стабилизации полета.

Для сбора данных с MPU-6050 в дроне, модуль должен быть подключен к микроконтроллеру, который будет обрабатывать данные. MPU-6050 имеет интерфейс I2C, который обеспечивает связь с микроконтроллером. Чтобы начать сбор данных, микроконтроллер должен отправить запрос на MPU-6050 для получения данных, а модуль будет отправлять данные об ускорении и угловой скорости в трех осях. Далее, полученные данные будут обработаны фильтром Калмана, который убирает шумы и лишнюю информацию.

На рисунке 1 и 2 изображены сети процессов Кана. Рисунок 1 отражает взаимодействие верхнеуровневых процессов от получения команды из интерфейса пользователя до получения обработанных данных. На рисунке 2 изображены процессы происходящие в блоке “Обработка команды системой управления” (рис. 1).

Диаграмма последовательности на рисунке 3 описывает, как сигнал от пользователя обрабатывается системой управления дрона и передает его на контроллер, который находится на борту дрона.

Передача сигнала к MPU-6050 и получение от него данных происходит через интерфейс I2C. Для начала, контроллер должен отправить адрес MPU-6050 по шине I2C. Затем контроллер отправляет запрос на чтение данных. MPU-6050 отвечает на запрос, отправляя данные об ускорении и угловой скорости в трех осях. 

Для получения этих данных, контроллер должен считать данные из регистров MPU-6050. Регистры хранят данные в двухбайтовом формате, поэтому микроконтроллер должен объединить два байта для получения значения. После получения данных, контроллер передает их обратно для фильтрации шума и вибраций. 

<br/>
<img src = "https://github.com/ZemtsH/STM32-Laba/blob/main/img/n%20(2).png?raw=true"> 
Рисунок 1 – Сеть процессов Кана. Верхний уровень <br/>
<br/>
<img src = "https://github.com/ZemtsH/STM32-Laba/blob/main/img/n%20(3).png?raw=true"> 
Рисунок 2 – Сеть процессов Кана. Процессы блока “Обработка команды системой управления” <br/>
<br/>
<img src = "https://github.com/ZemtsH/STM32-Laba/blob/main/img/n%20(4).png?raw=true"> 
Рисунок 3 – Диаграмма последовательности <br/>


## Разработка программного обеспечения
### Краткое описание алгоритма работы
В работе было реализовано получение данных с гироскопа и акселерометра, а также дальнейшая фильтрация данных с помощью алгоритма Калмана.

Алгоритм Калмана используется для оценки состояния системы на основе последовательных измерений с учетом шума и ошибок в измерениях. В данном коде алгоритм Калмана используется для фильтрации данных, получаемых с акселерометра и гироскопа MPU-6050. 

Для этого создаются два экземпляра структуры Kalman_t - KalmanX и KalmanY, которые содержат параметры алгоритма: Q_angle, Q_bias и R_measure. Q_angle и Q_bias отвечают за ожидаемую ошибку в измерениях угла и скорости, а R_measure - за шум в измерениях. 

Затем данные с акселерометра и гироскопа считываются в структуру MPU6050_t и преобразуются в значения ускорения и скорости в градусах на секунду. После этого, значения используются в алгоритме Калмана для фильтрации шума и ошибок в измерениях. 

Алгоритм Калмана для обработки данных с гироскопа и акселерометра состоит из следующих шагов:

- Определение матрицы состояния системы, которая описывает зависимости между входными данными (измерениями гироскопа и акселерометра) и выходными данными (углами наклона и скоростью изменения угла наклона).
- Определение матрицы ковариации ошибок измерений, которая описывает неопределенность в измерениях гироскопа и акселерометра.
- Определение матрицы ковариации ошибок модели, которая описывает неопределенность в модели системы.
- Определение матрицы ковариации состояния, которая описывает неопределенность в текущем состоянии системы.
- Обновление состояния системы на основе новых измерений гироскопа и акселерометра.
- Обновление матрицы ковариации состояния на основе новых измерений гироскопа и акселерометра.
- Предсказание состояния системы на следующий шаг времени на основе текущего состояния и модели системы.
- Предсказание матрицы ковариации состояния на следующий шаг времени на основе текущей матрицы ковариации состояния и матрицы ковариации ошибок модели.
- Повторение шагов 5-8 для каждого нового измерения гироскопа и акселерометра.
- Использование полученных углов наклона и скорости изменения угла наклона для управления дроном.

Ковариация - это мера степени линейной зависимости между двумя случайными величинами. Она описывает, насколько сильно две случайные величины изменяются вместе. Большое значение ковариации говорит о том, что две случайные величины сильно связаны друг с другом, а маленькое значение ковариации говорит о слабой связи между ними. Ковариация может быть положительной, отрицательной или равной нулю. Положительная ковариация означает, что две случайные величины изменяются в одном направлении, отрицательная - в противоположном направлении, а нулевая - что они независимы друг от друга

### Визуализация данных
На рисунке 4 представлены три графика, которые показывают данные по трем осям – X, Y, Z полученные от гироскопа, акселерометра и обработанные с помощью алгоритма Калмана. <br/> 
<img src = "https://github.com/ZemtsH/STM32-Laba/blob/main/img/n%20(1).png?raw=true" width=1020> <br/>
## Выводы
Использования MPU-6050 с фильтром Калмана заключается в улучшении точности и стабильности получаемых данных с гироскопа и акселерометра, что позволит более эффективно использовать их в различных приложениях, таких как навигация, управление дронами, робототехника и другие.


## Источники
<p>Lib: https://github.com/leech001/MPU6050.git</p>
<p>Datasheet: https://roboparts.ru/upload/iblock/d38/d38745334b67536e8a9956d370b45f18.pdf</p>
