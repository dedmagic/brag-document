# NUnit: параметризованные тесты

* * *

**Вопрос:** что такое параметризованный тест?

**Ответ:** это тестовый метод, который при одном запуске выполняется несколько раз для разных наборов входных данных.

* * *

**Вопрос:** с помощью каких средств NUnit можно создать параметризованный тест?

**Ответ:** вот их список:

*   атрибут метода **`[TestCase]`**
*   атрибут метода **`[TestCaseSource]`**
*   атрибут параметра метода **`[Values]`**

* * *

**Вопрос:** как задать несколько наборов входных данных для теста с помощью атрибута **`[TestCase]`**?

**Ответ:**

Первый вариант использования: перед методом перечислить в атрибутах **`[TestCase]`** все наборы входных данных. Тестовый метод будет выполнен столько раз, сколько атрибутов указано, при этом параметры атрибута буду переданы в метод как входные параметры. Естественно, количество, тип и порядок параметров атрибутов и параметров метода должны совпадать.

**Пример**
```csharp
[TestCase("2018-01-01", "2018-01-03", "2018-01-02", true)]
[TestCase("2018-01-01", "2018-01-01", "2018-01-01", true)]
[TestCase("2018-01-01", "2018-01-02", "2018-01-03", false)]
public void IsInRange_AllCases(string beginDateStr, string endDateStr, string checkedDateStr, bool expectedResult)
```

Можно несколько атрибутов объединить в один, но не рекомендуется, т.к. это ухудшает читабельность:

**Пример**
```csharp
[TestCase(2, true), TestCase(42, false)]
public void SomeTest(int budgetCycleId, bool hasElements)
```

* * *

**Вопрос:** а какой второй вариант?

**Ответ:** в атрибут **`[TestCase]`** добавить параметр **`ExpectedResult`**. В этом случае в тесте вместо **`Assert`** используется возврат значения через **`return`**. Тест считается пройденным, если возвращаемое им значение совпало со значением **`ExpectedResult`**.

**Пример**
```csharp
[TestCase("100500", ProgramTypeEnum.Program, ProgramTypeEnum.Program, ExpectedResult = "100500")]
[TestCase("100500", ProgramTypeEnum.Program, ProgramTypeEnum.SubProgram, ExpectedResult = null)]
[TestCase("100500", ProgramTypeEnum.Program, ProgramTypeEnum.MainAction, ExpectedResult = null)]
[TestCase("15.12.000.000.00", ProgramTypeEnum.SubProgram, ProgramTypeEnum.Program, ExpectedResult = "15.00.000.000.00")]
[TestCase("15.12.000.000.00", ProgramTypeEnum.SubProgram, ProgramTypeEnum.SubProgram, ExpectedResult = "15.12.000.000.00")]
[TestCase("15.12.333.000.00", ProgramTypeEnum.MainAction, ProgramTypeEnum.Program, ExpectedResult = "15.00.000.000.00")]
[TestCase("15.12.333.000.00", ProgramTypeEnum.MainAction, ProgramTypeEnum.SubProgram, ExpectedResult = "15.12.000.000.00")]
[TestCase("15.12.333.000.00", ProgramTypeEnum.MainAction, ProgramTypeEnum.MainAction, ExpectedResult = "15.12.333.000.00")]
public string GetStateRegulationItemCodeTest(string code, ProgramTypeEnum actualProgramType, ProgramTypeEnum desiredProgramType)
{
    return StateRegulationService.GetStateRegulationItemCode(code, actualProgramType, desiredProgramType);
}
```

* * *

**Вопрос:** как использовать атрибут **`[TestCaseSource]`**?

**Ответ**: в атрибуте **`[TestCaseSource]`** в качестве параметра указывается имя либо статической коллекции **`IEnumerable`** (см. пример 1), либо статического метода, возвращающего такую коллекцию (см. пример 2). Тестовый метод будет вызван для каждого элемента этой коллекции.

**Пример 1**
```csharp
private static readonly List<int> _indicatorIds = new List<int>
{
    FinManagementConfig.IndicatorP25Id,
    FinManagementConfig.IndicatorP26Id,
    FinManagementConfig.IndicatorP32Id,
    FinManagementConfig.IndicatorCurrentP62FormerP63Id,
    FinManagementConfig.IndicatorCurrentP63FormerP64Id,
    FinManagementConfig.IndicatorP75Id,
    FinManagementConfig.IndicatorP712Id
};

[TestCaseSource(nameof(_indicatorIds))]
public void TrySpecialCalculationP_OivInCategory_ReturnsNotNull(int indicatorId)
```

**Пример 2**
```csharp
private static object[] ValidateCodeFormatTestCases()
{
    return new object[]
    {
        // Код отсутствует или пустой
        new object[] { null, true, Config.FieldIsMandatoryErrorMsg },
        new object[] { String.Empty, true, Config.FieldIsMandatoryErrorMsg },
        new object[] { " ", true, Config.FieldIsMandatoryErrorMsg },
        new object[] { new String(' ', 42), true, Config.FieldIsMandatoryErrorMsg },
        new object[] { "\t \r\n ", true, Config.FieldIsMandatoryErrorMsg },

        // Первый или второй символ -- не буква латинского алфавита
        new object[] { "_A1.2", true, Config.VariableCodeErrorMsg },
        new object[] { "@A1.2", true, Config.VariableCodeErrorMsg },
        new object[] { "1.2", true, Config.VariableCodeErrorMsg },
        new object[] { "Ы1.2", true, Config.VariableCodeErrorMsg },
        new object[] { "ZЫ1.2", true, Config.VariableCodeErrorMsg },

        // Больше двух букв в начале
        new object[] { "ABC1.1", true, Config.VariableCodeErrorMsg },

        // Перед и после точки должны быть группы цифр
        new object[] { "A1x.1", true, Config.VariableCodeErrorMsg },
        new object[] { "A1.x1", true, Config.VariableCodeErrorMsg },
        new object[] { "A1.1x", true, Config.VariableCodeErrorMsg },

        // Корректный код
        new object[] { "A1.1", false, null },
        new object[] { "A11.1", false, null },
        new object[] { "A111.1", false, null },
        new object[] { "A11111111111.1", false, null },
        new object[] { "A.1", false, null },
        new object[] { "A1", false, null },
        new object[] { "A1.", false, null },
        new object[] { "AB1.1", false, null },
        new object[] { "a1.1", false, null },
        new object[] { "A1.111111", false, null },
        new object[] { "Z1.1", false, null }
    };
}

[TestCaseSource(nameof(ValidateCodeFormatTestCases))]
public void ValidateCodeFormat_AllTestCases(string code, bool hasCodeErrors, string errorMsg)
```
* * *

**Вопрос:** как сформировать набор тест-кейсов, если тесту для работы требуется несколько параметров?

**Ответ:** первый вариант – данные для теста можно представить в виде массива **`object[]`** (или коллекции **`IEnumerable<object>`**), который содержит все тест-кейсы, каждый элемент которого в свою очередь является массивом **`object[]`**, который содержит значения всех параметров теста. См. пример 2 из предыдущего вопроса и пример ниже.

**Пример**
```csharp
private static IEnumerable<object> GetPeriodTitleForCaptionTestCases()
{
    yield return new object[] { null, 1, "за I квартал 0 года" };
    yield return new object[] { null, 2, "за первое полугодие 0 года" };
    yield return new object[] { null, 3, "за III квартал 0 года" };
    yield return new object[] { null, 4, "за 0 год" };
    yield return new object[] { 2017, null, $"за {REPORTING_STAGE_NAME} 2017 года" };
    yield return new object[] { 2018, null, $"за {REPORTING_STAGE_NAME} 2018 года" };
    yield return new object[] { 2017, 1, "за I квартал 2017 года" };
    yield return new object[] { 2018, 1, "за I квартал 2018 года" };
    yield return new object[] { 2017, 2, "за первое полугодие 2017 года" };
    yield return new object[] { 2018, 2, "за первое полугодие 2018 года" };
    yield return new object[] { 2017, 3, "за III квартал 2017 года" };
    yield return new object[] { 2018, 3, "за III квартал 2018 года" };
    yield return new object[] { 2017, 4, "за 2017 год" };
    yield return new object[] { 2018, 4, "за 2018 год" };
}

[TestCaseSource(nameof(GetPeriodTitleForCaptionTestCases))]
public void GetPeriodTitleForCaption_AllCasesExpceptPeriodIsNull(int year, int reportingStageId, string expectedResult)
```

* * *

**Вопрос:** конечно же, хочется и второй вариант.

**Ответ:** можно "упаковать" все параметры теста в один объект типа **`TestCaseData`**, в этом случае источником данных для теста является коллекция **`IEnumerable<TestCaseData>`** или метод, возвращающий такую коллекцию.

**Пример**
```csharp
private static IEnumerable<TestCaseData> ContainsRowCases()
{
    yield return new TestCaseData(false, Dimension.MinValue, false, null);
    yield return new TestCaseData(true, Dimension.MinValue, true, null);
    yield return new TestCaseData(false, Dimension.MinValue, true, Guid.NewGuid());
    yield return new TestCaseData(true, Dimension.ExpensesType, false, null);
    yield return new TestCaseData(true, Dimension.ExpensesType, true, null);
    yield return new TestCaseData(false, Dimension.ExpensesType, true, Guid.NewGuid());
}

[TestCaseSource(nameof(ContainsRowCases))]
public void ToFlat_TopLevelDimension_ContainsRows(bool isResultContainsRow, Dimension dimension, bool isMinValue, Guid? rootRowKey)
```

* * *

**Вопрос:** что ещё умеют объекты **`TestCaseData`**?

**Ответ:** к ним можно с помощью метода **`Returns`** "прикреплять" ожидаемое возвращаемое значение, это действует подобно параметру **`ExpectedResult`** в атрибуте **`[TestCase]`**. Т.е. при использовании данной возможности тест должен вместо явного использования assert-ов возвращать значение через **`return`**, оно будет сверяться с указанным в объекте **`TestCaseData`** и в случае несовпадения тест счтитается непройденным.

**Пример**
```csharp
private static IEnumerable<TestCaseData> GetKspsTestCases()
{
    var ksps = GetDalKsps();
    foreach (SortDirectionType sortType in Enum.GetValues(typeof(SortDirectionType)))
    {
        var isAscendingSorting = sortType == SortDirectionType.Asc;
        yield return new TestCaseData(KspSortType.ByCode, sortType, ksps)
            .Returns(ksps.OrderBy(k => k.Code, isAscendingSorting).Select(k => k.Id));
        yield return new TestCaseData(KspSortType.ByName, sortType, ksps)
            .Returns(ksps.OrderBy(k => k.Name, isAscendingSorting).Select(k => k.Id));
        yield return new TestCaseData(KspSortType.ByBeginDate, sortType, ksps)
            .Returns(ksps.OrderBy(k => k.BeginDate, isAscendingSorting).Select(k => k.Id));
        yield return new TestCaseData(KspSortType.ByEndDate, sortType, ksps)
            .Returns(ksps.OrderBy(k => k.EndDate, isAscendingSorting).Select(k => k.Id));
    }

    yield return new TestCaseData(null, null, ksps)
            .Returns(ksps.OrderBy(k => k.Code).Select(k => k.Id));
    yield return new TestCaseData(null, It.IsAny<SortDirectionType>(), ksps)
            .Returns(ksps.OrderBy(k => k.Code).Select(k => k.Id));
    yield return new TestCaseData(It.IsAny<KspSortType>(), null, ksps)
            .Returns(ksps.OrderBy(k => k.Code).Select(k => k.Id));
}

[TestCaseSource(nameof(GetKspsTestCases))]
public IEnumerable<int> GetKsps_AllScenarios(KspSortType? sortType, SortDirectionType? sortDirection, IQueryable<DAL.Ksp> dalEntities)
{
    // Arrange
    var stubKspRepository = Mock.Of<IKspRepository>(r => r.GetAllKsps() == dalEntities);

    // Act
    return CreateSut(kspRepository: stubKspRepository)
        .GetKsps(pageNumber: 1, sortType, sortDirection)
        .Select(ksp => ksp.Id);
}
```

* * *

**Вопрос:** а что там с **`[Values]`**?

**Ответ:** в простейших случаях тест-кейсы можно перечислить в атрибуте , применяемом к параметру метода. Вместо

**Пример. TestCase**
```csharp
[TestCase("12")]
[TestCase("123")]
[TestCase("")]
[TestCase(null)]
public void OrderedCode_CodeLengthNotEqualOne_ReturnTheSameCode(string code)
```

можно написать

**Пример. Values**
```csharp
public void OrderedCode_CodeLengthNotEqualOne_ReturnTheSameCode([Values("12", "123", "", null)] string code)
```

* * *

**Вопрос:** что-нибудь ещё с помощью **`[Values]`** можно делать?

**Ответ:** можно перебрать **все** значения для ограниченных типов, таких как **`bool`** или **`enum`**. Вместо

**Пример. TestCase**
```csharp
[TestCase(true)]
[TestCase(false)]
public void SetSaveState_TheSameSaveState_NoChanges(bool saveState)
```

можно

**Пример. Values**
```csharp
[Test]
public void SetSaveState_TheSameSaveState_NoChanges([Values] bool saveState)
```