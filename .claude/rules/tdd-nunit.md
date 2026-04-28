# Rules - TDD NUnit

## Principe
Tests ecrits AVANT le code de production.
Cycle : Red -> Green -> Refactor

## Structure obligatoire
[TestFixture]
public class {Service}Tests {
  private Mock<I{Dependency}> _mock;
  private {Service} _sut; // System Under Test

  [SetUp]
  public void SetUp() {
    _mock = new Mock<I{Dependency}>();
    _sut = new {Service}(_mock.Object);
  }

  [Test]
  public async Task {Method}_{Condition}_{ExpectedResult}() {
    // Arrange
    // Act
    // Assert
  }
}

## Nommage des tests
Methode_Condition_ResultatAttendu
Exemples :
- CreateAsync_WhenValidRequest_ShouldReturnResponse
- GetById_WhenNotFound_ShouldThrowNotFoundException
- Login_WhenInvalidPassword_ShouldThrowUnauthorized

## Couverture minimale
- Services    : 90%
- Validators  : 100%
- Controllers : 80%
- Repositories: 70%

## Packages
NUnit, NUnit3TestAdapter, Moq, FluentAssertions
Microsoft.EntityFrameworkCore.InMemory
Microsoft.AspNetCore.Mvc.Testing

## Interdit
- Tester plusieurs comportements dans un seul test
- Tests qui dependent les uns des autres
- Mocks inutiles
