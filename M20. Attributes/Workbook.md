***In progress***

# Рефлексия, атрибуты и динамическое программирование

***Materials from book by Mark Michaelis; Eric Lippert. Essential C# 7.0. Publishing Hous "Boston : Addison-Wesley : Pearson Education", 2018***

[Source code]()

<a name="content"></a>
 + [Отражение (Рефлексия)](#1)
 + [Доступ к метаданным с использованием System.Type](#2)
 + [](#3)

> Атрибуты - это средство вставки дополнительных метаданных в сборку и связывания метаданных с конструкцией программирования, такой как класс, метод или свойство. В этой главе рассматриваются детали, окружающие атрибуты, встроенные в структуру, и описываются способы определения пользовательских атрибутов. Чтобы воспользоваться пользовательскими атрибутами, необходимо их идентифицировать. Это осуществляется через отражение. Эта глава начинается с анализа рефлексии, в том числе того, как ее можно использовать ее для динамического связывания во время выполнения на основе вызова участника по имени (или метаданным) во время компиляции. Это часто выполняется в таких инструментах, как генератор кода. Кроме того, отражение используется во время выполнения, когда цель вызова неизвестна.

##  <a name="1"></a> Отражение (Рефлексия)

Используя отражение, можно сделать следующее.
  - Получить доступ к метаданным для типов внутри сборки, например, полное имя типа, имена членов и любые атрибуты, деклрирующие конструкцию.
  - Динамически вызывать элементы типа во время выполнения с использованием метаданных, не привязываваясь к компиляции.

> Отражение - это процесс изучения метаданных внутри сборки. Традиционно, когда код компилируется до машинного языка, все метаданные (такие как имена типов и методов) о коде отбрасываются. Напротив, когда C# компилируется в CIL, он поддерживает большинство метаданных о коде. Кроме того, используя отражение, можно перечислить все типы внутри сборки и выполнить поиск тех, которые соответствуют определенным критериям. Вы получаете доступ к метаданным типа экземпляров System.Type, и этот объект включает методы для перечисления членов экземпляра типа. Кроме того, эти элементы можно вызвать для объектов, которые относятся к обследованному типу.

Отражение открывает возможности множеству новых парадигм, которые в противном случае недоступны. Например, отражение позволяет перечислять все типы внутри сборки вместе с их членами и в процессе создавая заглушки для документации API сборки. Затем можно объединить метаданные, полученные из отражения, с XML-документом, созданным из комментариев XML (с помощью переключателя /doc), чтобы создать документацию по API. Аналогичным образом, программисты используют метаданные отражения для генерации кода для сохранения (сериализации) бизнес-объектов в базе данных. Его также можно использовать в элементе управления списком, который отображает коллекцию объектов. Учитывая коллекцию, элемент управления списком может использовать отражение для итерации по всем свойствам объекта в коллекции, определяя столбец в списке для каждого свойства. Кроме того, вызывая каждое свойство для каждого объекта, элемент управления списком может заполнять каждую строку и столбец данными, содержащимися в объекте, даже если тип данных объекта неизвестен во время компиляции.

XmlSerializer, ValueType и DataBinder Microsoft являются некоторыми из классов в фреймворка .NET Framework, которые используют отражение для частей их реализации.

### Доступ к метаданным с использованием System.Type

Ключом к чтению метаданных типа является получение экземпляра System.Type, который представляет экземпляр целевого типа. System.Type предоставляет все методы для получения информации о типе. Вы можете использовать его для ответа на такие вопросы, как:

 - Каково имя типа (Type.Name)?
 - Является ли тип public (Type.IsPublic)?
 - Каков базовый тип типа (Type.BaseType)?
 - Поддерживает ли тип какие-либо интерфейсы (Type.GetInterfaces())?
 - Какая сборка является типом, определенным в (Type.Assembly)?
 - Какие у типа есть свойства, метода, поля и т.д. (Type.GetProperties(), Type.GetMethods(), Type.GetFields(), and so on)?
 - Какие атрибуты декорирует тип (Type.GetCustomAttributes())?

Таких членов больше, но все они предоставляют информацию о конкретном типе. Ключ заключается в том, чтобы получить ссылку на тип объекта Type, и существует два основных способа сделать это - через object.GetType() и typeof(). Следует обратить внимание, что вызов метода GetMethods() не возвращает методы расширения. Эти методы доступны только как статические члены в типе реализации.

### GetType()

Любой объект включает элемент GetType(), и поэтому все типы включают эту функцию. Вы вызываете GetType () для извлечения экземпляра System.Type, соответствующего исходному объекту. Листинг 18.1 демонстрирует этот процесс, используя экземпляр Type из DateTime. Результаты.

     DateTime dateTime = new DateTime();
     Type type = dateTime.GetType();
     foreach(Reflection.PropertyInfo property in type.GetProperties())
     {
        Console.WriteLine(property.Name);
     } 
     Date
     Day
     DayOfWeek
     DayOfYear
     Hour
     Kind
     Millisecond
     Minute
     Month
     Now
     UtcNow
     Second
     Ticks
     TimeOfDay
     Today
     Year

После вызова GetType() выполняется итерирование по каждому экземпляру System.Reflection.PropertyInfo, возвращенному из Type.GetProperties() и отображающему имена свойств. Ключ к вызову GetType() заключается в том, что должен существовать экземпляр объекта. Однако иногда такой экземпляр недоступен. Объекты статических классов, например, не могут быть созданы, поэтому нет способа вызвать GetType().

### typeof()

Другой способ получить объект Type - выражение typeof. typeof связывается во время компиляции с конкретным экземпляром Type, и он принимает тип непосредственно в качестве параметра. Пример использования typeof с Enum.Parse().

     using System.Diagnostics;
     // ...
     ThreadPriorityLevel priority;
     priority = (ThreadPriorityLevel)Enum.Parse(typeof(ThreadPriorityLevel), "Idle");
     // ...

В этом листинге Enum.Parse () принимает объект Type, идентифицирующий перечисление, а затем преобразует строку в конкретное значение перечисления. В этом случае он преобразует «Idle» в System.Diagnostics.ThreadPriorityLevel.Idle. Выражение typeof разрешается во время компиляции.

### Вызов членов типа

Возможности отражения не останавливаются на поиске метаданных. Следующий возможный шаг - взять метаданные и динамически вызвать члены, на которые он ссылается. Рассмотрим возможность определения класса для представления командной строки приложения.1 Трудность с классом CommandLineInfo, например, связана с заполнением класса фактическими данными командной строки, которые запускали приложение. Однако, используя отражение, вы можете сопоставить параметры командной строки с именами свойств и затем динамически установить свойства во время выполнения. В листинге 18.3 показан этот процесс.

[Up](#content)

 ## <a name="2"></a> ????

[Up](#content)

## <a name="3"></a> ????

[Up](#content)

## <a name="4"></a> ????

[Up](#content)

# <a name="G"></a> Глоссарий

- 