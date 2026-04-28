# TDD avec NUnit et Moq

## Règle fondamentale
Les tests sont écrits AVANT le code de production.
Cycle : Red → Green → Refactor

## Structure d'un test
[TestFixture]
public class UserServiceTests
{
    private Mock<IUnitOfWork> _uowMock;
    private Mock<IMapper> _mapperMock;
    private UserService _sut;  // System Under Test

    [SetUp]
    public void SetUp()
    {
        _uowMock = new Mock<IUnitOfWork>();
        _mapperMock = new Mock<IMapper>();
        _sut = new UserService(_uowMock.Object, _mapperMock.Object);
    }

    [Test]
    public async Task CreateAsync_WhenValidRequest_ShouldReturnUserResponse()
    {
        // Arrange
        var request = new CreateUserRequest("John", "Doe", "john@test.com", "Pass123!");
        var user = new User { Id = 1, Email = "john@test.com" };
        var response = new UserResponse(1, "John", "Doe", "john@test.com", DateTime.UtcNow);

        _mapperMock.Setup(m => m.Map<User>(request)).Returns(user);
        _uowMock.Setup(u => u.Users.AddAsync(user)).ReturnsAsync(user);
        _mapperMock.Setup(m => m.Map<UserResponse>(user)).Returns(response);

        // Act
        var result = await _sut.CreateAsync(request);

        // Assert
        Assert.That(result, Is.Not.Null);
        Assert.That(result.Email, Is.EqualTo("john@test.com"));
        _uowMock.Verify(u => u.SaveChangesAsync(), Times.Once);
    }

    [Test]
    public async Task GetByIdAsync_WhenUserNotFound_ShouldThrowNotFoundException()
    {
        // Arrange
        _uowMock.Setup(u => u.Users.GetByIdAsync(999)).ReturnsAsync((User?)null);

        // Act & Assert
        Assert.ThrowsAsync<NotFoundException>(
            async () => await _sut.GetByIdAsync(999));
    }
}

## Nommage des tests
Méthode_Condition_Résultat_Attendu
Exemples :
- CreateAsync_WhenValidRequest_ShouldReturnUserResponse
- GetById_WhenUserNotFound_ShouldThrowNotFoundException
- Update_WhenUserExists_ShouldUpdateAndSave

## Couverture minimale
- Services    : 90%
- Validators  : 100%
- Controllers : 80% (tests d'intégration)
- Repositories: 70% (tests d'intégration avec DB en mémoire)

## Packages NuGet pour les tests
<PackageReference Include="NUnit" Version="4.*" />
<PackageReference Include="NUnit3TestAdapter" Version="4.*" />
<PackageReference Include="Moq" Version="4.*" />
<PackageReference Include="FluentAssertions" Version="6.*" />
<PackageReference Include="Microsoft.EntityFrameworkCore.InMemory" Version="8.*" />
<PackageReference Include="Microsoft.AspNetCore.Mvc.Testing" Version="8.*" />
