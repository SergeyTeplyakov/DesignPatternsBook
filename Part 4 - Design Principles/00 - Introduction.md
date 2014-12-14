# Часть 2. Принципы проектирования

Любая индустрия по мере взросления старается делиться своим опытом с подрастающим поколением. Паттерны проектирования, рассмотренные в предыдущей части, являются отличным примером повторного использования знаний и опыта более опытных проектировщиков. Паттерны весьма полезны (иначе зачем бы я решил написать еще одну книгу по этой теме?), но они показывают типовые решения типовых задач. Их можно обобщить, можно придумать паттерны для конкретной предметной области, но многим все равно будет не хватать более фундаментальных путеводных нитей, за которыми будет двигаться развитие дизайна.

В этой части мы рассмотрим популярные сегодня принципы проектирования, получившие звучную аббревиатуру - SOLID. В середине 90-х годов Боб Мартин начал публикацию статей в журнале C++ Report на тему проектирования. В качестве основы были взяты несколько известных ранее принципов проектирования, добавлены свои собственные мысли и на свет появились фундаментальные принципы объектно-ориентированного проектирования.
Изначально эти принципы были описаны несколько в ином порядке, и в архиве старины "дядюшки Боба" значатся под абревиатурой SOLDI. Через несколько лет они перекочевали в книгу ["Agile Software Development, Principles, Patterns, and Practices"](http://www.amazon.com/Software-Development-Principles-Patterns-Practices/dp/0135974445/), а со временем в аналогичную книгу с примерами на языке C# - ["Принципы, практики и методики гибкой разработки на языке C#"](http://www.ozon.ru/context/detail/id/5800704/), но в них порядок уже был иным и появились "цельные" (SOLID) принципы, звучность названия которых в немалой степени обеспечила им успех.

Оригинальные статьи были опубликованы почти 20 лет назад, в качестве примеров использовался язык С++, а объектно-ориентированное мышление лишь только становилось "мейнстримом". Столь почтенный возраст дает о себе знать, и оригинальные статьи старины Боба содержат ряд советов, полезных лишь в языке С++ двадцатилетней давности, и основной упор делается на критике структурного и восхвалении объектно-ориентированного подхода. Любопытно, что многие "современные" описания SOLID принципов все еще показывают пользу полиморфизма на примере иерархии фигур и говорят о проблемах транзитивных зависимостей, которых далеко не столь актуальны в C# или Java.

Я же хочу рассмотреть SOLID принципы с более абстрактной точки зрения, подумать о том, какую проблему они призваны решать (и решают ли), и как мы должны смотреть на них сегодня, когда ООП уже давно стало широко распространённой парадигмой программирования.

В качестве примеров, в этой части будут рассмотрены фрагменты кода и дизайна из одного из моих домашних проектов - [Code Contract Editor Extension](https://github.com/SergeyTeplyakov/ReSharperContractExtensions) - плагина для популярного инструмента разработки [ReSharper](https://www.jetbrains.com/resharper/), суть которого сводится к упрощению контрактного программирования и использования библиотеки [Code Contracts]().

## Глава 1. Размышления о принципах проектирования
**ЦИТАТА**: _Век живи — век учись! И ты наконец достигнешь того, что, подобно мудрецу, будешь иметь право сказать, что ничего не знаешь._ Козьма Прутков
Или: _Отыщи всему начало, и ты многое поймешь._

Для чего выдумывать все эти паттерны проектирования, принципы и методики? Разве не было бы проще обойтись без всего этого, а просто научить разработчиков хорошему дизайну? Или почему бы не формализовать этот процесс и ввести четкие количественные метрики, которые бы говорили, что одно решение однозначно лучше другого?

«Правильный дизайн» - это святой Грааль молодых разработчиков и молодых менеджеров. И те, и другие мечтают найти ответ на главный вопрос ~~жизни, вселенной и всего такого~~ разработки ПО – как добиться качественного дизайна, в сжатые сроки и с минимумом усилий.

Со временем и молодой разработчик, и молодой менеджер придут к пониманию того, что это невозможно. Невозможно найти идеальный абстрактный дизайн, поскольку слова «идеальный» и «абстрактный» противоречат друг другу. Дизайн – это постоянный поиск компромисса между противоречивыми требованиями: производительностью и читабельностью, простотой и расширяемостью, тестируемостью и цельностью решения.

Даже если учитывать, что разработчик всегда решает правильную задачу, а не борется с ветряными мельницами, нельзя «абстрактно» сказать, какие характеристики дизайна являются ключевыми здесь и сейчас.

Существуют формальные критерии, которые описывают качество кода или дизайна: цикломатическая сложность методов, глубина иерархии наследования, число входящих и исходящих связей класса или модуля, число строк метода, в конце концов. Эти количественные показатели полезны, но попадание их в заданные границы является необходимым, но недостаточным условием хорошего дизайна. Если классы разбиты неумело, а важные абстракции предметной области не выявлены, то какими бы количественными характеристиками не обладал дизайн, он никогда не будет хорошим.

Помимо формальных критериев, есть универсальные понятия хорошего дизайна – слабая связанность (low coupling) и сильная связность (high cohesion). Данные свойства полезны, но слишком неформальны.

Между формальными и неформальными критериями находятся принципы проектирования – набор правил, на которые опираются опытные проектировщики. Их цель: описать простыми словами, что такое «хорошо, а что такое плохо» в вопросах дизайна: «Идеальный класс должен иметь лишь одну причину для изменения, обладать минимальным интерфейсом, правильно реализовывать наследование и предотвращать каскадные изменения в коде при изменении требований».

Бертран Мейер в своей книге “Agile!: The Good, The Hype, and The Ugly” дает достаточное четкое определение того, что такое принцип проектирования [Meyer2014]:

"Принцип – это методологическое правило, которое выражает общий взгляд на разработку ПО. Хороший принцип является одновременно _абстрактным_ и _опровергаемым_ (falsifiable). **Абстрактность** отличает принцип от практик, а опровергаемость отличает принцип от банальности (platitude). Абстрактность означает, что принцип должен описывать универсальное правило, а не конкретную практику. **Опровергаемость** означает, что у разумного человека должна быть возможность не согласиться с принципом. Если никто в здравом уме не будет оспаривать предложенный принцип, то это правило будет полезным, но не интересным. Чтобы правило являлось принципом – не зависимо от вашего мнения – вы должны предполагать наличие людей, придерживающихся противоположной точки зрения."

### SOLID принципы
В области разработки ПО существует множество разных паттернов, но именно "цельные" принципы наиболее популярны. Звучность название, простота в запоминании, а также их универсальная применимость сделали их самыми популярными сегодня принципами объектно-ориентированного проектирования.

**Принцип единой обязанности** (SRP - The **S**ingle Responsibility Principle: _у класса/модуля должна быть лишь одна причина для изменения_.
Данный принцип говорит о борьбе с изменениями, но на самом деле суть его сводится к борьбе со сложностью (tackling the complexity). Любой сложный класс должен быть разбит на несколько простых составляющих, отвечающих за определенный аспект поведения, что упрощает как понимание, так и будущее развитие класса или модуля. Простой класс с небольшим числом зависимостей легко изменить, не зависимо от того, сколько причин для изменения существует. Разработчик очень редко знает, куда в следующий раз подует ветер ~~перемен~~ изменения требований, поэтому лучший способ обеспечения гибкости решения является его простота, когда класс модуль отвечает лишь за один аспект.

**Принцип открыт/закрыт** (OCP - The **O** pen-Closed Principle): _программные сущности (классы, модули, функции и т.п.) должны быть открытыми для расширения, но закрытыми для модификации_. Любой готовый модуль должен быть стабильным (закрытым) с точки зрения своего интерфейса, но открытым с точки зрения реализации. Закрытость модулей означает стабильность интерфейса и возможность использования классов/модулей клиентами. Открытость модулей означает возможность внесения изменений поведения, путем изменения реализации или же путем переопределения поведения в наследниках. Сокрытие информации и полиморфизм позволяют ограничить количество изменений, которые понадобится внести в систему при очередном изменении требований, что сделает этот процесс более простым и управляемым.

**Принцип замещения Лисков** (LSP - The **L** iskov Substitution Principle): _должна быть возможность вместо базового типа подставить любой его подтип_. Поскольку наследование является одним из ключевых механизмов объектно-ориентированного проектирования, очень важно использовать его корректным образом. Данный принцип дает четкие рекомендации о том, в каких пределах может изменяться поведение переопределенных наследником методов, чтобы между классами сохранялось отношение "ЯВЛЯЕТСЯ".

**Принцип разделения интерфейсов** (ISP - The **I** nterface Segregation Principle): _клиенты не должны вынужденно зависеть от методов, которыми не пользуются_. Интерфейс класса определяется некоторым контрактом, который он обязуется выполнять для своих клиентов. Иногда возникают ситуации, когда две разные группы клиентов, начинают тянуть интерфейс класса в разные стороны, в результате чего интерфейс становится несогласованный и неудобным в использовании. Данный принцип говорит о том, что клиенты хотят иметь цельный и согласованный интерфейс сервисов, не зависимо от того, пользуются ли этими сервисами кроме них кто-то еще или нет.

**Принцип инверсии зависимостей** (DIP - The **D** ependency Inversion Principle): _Модули верхнего уровня не должны зависеть от модулей нижнего уровня. И те и другие должны зависеть от абстракций._ Слишком большое число зависимостей класса говорит о проблемах в его дизайне. Возможно класс делает слишком многое, или же текущий класс неудачно спроектирован, что приводит к необходимости вызова по одному методу у слишком большого числа зависимостей. Любой объектный дизайн представляет собой некоторый граф взаимодействующих объектов, при этом некоторые зависимости являются частью реализации и должны создаваться напрямую, а некоторые – передаваться ему извне. Данный принцип говорит о необходимости выделения и передачи ключевых зависимостей через аргументы конструктора, что позволяет перенести проблемы по созданию и выбору конкретного типа "внешнего поведения" на более верхний уровень.

Данные принципы не являются панацеей для получения хорошего дизайна, они не всегда применимы (а значит, они удовлетворяют данному выше определению), и не всегда понятны новичку.

### Уровни владения принципами проектирования
TODO: найти такую же картинку, но с copywritами.

![Su Ha Ri](https://github.com/SergeyTeplyakov/DesignPatternsBook/raw/master/Part%204%20-%20Design%20Principles/Images/00_Introduction_Image_1.jpg )

Разработчик за свою профессиональную карьеру проходит несколько стадий владения таким инструментом, как принципы проектирования.

**На первой стадии** молодой разработчик еще не дорос до абстрактных принципов, вместо этого он ищет набор конкретных практик, которые позволят ему получить качественное решение. «Я хочу узнать набор шагов, четкое следование которым приведет меня к нужному результату!». На этом этапе разработчик хорошо копирует чужое решение, и сталкивается с серьезными трудностями, если описание слишком абстрактное или не абсолютно подходит к его случаю.

**На второй стадии** разработчик начинает понимать, что лежит в основе конкретной практики, и для чего нужны принципы проектирования. «Ага, этот класс нарушает принцип единой обязанности, поскольку он ходит в базу и содержит бизнес логику! Получается, что у него есть две четкие причины для изменения!».

Несмотря на возросший уровень, именно на этом этапе происходит наиболее частое использование принципов или паттернов не по назначению. Легко доказать, что конкретный класс нарушает данный принцип, но на этом этапе развития не всегда очевидно, когда это нарушение оправдано, а когда нет.

**На третьей стадии** у разработчика (или уже скорее Архитектора) развивается чутье и появляется довольно четкое понимание, какую проблему призван решить конкретный принцип проектирования. Поскольку по своему определению принцип не является однозначным, у опытного разработчика появляется понимание, когда нарушение оправдано, когда с ним можно жить, а когда пришло время браться за исправление дизайна.

На этом этапе принципы не управляют дизайном приложения, а начинают играть скорее коммуникативную роль: «Смотри, у тебя класс делает слишком многое, значит он нарушает принцип единой обязанности! А у этого класса есть два вида клиентов, каждый из которых использует этот класс по-своему, значит он нарушает принцип разделения интерфейсов!».

**ПРИМЕЧАНИЕ**
В боевых искусствах принято выделять три стадии мастерства: сю, ха, и ри ([Shu, Ha, Ri](https://ru.wikipedia.org/wiki/%D0%A1%D1%8E%D1%85%D0%B0%D1%80%D0%B8)). На первой ступени находится ученик, который лишь повторяет движения за мастером. На второй ступени ученик начинает освобождаться от правил и сам начинает решать, когда им следовать, а когда – нет. На третьей стадии правила пропадают, ученик становится мастером и может сам эти правила создавать. Эта же модель, но несколько под иным соусом появилась и в американской культуре, под названием [Модель Дрейфуса](http://en.wikipedia.org/wiki/Dreyfus_model_of_skill_acquisition).

### Правильное использование принципов
Применение любого принципа проектирования имеет свою цену. Дробление класса на более мелкие составляющие, чтобы он отвечал принципу единой обязанности, может привести к размазыванию логики по нескольким классам (low cohesion), а иногда и к падению производительности.

Нарушение принципа открыт-закрыт может быть оправдано вопросами обратной совместимости. Мы можем игнорировать принцип замещения Лисков, поскольку наследование не всегда определяет отношение подтипов. Интерфейс может быть толстым из-за обратной совместимости или удобства использования. А инверсия зависимостей легко может подорвать инкапсуляцию и привести к полнейшему ООП головного мозга.

Чтобы эффективно пользоваться принципами нужно знать, какую проблему они на самом деле решают и является ли она для вас сейчас актуальной. Обычно актуальность принципов возрастает при увеличении сложности, или при увеличении стоимости внесения изменений. Ключевая бизнес-логика приложения является одним из примеров того, что должно быть изолировано от остального мира. Любые публично доступные классы должны иметь минимум дополнительных связей с внешним миром и должны легко позволять делать простые вещи. Классы, которые находятся на стыке модулей должны быть продуманы лучше остальных.

Ключом для получения хорошего дизайна и эффективного использования принципов проектирования является итеративный подход к дизайну. На ранних этапах мы мало что понимаем в предметной области, поэтому все, что мы можем сделать – это разбить модули по ролям: инфраструктуру отдельно, логику отдельно. При этом автономность классов и минимизация побочных эффектов делает дизайн проще, поскольку количество связей класса/модуля остается строго ограниченным.

По мере развития приложения улучшается понимание разработчиками предметной области. Но одновременно с этим, приложение обрастает заплатками и неуклюжими решениями. Накапливается технический долг, который приводит к усложнению добавления новых возможностей, а исправление одной ошибки приводит к появлению двух других. Чтобы не доводить систему до такого состояния, лучше постоянно думать о дизайне приложения и улучшать его по ходу развития. Перед реализацией новой возможности или исправлением ошибки, я стараюсь ответить на такой вопрос: отвечает ли текущий дизайн моему новому пониманию задачи, и требуется ли некоторая его переработка?

Главной движущей силой изменений в конечном счете является возрастающая сложность. Как только мне не удается держать в голове решение, структура становится сложной и запутанной или не понятно, куда вносить изменения, значит пришло время подумать, как его улучшить.

«Класс стал слишком сложным, он нарушает принцип единой обязанности, пришло время разбить его на два. Информация об иерархии наследования расползлась по всему модулю, наверное, стоит спрятать ее за фабрикой. Класс невозможно протестировать из-за обилия скрытых зависимостей, пришло время выделить зависимости класса.»

При этом я всегда проверяю, не привели ли меня принципы или паттерны в мир излишнего проектирования (overdesign): стал ли мой дизайн после внесения изменений проще? Не решаю ли я проблему, которой на самом деле не существует? Не попал ли я в сети преждевременного обобщения (premature generalization)? Иногда приходится провести несколько итераций, прежде чем удается найти разумное решение некоторой задачи.

### Антипринципы проектирования

Популярность принципов проектирования легко может сыграть с вами и вашей командой злую шутку. Чрезмерная любовь к принципам и паттернам может проявляться в виде оверинжиниринга и чрезмерной сложности. Но зная об этом, мы можем подготовиться и предугадать, как именно будет проявляться чрезмерная любовь принципов в коде приложения.

**Anti-SRP – Принцип размытой обязанности**. Классы разбиты на множество мелких классов, в результате чего логика размазывается по нескольким классам/модулям.

**Anti-OCP – Принцип фабрики-фабрик**. Дизайн является слишком обобщенным и расширябельным, выделяется слишком большое число уровней абстракции.

**Anti-LCP – Принцип непонятного наследования**. Принцип проявляется либо в чрезмерном количестве наследования, либо в его полном отсутствии, в зависимости от опыта и взглядов местного главного архитектора.

**Anti-ISP – Принцип тысячи интерфейсов**. Интерфейсы классов разбиваются на слишком большое число составляющих, что делает их неудобными для использования всеми клиентами.

**Anti-DIP – Принцип инверсии сознания или DI-головного мозга**. Интерфейсы выделяются для каждого класса и пачками передаются через конструкторы. Понять, где находится логика становится практически невозможно.”.