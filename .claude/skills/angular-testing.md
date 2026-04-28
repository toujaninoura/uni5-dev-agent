---
name: angular-testing
description: Patterns Jasmine + Karma - unit tests composants et services
---

# Skill - Angular Testing Patterns

## Test service avec HttpClientTestingModule
describe("{Service}", () => {
  let service: {Service};
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [{Service}]
    });
    service = TestBed.inject({Service});
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => httpMock.verify());

  it("should fetch {entities}", () => {
    service.get{Entities}().subscribe(result => {
      expect(result.length).toBeGreaterThan(0);
    });
    const req = httpMock.expectOne("/api/v1/{entities}");
    expect(req.request.method).toBe("GET");
    req.flush(mock{Entities});
  });
});

## Test composant avec TestBed
describe("{Component}", () => {
  let component: {Component};
  let fixture: ComponentFixture<{Component}>;
  let {service}Spy: jasmine.SpyObj<{Service}>;

  beforeEach(async () => {
    {service}Spy = jasmine.createSpyObj("{Service}", ["{method}"]);

    await TestBed.configureTestingModule({
      imports: [{Component}],
      providers: [
        { provide: {Service}, useValue: {service}Spy }
      ]
    }).compileComponents();

    fixture = TestBed.createComponent({Component});
    component = fixture.componentInstance;
  });

  it("should create", () => {
    expect(component).toBeTruthy();
  });

  it("should display {data}", () => {
    component.{data} = mock{Data};
    fixture.detectChanges();
    const items = fixture.debugElement.queryAll(By.css(".{selector}"));
    expect(items.length).toBe(mock{Data}.length);
  });

  it("should call service on init", () => {
    {service}Spy.{method}.and.returnValue(of(mock{Data}));
    fixture.detectChanges();
    expect({service}Spy.{method}).toHaveBeenCalled();
  });
});

## Test guard
describe("AuthGuard", () => {
  it("should allow when authenticated", () => {
    authServiceSpy.isAuthenticated.and.returnValue(true);
    expect(guard.canActivate()).toBeTrue();
  });

  it("should redirect when not authenticated", () => {
    authServiceSpy.isAuthenticated.and.returnValue(false);
    expect(guard.canActivate()).toBeFalse();
  });
});

## Commandes
ng test --watch=false
ng test --watch=false --code-coverage
ng test --include="**/auth/*.spec.ts"
