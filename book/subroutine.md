# Подпрограммы

Подпрограмма - это именованный фрагмент кода, содержащий описание определенного набора действий. К подпрограммам относятся замыкания (процедуры и функции), методы и сопрограммы.

## Замыкания (Proc)

Замыкание (англ. closure) в программировании - это процедура или функция, в теле которой присутствуют ссылки на переменные, объявленные вне тела этой функции и не в качестве её параметров (а в окружающем коде). Эти переменные продолжают существовать во время выполнения функции, даже если во внешнем коде их уже не существует.

Замыкания создаются заново каждый раз в момент выполнения.

Замыкания, так же как и объекты служат для инкапсуляции функциональности и данных.

Замыкания могут быть переданы методам как обычные блоки. Для этого перед аргументом используется амперсанд.

Псевдопеременная self сохраняет значение, сущестовавшее в момент создания замыкания.

~~~~ ruby
  class FirstKlass
    def self.first; 1; end

    def first; 2; end

    @@first = proc { first }

    def klass; @@first.call; end

    def instance; instance_exec &@@first; end
  end

  FirstKlass.new.klass # -> 1
  FirstKlass.new.instance # -> 2
~~~~~

Замыкания разделяются на процедуры (ведущие себя как блоки, но в действительности также относящиеся к функциям) и лямбда-функции (ведущие себя как методы).

### Процедуры

Инструкция return в теле процедуры приводит к завершению выполнения метода, в теле которого объект был создан. Если выполнение метода уже завершено, то вызывается исключение `LocalJumpError`.

Процедуры принимают аргументы также как и обычные блоки.

`::new { |*params| } # -> proc`

Используется для создания нового замыкания.

~~~~~ ruby
  def proc_from
    Proc.new
  end

  proc = proc_from { "hello" }
  proc.call # -> "hello"
~~~~~

`.proc { |*params| } # -> proc [PRIVATE: Kernel]`

Версия предыдущего метода из модуля Kernel.

~~~~~ ruby
  def foo
    f = Proc.new { return "return from foo from inside proc" }
    f.call # после вызова функции замыкания f осуществляется выход из foo
    # результатом работы функции foo является результат работы f замыкания
    return "return from foo"
  end

  puts foo # печатает "return from foo from inside proc"
~~~~~

### Лямбда-функции

Инструкция return в теле лямбда-функций приводит к завершению выполнения функции.

Лямбда-функции принимают аргументы также как и обычные методы.

Для создания лямбда функций существует специальный синтаксис. Круглые скобки не обязательны. Параметры могут иметь значения по умолчанию.

`->(*params) { } # -> lambda`

`.lambda { |*params| } # -> lambda`

Используется для создания объекта.

~~~~~ ruby
  def bar
    f = lambda { return "return from lambda" }
    f.call # после вызова функции замыкания f продолжается выполнение bar
    return "return from bar"
  end

  puts bar # печатает "return from bar"
~~~~~

### Использование замыканий

#### Приведение типов

`.to_proc # -> self`

`.to_s # -> string`

Информация об объекте.  
`Proc.new { }.to_s # -> "#<Proc:0x8850f18@(irb):31>"`

#### Операторы

`.==(proc)`  
Синонимы: `eql?`

Проверка на равенство. Два замыкания равны, если относятся к копиям одного и того же объекта.  
`proc {} == proc {} # -> true`

`.===(arg) # -> object`

Используется для выполнения замыкания.  
`proc { |a| a } === 1 # -> 1`

#### Выполнение замыкания

`.call(*arg) # -> object`  
Синонимы: `yield, .(*arg), [*arg]`

Используется для выполнения замыкания.  
`-> x {x**2}.(5) # -> 25`

\declare{.curry( count = nil )*arg}{ \# -> proc}

Используется для подготовки замыкания. Замыкание будет выполнено, когда передается достаточное количество аргументов. В другом случае замыкание сохраняет информацию о переданных аргументах.

Дополнительный аргумент ограничивает количество передаваемых аргументов (остальным параметрам присваивается nil).

~~~~~ ruby
  b = proc { | x, y, z | x + y + z }
  b.curry[1][2][3] # -> 6
  b.curry[1, 2][3, 4] # -> 6
  b.curry[1][2][3][4][5] # -> 0
  b.curry(5) [ 1, 2 ][ 3, 4][5] # -> 6
  b.curry(1) [1] # -> type_error!

  b = lambda { | x, y, z | x + y + z }
  b.curry[1][2][3] # -> 6
  b.curry[1, 2][3, 4] # -> argument_error!
  b.curry(5) # -> argument_error!
  b.curry(1) # -> argument_error!
~~~~~

#### Остальное

`.arity # -> integer`

Количество принимаемых аргументов. Для произвольного количества возвращается отрицательное число. Его инверсия с помощью оператора `~` в результате возвращает количество обязательных аргументов.

~~~~~ ruby
  proc {}.arity # -> 0
  proc { || }.arity # -> 0
  proc { |a| }.arity # -> 1
  proc { |a, b| }.arity # -> 2
  proc { |a, b, c| }.arity # -> 3
  proc { |*a| }.arity # -> -1
  proc { |a, *b| }.arity # -> -2
  proc { | a, *b, c | }.arity # -> -3
~~~~~

`.parameters # -> array`

Информация о параметрах.

~~~~~ ruby
  proc = lambda { | x, y = 42, *other | }
  proc.parameters
  # -> [ [:req, :x],  [:opt, :y], [:rest, :other] ]
~~~~~

`.binding # -> binding`

Используется для получения состояния выполнения программы для замыкания.

`.lambda? # -> bool`

Проверка относится ли объект к лямбда-функциям.  
`proc {}.lambda? # -> false`

`.source_location # -> array`

Местоположение создания замыкания в виде `[filename, line]`. Для замыканий, создаваемых не на Ruby, возвращается nil.  
`proc {}.source_location # -> ["(irb)", 19]`

`.hash # -> integer`

Цифровой код объекта.  
`proc {}.hash # -> -259341767`

#### Мемоизация

Мемоизация - это хранение ранее вычисленного результата с помощью замыканий и вложенных подпрограмм.

~~~~~ ruby
  closure = proc { counter = 0; proc {counter += 1} }.call
  closure.call # -> 1
  closure.call # -> 2
  closure.call # -> 3
~~~~~

## Методы

Подпрограммы могут быть созданы на основе уже существующих методов с помощью классов Method и UnboundMethod.

### Method

Экземпляры класса сохраняют информацию о методе вместе с объектом, для которого он вызывается.

`.method(name) # -> method`

Используется для сохранения информации о переданном методе объекта. Отсутствие метода считается исключением.  
`12.method(?+) # -> #<Method: Fixnum#+>`

`.public_method(name) # -> method`

Версия предыдущего метода для поиска только общих методов.  
`12.public_method(?+) # -> #<Method: Fixnum#+>`

#### Операторы

`.==(method)`  
Синонимы: `eql?`

Проверка на равенство. Объекты равны, если связаны с одним и тем же объектом и содержат информацию об одном и том же методе.  
`12.method(?+) == 13.method(?+) # -> false`

#### Приведение типов

`.to_proc # -> lambda`

Используется для создания лямбда-функции на основе метода.  
`12.method(?+).to_proc # -> #<Proc:0x88cdd60 (lambda)>`

`.to_s # -> string`

Информация об объекте.  
`12.method(?+).to_s # -> "#<Method: Fixnum#+>"`

`.unbind # -> umethod`

Испольуется для удаления информации об объекте, для которого вызывается метод.  
`12.method(?+).unbind -> #<UnboundMethod: Fixnum#+>`

#### Вызов метода

`.call(*arg) # -> object`

Используется для вызова метода с переданными аргументами.  
`12.method(?+).call 3 # -> 15`

#### Остальное

`.arity # -> integer`

Количество принимаемых аргументов. Для произвольного количества возвращается отрицательное число. Его инверсия с помощью оператора `~` в результате возвращает количество обязательных аргументов. Для методов, определенных без помощи Ruby возвращается -1.  
`12.method(?+).arity # -> 1`

`.name # -> symbol`

Идентификатор метода.  
`12.method(?+).name # -> :+`

`.owner # -> module`

Модуль, в котором объявлен метод.  
`12.method(?+).owner # -> Fixnum`

`.parameters # -> array`

Массив параметров метода.  
`12.method(?+).parameters # -> [ [:req] ]`

`.receiver # -> object`

Объект, для которого метод вызывается.
`12.method(?+).receiver # -> 12`

`.source_location # -> array`

Местоположение объявления метода в виде массива `[filename, line]`. Для методов, определенных без помощи Ruby, возвращается nil.  
`12.method(?+).source_location # -> nil`

`.hash # -> integer`

Цифровой код объекта.  
`12.method(?+).hash # -> -347045594`

### UnboundMethod

Экземпляры класса сохраняют информацию только о методе.

`.instance_method(name) # -> umethod [Module]`

Используется для хранения информации о переданном методе. Отсутствие метода считается ошибкой.  
`Math.instance_method :sqrt # -> #<UnboundMethod: Math#sqrt>`

`.public_instance_method(name) # -> umethod [Module]`

Версия предыдущего метода для поиска только общих методов.  
`Math.public_instance_method :sqrt # -> error`

#### Операторы

`.==(umethod)`  
Синонимы: `eql?`

Проверка на равенство. Объекты равны, если содержат информацию об одном и том же методе.  
`Math.instance_method(:sin) == Math.instance_method(:sin) # -> true`

#### Приведение типов

`.to_s # -> string`
Синонимы: `inspect`

Информация об объекте.  
`Math.instance_method(:sqrt).to_s # -> "#<UnboundMethod: Math#sqrt>"`

`.bind(object) # -> method`

Используется для добавления информации об объекте, для которого метод вызывается. Если такая информация уже существовала, то объекты должны принадлежать к одному классу.

~~~~~ ruby
  12.method(?+).unbind.bind 1 # -> #<Method: Fixnum#+>
  12.method(?+).unbind.bind 1.0 # -> error!
~~~~~

#### Остальное

`.arity # -> integer`

Количество принимаемых аргументов. Для произвольного количества возвращается отрицательное число. Его инверсия с помощью оператора `~` в результате возвращает количество обязательных аргументов. Для методов, определенных без помощи Ruby возвращается -1.  
`Math.instance_method(:sqrt).arity # -> 1`

`.name # -> symbol`

Идентификатор метода.  
`Math.instance_method(:sqrt).name # -> :sqrt`

`.owner # -> module`

Модуль, в котором объявлен метод.  
`Math.instance_method(:sqrt).owner # -> Math`

`.parameters # -> array`

Массив параметров метода.  
`Math.instance_method(:sqrt).parameters # -> [ [:req] ]`

`.source_location # -> array`

Местоположение объявления метода в виде массива `[filename, line]`. Для методов, определенных без помощи Ruby, возвращается nil.  
`Math.instance_method(:sqrt).source_location # -> nil`

`.hash # -> integer`

Цифровой код объекта.  
`Math.instance_method(:sqrt).hash # -> 563385534`

## Сопрограммы (Fiber)
[](fiber)

Сопрограмма - это фрагмент кода, поддерживающий несколько входных точек и остановку или продолжение выполнения с сохранением состояния выполнения. Сопрограмму также можно рассматривать как поток выполнения, прерываемый специальной инструкцией.

Сопрограммы также могут использоваться для реализации многопоточности на уровне программы (в действительности код выполняется в единственном потоке выполнения). Такие потоки также называют "green thread". Использование сопрограмм позволяет уменьшить накладные расходы на переключение и обмен данными, так как не требует взаимодействия с ядром ОС.

Проблема при использовании сопрограмм в том, что выполнение системного вызова будет блокировать процесс выполнения программы. Сопрограммы должны использовать специальные методы ввода/вывода, не блокирующие процесс выполнения. Также стоит заметить что управление переключением сопрограмм выполняется вручную и требует дополнительных затрат при разработке.

Для управления сопрограммами создаются контрольные точки с помощью метода `::yield` и осуществляется последовательный переход между ними с помощью метода `.resume`.

`::new { |*params| } # -> fiber`

Используется для создания сопрограммы. Блок при этом не выполняется.

`::yield(*temp_result) # -> object`

Используется для создания контрольной точки. Переданные аргументы возвращаются в результате вызова `.resume`.

Возвращаются объекты, переданные при вызове метода `.resume` или полученные в результате выполнения последнего выражения в теле сопрограммы.

`.resume(*args) # -> temp_result`

Используется для выполнения блока до следующей контрольной точки.

Если метод был вызван впервые, то переданные аргументы отправляются в сопрограмму. В другом случае они возвращаются в результате вызова метода `::yield` в теле блока.

Если сопрограмма уже выполнена, то вызывается исключение `FiberError`.