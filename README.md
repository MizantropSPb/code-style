# Verilog (SystemVerilog) Coding Style

## Отступы и пробелы
Отступ - 2 пробела. Табы (`\t`) запрещены.

Отступ нужны для обозначения "вложения".

Пример:
```systemverilog
if (a)
  b <= c;
```

```systemverilog
...
  input  wire iclk,
  input  wire irst,

  output reg  odata_dir,
  inout  wire [ 7: 0] iodata,
...
```

### Пробелы
Пробел должен ставится:
  - перед открывающей скобкой `(` и после закрывающей скобки `)` в конструкциях `if`, `for`, `always`, `function`, `task`, сложных логических условиях
  - перед и после элементов, требующих двух операндов ( `&`, `|`, `&&`, `||`, `=`, `<=`, `+`)
  - перед и после использованием переменных и констант. Исключение: перед `,` и `;` пробел не ставится.

Примеры:
```systemverilog
if (a > c)
```

```systemverilog
a = (b && (c > d));
```

```systemverilog
a = (b == c) ? y : z;
```

### Пустые строчки
Любые, логические связаные, порты необходимо отделять пустой строчкой.
Под `логические связаные порты` понимается совокупность сигналов, которые имеют одинаковое назначение или цель.

Пример:
Неправильно:
```systemverilog
...
  input  wire         iclk,
  input  wire         irst,
  input  wire [ 1: 0] imode,
  input  wire         imode_en,
  output reg  [ 7: 0] odata,
  output reg          odata_val
...
```

Здесь мы видим, что есть три разные сущности:
  - клок `iclk` и синхронный сброс `irst`
  - настройка какого-то режима работы `imode` и его разрешения `imode_en`
  - сигнал выходных данных `odata` и валидность этого сигнала `odata_valid`

Правильно:
```systemverilog
...
  input  wire         iclk,
  input  wire         irst,

  input  wire [ 1: 0] imode,
  input  wire         imode_en,

  output reg  [ 7: 0] odata,
  output reg          odata_val
...
``````

## Выравнивание
Выравнивание необходимо для облегчения чтения кода.
Выравнивание производится с помощью пробелов. Табы (`\t`) запрещены.

Необходимо выравнивать названия, размерность и комментарии по "логическим" колонкам.

### Пример #1
Неправильно:
```systemverilog
// BAD EXAMPLE
logic rd_stb; //buffer read strobe
logic [31:0] ram_data; //data from RAM block
logic [1:0]mode; //interface mode
```

Правильно:
```systemverilog
// GOOD EXAMPLE
logic         rd_stb;    // buffer read strobe
logic [31: 0] ram_data;  // data from RAM block
logic [ 1: 0] mode;      // interface mode
```

### Пример #2
Неправильно:
```systemverilog
// BAD EXAMPLE
input iclk,
input irst,
input [31:0] idata,
input idata_valid,
output logic [7:0] odata,
output odata_valid
```

Правильно:
``` systemverilog
// GOOD EXAMPLE
input                 iclk,
input                 irst,

input         [31: 0] idata,
input                 idata_valid,

output  logic [ 7: 0] odata,
output                odata_valid

```
Примечание:
  - по умолчанию логические колонки "разделяются" одним пробелом, однако, допускается делать 
    больше пробелов, если это улучшает читаемость (как в примере выше).

### Пример #3

Неправильно:
```systemverilog
// BAD EXAMPLE
assign next_len = len + 16'd8;
assign next_pkt_len = pkt_len + 16'd4;
```

Правильно:
```systemverilog
// GOOD EXAMPLE
assign next_len     = len     + 16'd8;
assign next_pkt_len = pkt_len + 16'd4;
```

### Пример #4
Неправильно:
```systemverilog
// BAD EXAMPLE
always_ff @(posedge iclk)
  begin
    pkt_data_d1 <= ipkt_data;
    pkt_empty_d1 <= ipkt_empty;
  end
```

Правильно:
```systemverilog
// GOOD EXAMPLE
always_ff @(posedge iclk)
  begin
    pkt_data_d1  <= ipkt_data;
    pkt_empty_d1 <= ipkt_empty;
  end
```

## Примеры использования конструкций и операторов

### Логические выражения
  - Необходимо использовать логическое И/ИЛИ/НЕ (`&&`/`||`/`!`) при описании каких-то логических выражений,
    а не их побитовые аналоги (`&`/`|`/`~`).
  - Любые сравнения, отрицания, и пр. берутся в скобки.

#### Пример #1
```systemverilog
logic data_ready;

assign data_ready = (pkt_word_cnt > 8'd5) && (!data_enable) && (pkt_len <= 16'd64);
```

#### Пример #2

```systemverilog
always_comb
  begin
    if ((data_enable == 1'h1) && (fifo_bytes_empty >= pkt_size))
      ...
  end
```

#### Пример #3

```systemverilog
assign start_stb = ((state == RED_S   ) && (next_state != IDLE_S )) ||
                   ((state == YELLOW_S) && (next_state != GREEN_S));
```

### `if-else`

```systemverilog
if (a > 5)
  d = 5;
else
  d = 15;
```

### `if-else` вместе с `begin/end`

```systemverilog
if (a > 5)
  begin
    c = 7;
    d = 5;
  end
else
  begin
    c = 3;
    d = 7;
  end
```

### Вложенный `if-else`

```systemverilog
if (a > 5)
  c = 7;
else
  if (a > 3)
    c = 4;
  else 
    c = 5;
```

### Тернарный оператор `?`

```systemverilog
assign y = (a > c) ? d : e;
```

Обращаю внимание, что условие взято в круглые скобки.

Для облегчения чтения (и проверки/написания) допускается (и во многих случаях рекомендуется) писать в две строки:
```systemverilog
assign y = (a > c) ? cnt_gt_zero :
                     cnt_le_zero;
```

### `case`

Каждый из вариантов описывается в своем `begin/end` блоке, отделяя
варианты друг от друга пустой строкой.

```systemverilog
case (code [ 1: 0])
  2'b00:
    begin
      state <= OR_S;
    end

  2'b01:
    begin
      state <= AND_S;
    end

  2'b10:
    begin
      state <= NOT_S;
    end

  2'b11:
    begin
      state <= XOR_S;
    end

  default:
    begin
      state <= AND_S;
    end
endcase
``` 

Если `case` простой и не подразумевает никаких вложенных конструкций в `begin/end` блоке, 
то допускается делать так (для уменьшения строк и текста):

```systemverilog
case (code[ 1: 0])
    2'b00: state <= OR_S;
    2'b01: state <= AND_S;
    2'b10: state <= NOT_S;
    2'b11: state <= XOR_S;
  default: state <= AND_S;
endcase
```

Обратите внимание на наличие выравнивания для облегчения чтения.

### `function` и `task`
Использование в большинстве случаев не привествуется. Разумная альтернатива: создание отдельного модуля, явное комбинаторное описание и т.п.

```systemverilog
function int calc_sum (input int a, int b);
  return a + b;
endfunction
```

```systemverilog
task receive_pkt (output packet_t pkt);
  ...
endtask
```

При большом количестве аргументов допустимо сводить к описанию в столбик:
```systemverilog
task some_magic( 
  input   int        a,
  input   bit [31:0] data,
  output  packet_t   pkt 
);
...
endtask
```
Однако, если у вас большое количество аргументов, у вас что-то пошло не так...

### Еще один пример 
```systemverilog
if (condition1)
  begin
    for (int i = 0; i < 10; i = i + 1)
      statement1;
  end
else
  begin
    if (condition2)
      statement2;
    else
      if (condition3)
        statement3;
      else
        statement4;
  end
```

## Комментарии
Комментарии на английском языке пишут только носители языка. Остальные довольствуются русским в кодировке win-
После знака комментария `//` ставится один пробел.
Если модуль реализует математический алгоритм то заголовок модуля должен содержать коментарий с математическим описанием. Кроме того, необходимо указать источника алгоритма (книга, статья, рисунок и т.п.). Допускается применение сокращенных названий. Расшифровка сокращений производится в файле дефайнов.
```systemverilog
...
// Алгоритм суммирования по скользящему окну
...
```
Желательно писать комментарии перед тем блоком, который вы хотите пояснить:
```systemverilog

// current packet word number
always_ff @(posedge iclk)
  if (irst)
    pkt_word <= 16'd0;
  else
    // reset counter at last valid packet word
    if (pkt_valid && pkt_eop)
      pkt_word <= 'd0;
    else
      if (pkt_valid)
        pkt_word <= pkt_word + 1'd1;
```

## Наименование переменных и модулей

- Стиль названия переменных, функций, тасков и модулей - `snake_case`, т.е. слова или префиксы
  разделяются `_`, и всё пишется маленькими буквами.

  Пример:
    - `pkt_anlz`
    - `pkt_ptr`
    - `super_task`
  
- Большими буквами (но так же через `_`) пишутся только константы, параметры, дефайны, состояния FSM

  Пример:
    - `parameter A_WIDTH = 9;`
    - `define CRC_LEN 32'h04`

- Имя переменной должно отражать ее назначение. 
  Следует избегать чрезмерно длинных и, особенно, чрезмерно коротких названий (`rd_addr` лучше чем `ra`).
  Пример подключения fifo (frd_data, frd_data_valid, frd_en, frd_empty, fwr_data, fwr_data_valid, fwr_en, fwr_full)

- Названия портов должны содержать префикс `i` для входных, `o` для выходных, `io` для 
  двунаправленных сигналов, без занака `_` после префикса

## Описание клоков
- Для описания клоков используется префикс `clk`.
- При объявлении портов модуля клоки описываются в самом начале. 
- Если необходимо указать, какой частоты предполагается этот клок, то необходимо использовать шаблон `clk_XmABC`, где:
  - `X` - значение частоты в МГц
  - `ABC` - оставшая часть (старшие три знака), если она не равна нулю.
    Примеры:
    - `2.048 МГц -> clk_2m048`
    - `156.25 МГц -> clk_156m25`
    - `250 МГц -> clk_250m`
- Если модуль содержит один клок то используется название `clk`.
- Работа с внешними клоками и генерация необходимых внутренних клоков производится в одном модуле расположенном на уровне top.
- Следует делать разделение на клоковые домены максимально близко уровню top.

  ```systemverilog
    .iclk    ( clk_100m      ),
    ....
  ```

- Если нужна какая-то параметризация в зависимости от клока, то необходимо сделать параметр в модуле:

  ```systemverilog
    parameter CLK_FREQ_HZ = 100_000_000;
  ```

  И затем его использовать в модуле.

- Все триггеры должны защелкиваться только по положительному фронту `posedge`. 
  Исключение: требования физических интерфейсов (например, DDR).

Категорически **запрещается**:
  - использовать сигнал клока для описания условий или создания комбинационной логики, типа:

    ```systemverilog
       assign my_super_clk = iclk && ( cnt == 8'd4 );
    ```

## Описание ресетов
Есть два типа ресета (сброса):
  - асинхронный `async_rst` (синоиним `rst`)
  - синхронный  `sync_rst`

В большинстве случаев мы используем синхронный сброс.
При объявлении портов модуля синхронный сброс описываются сразу после соотвествующего клока. При необходимости можно использовать шаблон, аналогично клоку.

```systemverilog
...

  input  wire iclk,
  input  wire irst,

...
```

Активным уровнем для сброса считаем `1`, т.е. когда `irst == 1'b1`, то схема находится в ресете. 

Описание триггера с синхронным ресетом:
```
always_ff @(posedge iclk)
  if (irst)
    cnt <= 8'h0;
  else
    ...
```

Описание триггера с асинхронным ресетом:
```
always_ff @(posedge iclk or posedge irst)
  if (irst)
    cnt <= 8'h0;
  else
    ...
```

Описание триггера с синхронным и асинхронным ресетом:
```
always_ff @(posedge iclk or posedge irst)
  if (irst)
    cnt <= 8'd0;
  else
    if (isrst)
      cnt <= 8'd0;
    else
    ...
```

  - **асинхронный сброс** используется в редких случаях.
  - В момент загрузки должна производиться иницализация всех тригеров.

```systemverilog
  ...
  output logic odata_req = 1'h0,
  ...
  reg  [ 7: 0] acc = 8'h00;
  ...
```

Категорически **запрещается**:
  - путать синхронный и асинхронный сброс как по заведению сигналов, так и по их наименованию
  - создавать модули без сбросов. Исключение: модули без триггеров/памяти (чистая комбинационная логика).
  - производить какие-то манипуляции с асинхронным сбросом, типа:
    ```systemverilog
      logic [7:0] cnt;
      assign my_rst = irst || ( cnt == 8'd5 );

      always_ff @(posedge iclk or posedge my_rst )
        if (my_rst)
          cnt <= 8'd0;
        else
          ...
    ```

## Описание конечного автомата (FSM)

Для описания конечного автомата используется следующая схема:

```systemverilog
enum reg [1:0] { IDLE_S,
                   RUN_S,
                   WAIT_S } state = IDLE_S;

always_ff @(posedge iclk)
  if (irst)
    state <= IDLE_S;
  else
    case( state )
      IDLE_S:
        begin
          if( ... )
            state <= RUN_S;
        end
      
      RUN_S:
        begin
          if( ... ) 
            next_state <= WAIT_S;
          else
            if( )
              state <= IDLE_S; 
        end

      WAIT_S:
        begin
          if( ... )
            state <= RUN_S;
        end

      default:
        begin
          state <= IDLE_S;
        end

    endcase

```

Пояснение:
  - `state` обозначает текущее состояние.  
  - `IDLE_S` - дефолтное состояние, поэтому его устанавливают во время cинхронного сброса и `default` в `case`-блоке.
     для удобства считаем, что это состояние должно идти первым в `enum` списке.

- Имена состояний FSM описываются большими буквами и должны содержать суффикс `_S` (`IDLE_S`, `TX_S`, `WAIT_S` и т.д.).

Категорически **запрещается**:
  - делать в одном модуле больше одного конечного автомата
  - производить какие-либо другие операции, кроме операции сравнения (`==`, `!=`) с переменными `state`, например:

    ```systemverilog
      assign b = ( state > RED_S );
      assign c = ( state + 3'd2 ) > ( IDLE_S );
    ```

## Размещение исходников

Исходники размещаются в файлах. 
Правило простое: один модуль, package, интерфейса - один файл. Название файла - название этого модуля. 

- Расширения:
  - Verilog: `.v`
  - SystemVerilog: `.sv`
  - Verilog Header: `.vh`
  - SystemVerilog Header: `.svh`
  - VHDL: `.vhd`

- Файлы содержащие только декларации package следует помечать окончанием `_pkg`: `func_pkg.sv`
- Файлы содержащие только константами, функции, таски, которые подгружаются в исходник при помощи include, следует именовать расширением .vh Пример
`defines.vh`.

## Описание логических блоков/элементов

Общее:
  - В одном блоке желательно описывать присваивание только в **одну** переменную.
  - Присваивания в переменную может происходить только в одном блоке.
  - Желательно блоки разделять пустой строчкой (сверху и снизу).
  
### Комбинационная логика
  - Для описания комбинационных схем можно использовать:
    - блок `always_comb`
    - непрерывное (continuous) присваивание `assign`
  - Разрешается использовать только блокирующие присваивание `=`.

Пример:
```systemverilog
always_comb
  begin
    a = b + d;
  end
```

Категорически **запрещается**:
  - описывать логику при "создании" переменной:
  ```systemverilog
  logic a = b && d;
  ```

  - описывать начальные значения для комбинационной логики:
  ```systemverilog
  logic [7:0] a = 8'd5;

  assign a = b + d;
  ```

### Триггеры
  - используется блок `always_ff`
  - разрешается использовать только неблокирующие присваивание (`<=`).
  - все тригеры инициализируются при объявлении

Пример: 
```systemverilog
logic [31:0] data_d1 = 32'h0;
logic [7:0]  cnt = 8'h01;

always_ff @(posedge iclk)
  begin
    data_d1 <= idata;
  end

always_ff @(posedge iclk)
  if (irst)
    cnt <= 8'h01;
  else
    if (cnt == 8'h92)
      cnt <= 8'h01;
    else
      cnt <= cnt + 8'h01;
```

### Защелки (латчи)

 - в существующих дизайнах не используются.
 - используется блок `always_latch`.
 - категорически запрещается использовать/создавать латчи в синхронных схемах.
 
Пример:
```systemverilog
always_latch
  begin
    if (en)
      data = idata;
  end
```

## Описание и объявление модулей

### Описание модуля

 - Каждый модуль описывается в отдельном файле.
 - Описание параметров и сигналов начинается с отступа (2 пробела).
 - Первым, в описании сигналов, должно идти описание клоков, синхронных ресетов и асинхронных рессетов.
 - При описании параметров и сигналов всё выравнивается по колонкам:
  - название сигналов 
  - квадратные скобки 
  - зарезервированные слова типа `parameter`, `input`, `output`
  - знак `=` в параметрах

- Сигналы не смешиваются в кучу. Те сигналы, которые формируют логический интерфейс, должны отделятся пустой строкой.

- Предпочтительно сначала описывать входные сигналы, потом выходные, однако, первочердным должен соблюдаться принцип интерфейсов, который описан выше.

Файл `some_module.sv`:

```systemverilog
module some_module #(
  parameter DATA_WIDTH = 32,
  parameter CNT_WIDTH  = 8
)
(
  input  wire  iclk,
  input  wire  irst,

  input  wire  [DATA_WIDTH-1:0] idata,
  input  wire  idata_val,

  output logic [ CNT_WIDTH-1: 0]  ocnt
);

// some code...

endmodule
```

### Инстанс модуля

- При подключении сигнала или переопределении параметра делается отступ (2 пробела).
- Cкобки, точки, запятые выравниваются.
- Для передачи параметров используется `#`, а не `defparam`.
- Пустые строчки, которые отделяют логические интерфейсы (или сигналы, которые должны быть вместе) расположены так же как и в описании модуля.
- Экземпляр модуля называется так же как и сам модуль с добавлением суффикса `_inst`.
  Если экземпляров одного модуля несколько, то их названия дополняются суффиксами `_inst0`, `_inst1` и т.д. Если название модуля слишком длинное, допускается инстансы (!) модулей типа: `main_engine` сокращать до `me`. Случай редчайший. Приведен для неумаления общности

```systemverilog
some_module #(
  .DATA_WIDTH ( 64 ),
  .CNT_WIDTH  ( 4  )
) some_module_inst
(
  .iclk       ( rx_clk      ),
  .irst       ( main_rst    ),

  .idata      ( rx_data     ),
  .idata_val  ( rx_data_val ),

  .ocnt       ( cnt         )
);  
```

## Описание переменных

Переменные в модуле "создаются" после описания сигналов модуля и до начала "работы" с этими сигналами. Тригеры инициализируются при описании

```systemverilog
  output logic oabc = 1'h0
);

logic [ 7: 0] cnt    = 8'h00;
logic         cnt_en = 1'h0;

logic [63: 0] pkt_data;
// ... some more signals

// work with signal starts here
```

- Связанные переменные, как и сигналы модуля необходимо логически отделять пустой строкой. 
- Допускается создавать сигналы "рядом" с тем местом, где они употребляются, например:
```systemverilog
logic [ 2: 0] vlan_mpls_cnt;
// more signals

// some code
always_comb
  begin
    if (vlan_mpls_cnt > 2)
      ...
  end

// calc vlan_mpls_cnt
logic [ 1: 0] vlan_cnt;
logic [ 1: 0] mpls_cnt;

assign vlan_cnt = vlan[ 0] + vlan[ 1] + vlan[ 2];
assign mpls_cnt = mpls[ 0] + mpls[ 1] + mpls[ 2];

assign vlan_mpls_cnt = vlan_cnt + mpls_cnt;
```

## Прочее

- Категорически запрещается реализовывать в устройствах сигналы с нулевым
  активным уровнем. Исключением могут быть стандартные интерфейсы, где такие
  сигналы изначально предусмотрены (sram interface, к примеру). Допускается
  наличие подобных сигналов на входах FPGA, но на верхнем уровне иерархии (в top-файле)
  они должны быть проинвертированы.

- При наличии двунаправленных шин, направления передачи должны развязываться на
  верхнем уровне иерархии (в top-файле).

- Необходимо в RTL описаниях использовать тип reg, wire, logic. Это позволит выявить ошибки, связанные с
  множественным назначением на этапе компиляции и ошибки, вызванные отсутствием
  инициализации. Рекомендуется использовать тип logic только в описаниях интерфейсов (порт с одним названием и разным направлением для разных модификаций интрефейса)

- Настоятельно рекомендуется защелкивать в триггеры выходные сигналы модулей.
  Это повысит максимальную скорость работы устройства.

- Все критичные к быстродействию узлы по возможности следует размещать в
  отдельном модуле. Это позволит оптимизировать его независимо от прочих узлов.

- Все макроподстановки (`define) должны определяться в отдельном файле. Допустимо определение в
  самом модуле.

- Категорически запрещается "пробрасывать" сигнал сквозь модуль если значение сигнала, внутри модуля, не изменяется.

- Настоятельно рекомендуется вводить в модуль только те сигналы которые реально используются

- По возможности, модули должны быть параметризованы. Не надо пытаться параметризовывать "все" на все случаи жизни. Это делат код менее читабельным и понятным. Лучше иметь два ясных и кратких модуля чем один путаный
