# Требования к unit-тестам

**Пример тестового метода**

```csharp
[Test]
public void GetIndicator_DalObjectExists_ReturnsBllObject()
{
    // Arrange
    var dalObject = new DAL.IndicatorHistory { Id = 42, IndicatorId = 123 };
 
    var stubRepository = new Mock<IIndicatorRepository>();
    stubRepository.Setup(r => r.GetIndicatorHistory(It.IsAny<int>(), It.IsAny<int>()))
                    .Returns(dalObject);
 
    var sut = CreateSut(repository: stubRepository.Object);
 
    // Act
    var indicator = sut.GetIndicator(indicatorId: 42_1, periodId: 42_2);
 
    // Assert
    indicator.Id.Should().Be(dalObject.IndicatorId);
    indicator.HistoryId.Should().Be(dalObject.Id);
}
```

1. Структура тестовых методов должна строится в соответствии с шаблоном AAA (Arrange, Act, Assert).

2. Каждая секция тестового метода должна начинаться с комментария-заголовка: `// Arrange`, `// Act`, `// Assert` (см. пример). Секции должны отделяться друг от друга одной пустой строкой.

3. В разделе `Arrange` тестового метода происходит создание тестовой конфигурации (т.е. задание исходных данных для теста), настройка заглушек (fakes) и создание тестируемого класса (SUT). Секция `Act` служит для вызова тестируемого метода (см. пример).

4. Имя тестового метода формируется по следующему шаблону: _**ИмяТестируемогоМетода_ОписаниеТестовойКонфигурации_ОжидаемыйРезультат**_. Например: `UpdateSegmentHistory_BadName_NoUpdate`, `GetFinVariableByHistoryId_NotExistVariableHistory_ReturnsNull`. Допустимо использовать несколько частей с описанием тестовой конфигурации: `UpdateRiskLevels_IdsIsNull_WeightsIsNull_NoChanges`.    
Тестовый метод, в котором проверяется работа основного сценария (не крайние случаи) допустимо назвать `ИмяТестируемогоМетода_MainScenario`.

5. В именах стабов и моков должны использоваться префиксы `stub` и `mock`.

6. Метод, возвращающий экземпляр тестируемого класса, должен называться `CreateSut`. Переменная-экземпляр тестируемого класса должна называться `sut`.

7. При именовании всех локальных переменных тестового метода, кроме переменной `sut`, необходимо придерживаться тех же принципов, что и в боевом коде.

8. Недопустимо создавать разные тестовые методы для тех классов эквивалентности, которые могут быть проверены одним тестовым методом с использованием атрибутов `[TestCase]` и `[TestCaseSource]`.

9. Числа 42 и 123 в любых их модификациях расцениваются как "любое значение". Например: 42, 123, "name42", 123.0, 4242 и т.д.

10. Использование конструкций вида `It.IsXXX` (например, `It.IsAny`) библиотеки `Moq` допустимо только при вызове методов `Setup` и `Verify` этой библиотеки.

11. Для проверки утверждений (assertions) должна использоваться библиотека `FluentAssertions`.