[Moq](https://conf.parma.ru/display/ASUGF/Moq)
----------------------------------------------

**Вопрос:** как создать и использовать фейк (стаб или мок) класса?

**Ответ:** обычный workflow выглядит следующим образом:

1.  Создать фейк с помощью конструктора **`new Mock<тип>()`**.
2.  Задать возвращаемые нужными методами класса значения с помощью методов **`Setup`** / **`Returns`**.
3.  Получить нужную зависимость из свойства **`Object`** фейка.

**Пример**
```csharp
// Arrange
var stubRepository = new Mock<ICalculationValueRepository>();
stubRepository.Setup(r => r.GetCalculationValues(It.IsAny<int>()))
              .Returns(emptyCollection);

// Act
var service = CreateSut(repository: stubRepository.Object);
```
* * *

**Вопрос:** есть ли другие способы создания фейков?

**Ответ:** да, есть, они описаны в этой презентации:

![Презентация](moq-img/presentation.png "Презентация") 

* * *

**Вопрос:** как засетапить метод так, чтобы он возвращал не конкретное значение, а в зависимости от значений входых параметров?

**Ответ:** в **`Returns`** может быть использована соответствующая лямбда.

**Вот несколько примеров**

```csharp
stubPeriodService.Setup(ps => ps.GetPeriodTitleForCaption(It.IsAny<Period>()))
                 .Returns<Period>(period => $"text{period.Id.ToString()}");
stubVariableService
    .Setup(vs => vs.GetFinVariable(It.IsAny<int>(), It.IsAny<int>()))
    .Returns<int, int>((variableId, periodId)
                 => variables.Single(v => v.Id == variableId));

mockIndicatorService
    .Setup(s => s.GetSegmentIdForIndicatorId(It.IsAny<int>(), It.IsAny<int>()))
    .Returns<int, int>((indicatorId, periodId) => indicatorId / 10);
```
* * *

**Вопрос:** как засетапить метод так, чтобы при его вызове возникало исключение?

**Ответ:** используй метод **`Throws`**, Люк!

**Пример**
```csharp
stubSrv.Setup(srv => srv.GetCurrentReportingStage())
       .Throws<InvalidOperationException>();
```
* * *

**Вопрос:** как замокать свойство класса?

**Ответ:** либо с помощью метода **`SetupGet`**, либо так же, как и метод.

**Пример 1. SetupGet**
```csharp
var stubDateTimeProvider = new Mock<IDateTimeProvider>();
var now = DateTime.Parse("2019-04-01");
stubDateTimeProvider.SetupGet(dtp => dtp.Now).Returns(now);
```
**Пример 2**
```csharp
var stubDateTimeProvider = new Mock<IDateTimeProvider>();
stubDateTimeProvider.Setup(dtp => dtp.CurrentYear).Returns(currentYear); // Read only property
```
Если нужно просто включить отслеживание свойства, чтобы потом можно было проверить вызовы с помощью **`Verify`**, то тогда:

**Пример**
`mock.SetupProperty(f => f.Name);`

Или одновременно отслеживание и возвращаемое значение:

**Пример**
`mock.SetupProperty(f => f.Name, "foo");`

* * *

**Вопрос:** как проверить вызов метода определённое количество раз?

**Ответ:** с помощью метода **`Verify`** мока и структуры **`Times`**.

**Пример**
```csharp
stubRepository.Verify(r => r.Update(It.IsAny<DAL.RiskLevelWeight>(), It.IsAny<int>(), It.IsAny<int>(), It.IsAny<decimal>())
                         , Times.Exactly(ELEMENTS_COUNT));
stubRepository.Verify(r => r.SaveRiskLevelWeights(It.IsAny<IEnumerable<DAL.RiskLevelWeight>>())
                         , Times.Once);
```
* * *

**Вопрос:** какие есть варианты проверки предоставляет структура **`Times`**?

**Ответ:**

**Методы структуры Times**
```csharp
public static Times AtLeast(int callCount);
public static Times AtLeastOnce();
public static Times AtMost(int callCount);
public static Times AtMostOnce();
public static Times Between(int callCountFrom, int callCountTo, Range rangeKind);
public static Times Exactly(int callCount);
public static Times Never();
public static Times Once();
```
* * *

**Вопрос:** как проверить, что не было никаких больше вызовов методов, кроме проверяемых через **`Verify`**?

**Ответ:** после всех нужных **`Verify`** используйте **`VerifyNoOtherCalls`**.

**Пример**
```csharp
mockDocumentRepository.Verify(repo => repo.UpdateDocumentComment(ANY_ID, comment)
                            , Times.Once);
mockDocumentRepository.VerifyNoOtherCalls();
```
* * *

**Вопрос:** что означает конструкция **It.IsAny**?

**Ответ:** в методе **`Setup`** – что происходит настройка вызова метода фейка для **любых** значений входного параметра, в методе **`Verify`** – что нужно проверить вызов метода мока с **любыми** значениями входного параметра.

* * *

**Вопрос:** кроме варианта "любое значение" что-нибудь ещё есть?

**Ответ:**

**Методы класса It**
```csharp
public static TValue Is<TValue>(Expression<Func<TValue, bool>> match);
public static TValue IsAny<TValue>();
public static TValue IsIn<TValue>(IEnumerable<TValue> items);
public static TValue IsIn<TValue>(params TValue[] items);
public static TValue IsInRange<TValue>(TValue from, TValue to, Range rangeKind) where TValue : IComparable;
public static TValue IsNotIn<TValue>(IEnumerable<TValue> items);
public static TValue IsNotIn<TValue>(params TValue[] items);
public static TValue IsNotNull<TValue>();
public static string IsRegex(string regex);
public static string IsRegex(string regex, RegexOptions options);
```
**Примеры использования**
```csharp
mock.Setup(foo => foo.Add(It.Is<int>(i => i % 2 == 0))).Returns(true);
mock.Setup(foo => foo.Add(It.IsInRange<int>(0, 10, Range.Inclusive))).Returns(true);
mock.Setup(x => x.DoSomethingStringy(It.IsRegex("[a-d]+", RegexOptions.IgnoreCase))).Returns("foo");
```
* * *

**Вопрос:** а как проверить, что метод мока был вызван с каким-то **конкретным** значением параметра?

**Ответ:** в методе **`Verify`** вместо использования класса **`It`** указать эти конкретные значения.

**Пример**
```csharp
mockDocumentRepository.Verify(repo => repo.UpdateDocumentComment(ANY_ID, comment)
                            , Times.Once);
```
* * *

**Вопрос:** как замокать метод, не возвращающий значение (**`void`**)?

**Ответ:** с помощью метода **`Verifiable`**.

**Пример**
```csharp
_mockUserRepository.Setup(mr => mr.Update(It.IsAny<int>(), It.IsAny<string>()))
                   .Verifiable();
```
* * *

**Вопрос:** можно ли засетапить метод фейка так, чтобы он не просто возвращал нужное значение, а производились какие-то действия?

**Ответ:** да, вместо **`Returns`** нужно использовать **`CallBack`**.

**Пример**
```csharp
var stubIndicatorValidationService = new Mock<IIndicatorValidationService>();
stubIndicatorValidationService.Setup(ivs => ivs.Validate(It.IsAny<Indicator>()
                                                       , It.IsAny<bool>()))
    .Callback((Indicator indicator, bool isNew) =>
    {
        indicator.SetFieldError("field42", "errorMsg42");
    });
```
Впрочем, можно использовать и то, и то одновременно:

**Пример**
```csharp
var calls = 0;
mock.Setup(accountService => accountService.Notify(It.IsAny<AccountType>))
    .Returns(true)
    .Callback(() => calls++);
```
* * *

**Вопрос:** можно ли замокать не весь класс, а только один его метод? Т.е. фейк имеет точно такое же поведение, как и оригинальный класс, за исключением одного метода.

**Ответ:** да, алгоритм таков:

1.  Нужно создавать мок самого класса, а не его интерфейса.
2.  При создании мока нужно передать значения параметров для конструктора класса.
3.  Для мока надо установить свойство `CallBase` в **true**.
4.  Затем можно стандартным образом подменить нужные методы, но, естественно, они должны быть виртуальными (т.к. фактически происходит наследование подменяемого класса).

В примере ниже в конструктор подменяемого класса передаются два значения **`null`**.

**Пример**
```csharp
private readonly object[] CTOR_PARAMS = new object[] { null, null };
public void GetReportingStageIdByPeriod_NoPeriod_ReturnNull()
{
    // Arrange
    // Создаём mock для тестируемого класса (SUT)
    var periodServiceMock = new Mock<PeriodService>(MockBehavior.Default, CTOR_PARAMS)
    {
        // Вместо заглушек должны вызываться реальные методы
        CallBase = true
    };

    // Подменяем метод в SUT (метод должен быть виртуальным)
    periodServiceMock.Setup(ps => ps.GetPeriod(It.IsAny<int>())).Returns<int>(periodId => null);
    var service = periodServiceMock.Object;

    // Act
    var result = service.GetReportingStageIdByPeriod(It.IsAny<int>());

    // Assert
    Assert.That(result, Is.Null);
}
```