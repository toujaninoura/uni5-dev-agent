---
name: dotnet-testing
description: Patterns NUnit + Moq - unit tests, integration tests, WebApplicationFactory
---

# Skill - .NET Testing Patterns

## Structure test unitaire
[TestFixture]
public class {Service}Tests {
  private Mock<IUnitOfWork> _uowMock;
  private Mock<IMapper> _mapperMock;
  private {Service} _sut;

  [SetUp]
  public void SetUp() {
    _uowMock = new Mock<IUnitOfWork>();
    _mapperMock = new Mock<IMapper>();
    _sut = new {Service}(_uowMock.Object, _mapperMock.Object);
  }

  [Test]
  public async Task {Method}_{Condition}_{Expected}() {
    // Arrange
    var request = new {Request}(...);
    _uowMock.Setup(u => u.{Repo}.{Method}(...)).ReturnsAsync(...);

    // Act
    var result = await _sut.{Method}(request);

    // Assert
    Assert.That(result, Is.Not.Null);
    Assert.That(result.{Property}, Is.EqualTo(...));
    _uowMock.Verify(u => u.SaveChangesAsync(), Times.Once);
  }

  [Test]
  public async Task {Method}_{Condition}_ShouldThrow{Exception}() {
    // Arrange
    _uowMock.Setup(u => u.{Repo}.GetByIdAsync(999)).ReturnsAsync((Entity)null);

    // Act & Assert
    Assert.ThrowsAsync<NotFoundException>(
      async () => await _sut.GetByIdAsync(999));
  }
}

## Test integration WebApplicationFactory
public class {Controller}IntegrationTests : IClassFixture<WebApplicationFactory<Program>> {
  private readonly HttpClient _client;

  public {Controller}IntegrationTests(WebApplicationFactory<Program> factory) {
    _client = factory.WithWebHostBuilder(builder => {
      builder.ConfigureServices(services => {
        services.AddDbContext<AppDbContext>(options =>
          options.UseInMemoryDatabase("TestDb"));
      });
    }).CreateClient();
  }

  [Test]
  public async Task GET_{Endpoint}_Returns200() {
    var response = await _client.GetAsync("/api/v1/{endpoint}");
    Assert.That(response.StatusCode, Is.EqualTo(HttpStatusCode.OK));
  }

  [Test]
  public async Task POST_{Endpoint}_Returns401_WhenNoToken() {
    var response = await _client.PostAsJsonAsync("/api/v1/{endpoint}", new{});
    Assert.That(response.StatusCode, Is.EqualTo(HttpStatusCode.Unauthorized));
  }
}

## Commandes
dotnet test
dotnet test --no-build
dotnet test --filter "ClassName={Service}Tests"
dotnet test --collect:"XPlat Code Coverage"
