# NBuilder

* * *

**Вопрос:** как с помощью NBuilder-а создавать объекты нужного типа?

**Ответ:** использовать статический класс **Builder**, параметризованный нужным типом, настроить параметры создаваемого объекта или объектов через вызовы соответствующих методов (см. далее), в конце вызвать метод **Build()**. См. примеры ниже.

Все свойства объекта (объектов) будут инициализированы, те, которые не заполняются явно, получат случайные значения.

* * *

**Вопрос:** как с помощью NBuilder-а создать экземпляр нужного типа с заполнением нужных полей?

**Ответ:** для создания экземпляра используется метод **CreateNew()**, для заполнения полей – метод **With()**.

**Пример**
```csharp
var dalGrbs = Builder<DAL.Grbs>
    .CreateNew()
    .With(g => g.Id = EXPECTED\_GRBS\_ID)
    .Build();
var dalGrbsHistory = Builder<DAL.GrbsHistory>
    .CreateNew()
    .With(gh => gh.GrbsId = EXPECTED\_GRBS\_ID)
    .With(gh => gh.Grbs = dalGrbs)
    .Build();
```

**Примечание:** в случае нескольких последовательных вызовов метода **`With()`** можно их все, кроме первого, заменить на **`And()`**.

**Пример**
```csharp
var person = Builder<Person>
    .CreateNew()
    .With(p => p.Name = "Juke")
    .And(p => p.BirthDay = DateTime.Now.AddYears(-30))
    .And(p => p.Id = 42)
    .Build();
```

* * *

**Вопрос:** как создать коллекцию экземпляров нужного типа?

**Ответ:** использовать метод **`CreateListOfSize()`**. См. пример "Создание коллекции".  

**Пример “Создание коллекции”**
```csharp
private static IQueryable<DAL.BudgetCycle> GetDalBudgetCycles()
{
    return Builder<DAL.BudgetCycle>
        .CreateListOfSize(10)
        .All()
        .With(bc => bc.IsCurrent = false)
        .Random(5)
        .With(bc => bc.IsHidden = true)
        .Random(1)
        .With(bc => bc.IsCurrent = true)
        .With(bc => bc.IsHidden = false)
        .Build()
        .AsQueryable();
}
```

* * *

**Вопрос:** как заполнить какое-либо свойство у всех элементов коллекции?

**Ответ**: использовать последовательный вызов методов **`All()`** и **`With()`**. См. пример "Создание коллекции".

* * *

**Вопрос:** как заполнить какое-либо свойство у N элементов коллекции?

**Ответ:** использовать последовательный вызов методов **`Random(N)`** и **`With()`**. См. пример "Создание коллекции".

* * *

**Вопрос:** какие ещё есть методы, позволяющие обработать некоторые из элементов коллекции?

**Ответ:**

*   **`TheFirst(N)`** – первые N элементов
*   **`TheLast(N)`** – последние N элементов
*   **`TheNext(N)`** – следующие N элементов
*   **`TheRest()`** – все оставшиеся элементы

**Примеры**
```csharp
var lst1 = Builder<Person>
    .CreateListOfSize(5)
    .TheFirst(2)
    .With(p => p.Name = "First")
    .TheLast(1)
    .With(p => p.Name = "Last")
    .Build();
var lst2 = Builder<Person>
    .CreateListOfSize(5)
    .TheFirst(1)
    .With(p => p.Name = "First")
    .TheNext(2)
    .With(p => p.Name = "Second")
    .TheRest()
    .With(p => p.Name = "Rest")
    .Build();
```

* * *

**Вопрос:** как заполнять ссылки на элементы родительского справочника?

**Ответ:** использовать класс **`Pick<T>`** и его метод **`RandomItemFrom()`**.

**Пример**
```csharp
var deps = Builder<Department>
    .CreateListOfSize(8)
    .Build();
var employees = Builder<Person>
    .CreateListOfSize(5)
    .All()
    .With(p => p.Department = Pick<Department>.RandomItemFrom(deps))
    .Build();
```

* * *

**Вопрос:** как заполнять ссылки на элементы дочернего справочника?

**Ответ:** использовать класс **`Pick<T>`** и его метод **`UniqueRandomList()`**.

**Пример**
```csharp
var employees = Builder<Person>
    .CreateListOfSize(20)
    .Build();
var deps = Builder<Department>.CreateListOfSize(3)
    .All()
    .With(d => d.Persons = Pick<Person>.UniqueRandomList(With.Between(3, 7).Elements)
                                       .From(persons))
    .Build();
```