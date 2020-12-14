# Angular Workshop: Testing / Component Tests - Functional

- [Angular Workshop: Testing / Component Tests - Functional](#angular-workshop-testing--component-tests---functional)
  - [Exercises](#exercises)
    - [1. Nested Components - Shallow](#1-nested-components---shallow)
    - [2. Nested Components - Declaration](#2-nested-components---declaration)
    - [3. Test Harnesses](#3-test-harnesses)
    - [4. Harness: Clearing the input field](#4-harness-clearing-the-input-field)
    - [5. Reusing Material Harnesses](#5-reusing-material-harnesses)
    - [Bonus](#bonus)
      - [6. HttpClientTestingModule / Detroit Style](#6-httpclienttestingmodule--detroit-style)
      - [7. Routing I](#7-routing-i)
      - [8. Routing II](#8-routing-ii)
      - [9. Upgrade all other Unit Tests to Harnesses](#9-upgrade-all-other-unit-tests-to-harnesses)
  - [Solutions](#solutions)
    - [2. Nested Components - Declaration](#2-nested-components---declaration-1)
    - [3. Test Harness](#3-test-harness)
    - [4. Harness: Clearing the input field](#4-harness-clearing-the-input-field-1)
    - [Bonus](#bonus-1)
      - [6. HttpClientTestingModule / Detroit Style](#6-httpclienttestingmodule--detroit-style-1)
      - [7. Routing I](#7-routing-i-1)
      - [8. Routing II](#8-routing-ii-1)
      - [9. Upgrade all other Unit Tests to Harnesses](#9-upgrade-all-other-unit-tests-to-harnesses-1)

## Exercises

### 1. Nested Components - Shallow

Let's pop up the request information screen. We will enhance the input and submit button visually with Angular Material.

1. Replace the form fields in `request-info.component.html` with the following.

```html
<p>
  <mat-form-field>
    <mat-label>Address</mat-label>
    <input data-test="address" formControlName="address" matInput />
    <mat-icon matSuffix>location_on</mat-icon>
    <mat-hint>Please enter your address</mat-hint>
  </mat-form-field>
</p>
<button color="primary" data-test="btn-search" mat-raised-button type="submit">Send</button>
```

2. Add a unit test with the prefix `only`.

```typescript
it.only('should mock components', () => {
  // tslint:disable-next-line:component-selector
  @Component({ selector: 'mat-form-field', template: '' })
  class MatFormField {}

  // tslint:disable-next-line:component-selector
  @Component({ selector: 'mat-hint', template: '' })
  class MatHint {}

  // tslint:disable-next-line:component-selector
  @Component({ selector: 'mat-icon', template: '' })
  class MatIcon {}

  // tslint:disable-next-line:component-selector
  @Component({ selector: 'mat-label', template: '' })
  class MatLabel {}

  const fixture = TestBed.configureTestingModule({
    declarations: [RequestInfoComponent, MatFormField, MatHint, MatIcon, MatLabel],
    imports: [ReactiveFormsModule],
    providers: [{ provide: AddressLookuper, useValue: null }]
  }).createComponent(RequestInfoComponent);

  fixture.detectChanges();
});
```

To verify that the test turns red, just don't add one of the modules to `imports` property.

### 2. Nested Components - Declaration

We would have to refactor the configuration of the TestModule for all our other unit tests. To avoid high cohesion, we did not use a common setup function, i.e. `beforeEach`. Instead, we will provide a customisable factory method for the test's module configuration.

**You don't have to do the refactoring for all tests**. Just add a skip to them like `it.skip(...`.

Apply the refactoring only to the following tests:

- should find an address
- should check both not-validation and validated

1. Create a new method inside `request-info.component.spec.ts`.

```typescript
const testModuleMetadata: TestModuleMetadata = {
  declarations: [RequestInfoComponent],
  imports: [
    MatButtonModule,
    MatFormFieldModule,
    MatIconModule,
    MatInputModule,
    NoopAnimationsModule,
    ReactiveFormsModule
  ],
  providers: [{ provide: AddressLookuper, useValue: null }]
};
```

2. Use this method in each unit test. The tests will keep isolation from each other but require less boilerplate code.

The tests that have the same configuration than the `testModuleMetadata` only need to have

```typescript
it('should check input fields have right values', () => {
  TestBed.configureTestingModule(testModuleMetadata);
  // ...
});
```

Customisation is very easy

```typescript
it('should fail on no input', () => {
  const fixture = TestBed.configureTestingModule({
    ...testModuleMetadata,
    providers: [{ provide: AddressLookuper, useValue: { lookup: () => of(false) } }]
  });
  // ...
});
```

If you are short on time, you can also directly copy the solution.

### 3. Test Harnesses

Create a harness for the address component.

**request-info.component.harness.ts**

```typescript
export class RequestComponentHarness extends ComponentHarness {
  static hostSelector = 'app-request-info';
  protected getTitle = this.locatorFor('h2');
  protected getInput = this.attrLocator('address');
  protected getButton = this.locatorFor('button[type=submit]');
  protected getLookupResult = this.attrLocator('lookup-result');

  async submit(): Promise<void> {
    const button = await this.getButton();
    return await button.click();
  }

  async writeAddress(address: string): Promise<void> {
    const input = await this.getInput();
    return await input.sendKeys(address);
  }

  async getResult(): Promise<string> {
    const p = await this.getLookupResult();
    return p.text();
  }

  protected attrLocator(tag: string): AsyncFactoryFn<TestElement> {
    return this.locatorFor(`[data-test=${tag}]`);
  }
}
```

Use the harness in the existing test "should find an address".

```typescript
it('should find an address', async () => {
  const lookuper = mockLookuper();
  const fixture = TestBed.configureTestingModule(
    createModuleMetadata({
      providers: [{ provide: AddressLookuper, useValue: lookuper }]
    })
  ).createComponent(RequestInfoComponent);

  const harness = await TestbedHarnessEnvironment.harnessForFixture(
    fixture,
    RequestInfoComponentHarness
  );

  // use harness here to set the query and submit

  return expect(harness.getResult()).resolves.toBe('Brochure sent');
});
```

### 4. Harness: Clearing the input field

Apply the harness to the `should check both not-validation and validated` unit test as well.

You will find an error, because the writeAddress only appends the value to the input field. This makes a query for `Diagon AlleyWinkelgasse` instead of just `Winkelgasse`.

Fix the `writeAddress` method in the harness.

### 5. Reusing Material Harnesses

Our harness can access its child components also via harnesses, if they provide them. As a matter of fact, we are using Angular Material which provides full support.

Inside your harness, change the selector for the input and button component. You just need to pass the Type inside the `locatorFor` method. That will `MatInputHarness` and `MatButtonHarness`.

You will also have to update the `writeAddress` method. Turns out that it supports the clearing of the fields internally.

### Bonus

#### 6. HttpClientTestingModule / Detroit Style

We will have to include the implementation of the `AddressLookuper` in our component test. The only dependency that gets mocked will be the `HttpClient`. It's an out-of-system dependency.

The test below is missing the crucial poin where the `HttpTestingController` steps in. Implement it that part.

```typescript
it('should use HttpClientTestingModule', () => {
  TestBed.configureTestingModule({
    declarations: [RequestInfoComponent],
    imports: [ReactiveFormsModule, HttpClientTestingModule],
    providers: [{ provide: ComponentFixtureAutoDetect, useValue: true }]
  });

  const httpController = TestBed.inject(HttpTestingController);
  const fixture = TestBed.createComponent(RequestInfoComponent);

  const input = fixture.debugElement.query(By.css('[data-test=address]'))
    .nativeElement as HTMLInputElement;
  input.value = 'Hauptstrasse 5';
  input.dispatchEvent(new Event('input'));

  const button = fixture.debugElement.query(By.css('[data-test=btn-search]'))
    .nativeElement as HTMLButtonElement;
  button.click();

  // use httpController here

  const lookupResult = fixture.debugElement.query(By.css('[data-test=lookup-result]'))
    .nativeElement as HTMLElement;
  expect(lookupResult.textContent.trim()).toBe('Brochure sent');
});
```

#### 7. Routing I

If the url `/home` is called, the user is redirected to `/`. Let's write a test in `app.component.spec.ts` to verify this.

```typescript
it('should check /home redirects to /', fakeAsync(() => {
  const fixture = TestBed.configureTestingModule({
    declarations: [HomeComponent, AppComponent, GdpcComponent],
    imports: [RouterTestingModule.withRoutes(APP_ROUTES)],
    schemas: [NO_ERRORS_SCHEMA]
  }).createComponent(AppComponent);

  const router = TestBed.inject(Router);
  const location = TestBed.inject(Location);
  fixture.ngZone.run(() => {
    router.initialNavigation();
    router.navigateByUrl('/home');
    tick();
    expect(location.path()).toBe('/');
  });
}));
```

#### 8. Routing II

We should ask our users for permission to store their addresses. This is done in a compnonent which redirects on approval.

1. Let's generate a proper gdpc component. We skip css and inline the template:

```bash
npx ng g c gdpc -it -is
```

2. Our component could look like this:

```typescript
import { Component } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  template: `
    <p>Do you give your consent that we can collect all your data?</p>
    <button mat-raised-button (click)="consent()">Of Course</button>&nbsp;
    <button mat-raised-button (click)="consent()">Do It</button>
  `
})
export class GdpcComponent {
  public constructor(private router: Router) {}

  consent(): void {
    this.router.navigateByUrl('/holidays');
  }
}
```

3. Add both the gdpc and the address-lookup component to our router in `app.routes.ts` and assign them the urls `/gdpc` and `/holidays`. This is also a good opportunity to import the `HttpClientModule` in your `app.module.ts`.

4. Write a test in London and one in the Detroit style in the generated `gdpc.spec.ts`. Verify that the redirection takes place.

#### 9. Upgrade all other Unit Tests to Harnesses

- You will have to find a way how to combine `async` with `fakeAsync`
- You will have extend the harness itself with further methods
- You might want to think about reducing the redundance of fetching the harness all the time.

## Solutions

### 2. Nested Components - Declaration

**request-info.component.spec.ts**

```typescript
describe('Test Address input', () => {
  afterEach(() => expect.hasAssertions());

  const mockLookuper = (results = [true]): Partial<AddressLookuper> => ({
    lookup: jest.fn<Observable<boolean>, [string]>(() => scheduled(results, asyncScheduler))
  });

  const createModuleMetadata = (
    customModuleMetadata: TestModuleMetadata = {}
  ): TestModuleMetadata => ({
    ...{
      declarations: [RequestInfoComponent],
      imports: [
        MatButtonModule,
        MatFormFieldModule,
        MatIconModule,
        MatInputModule,
        NoopAnimationsModule,
        ReactiveFormsModule
      ],
      providers: [{ provide: AddressLookuper, useValue: null }]
    },
    ...customModuleMetadata
  });

  it('should check if search calls service', () => {
    const lookuper = mockLookuper();
    const component = new RequestInfoComponent(
      lookuper as AddressLookuper,
      ({
        group: jest.fn()
      } as unknown) as FormBuilder
    );
    component.ngOnInit();
    component.formGroup = {
      value: { address: '' }
    } as FormGroup;
    component.lookupResult$.subscribe();

    component.search();

    expect(lookuper.lookup).toBeCalled();
  });

  it('should check if right parameters are passed to lookup service', () => {
    const lookuper = mockLookuper();
    const component = new RequestInfoComponent(
      lookuper as AddressLookuper,
      ({
        group: jest.fn()
      } as unknown) as FormBuilder
    );
    component.ngOnInit();
    component.formGroup = {
      value: { address: 'Lindenstrasse 5, de' }
    } as FormGroup;
    component.lookupResult$.subscribe();

    component.search();

    expect(lookuper.lookup).toHaveBeenCalledWith('Lindenstrasse 5, de');
  });

  it('should check the title', () => {
    const fixture = TestBed.configureTestingModule(createModuleMetadata()).createComponent(
      RequestInfoComponent
    );
    fixture.detectChanges();

    const title = fixture.debugElement.query(By.css('h2')).nativeElement as HTMLElement;
    expect(title.textContent).toBe('Request More Information');

    fixture.componentInstance.title = 'Test Title';
    fixture.detectChanges();
    expect(title.textContent).toBe('Test Title');
  });

  it('should check input fields have right values', () => {
    TestBed.configureTestingModule(createModuleMetadata());
    const fixture = TestBed.createComponent(RequestInfoComponent);
    const component = fixture.componentInstance;
    fixture.detectChanges();

    component.formGroup.patchValue({
      address: 'Hauptstraße 5'
    });
    const address = fixture.debugElement.query(By.css('input[data-test=address]'))
      .nativeElement as HTMLInputElement;

    expect(address.value).toBe('Hauptstraße 5');
  });

  it('should fail on no input', fakeAsync(() => {
    const fixture = TestBed.configureTestingModule(
      createModuleMetadata({
        providers: [{ provide: AddressLookuper, useValue: mockLookuper([false]) }]
      })
    ).createComponent(RequestInfoComponent);

    fixture.detectChanges();
    fixture.componentInstance.search();
    tick();
    fixture.detectChanges();
    const lookupResult = fixture.debugElement.query(By.css('[data-test=lookup-result]'))
      .nativeElement as HTMLElement;

    expect(lookupResult.textContent.trim()).toBe('Address not found');
  }));

  it('should trigger search on click', fakeAsync(() => {
    const lookuper = mockLookuper();
    const fixture = TestBed.configureTestingModule(
      createModuleMetadata({
        providers: [{ provide: AddressLookuper, useValue: lookuper }]
      })
    ).createComponent(RequestInfoComponent);

    fixture.detectChanges();
    const button = fixture.debugElement.query(By.css('[data-test=btn-search]'))
      .nativeElement as HTMLButtonElement;
    button.click();
    tick();

    expect(lookuper.lookup).toHaveBeenCalled();
  }));

  it('should find an address', fakeAsync(() => {
    const lookuper = mockLookuper();
    const fixture = TestBed.configureTestingModule(
      createModuleMetadata({
        providers: [{ provide: AddressLookuper, useValue: lookuper }]
      })
    ).createComponent(RequestInfoComponent);
    fixture.detectChanges();
    const input = fixture.debugElement.query(By.css('[data-test=address]'))
      .nativeElement as HTMLInputElement;
    input.value = 'Hauptstrasse';
    input.dispatchEvent(new Event('input'));
    tick();
    const button = fixture.debugElement.query(By.css('[data-test=btn-search]'))
      .nativeElement as HTMLButtonElement;
    button.click();
    tick();
    fixture.detectChanges();

    const lookupResult = fixture.debugElement.query(By.css('[data-test=lookup-result]'))
      .nativeElement as HTMLElement;

    expect(lookupResult.textContent.trim()).toBe('Brochure sent');
  }));

  it('should check both not-validation and validated', fakeAsync(() => {
    const lookup = (address) => scheduled([address === 'Winkelgasse'], asyncScheduler);

    const fixture = TestBed.configureTestingModule(
      createModuleMetadata({
        providers: [
          { provide: AddressLookuper, useValue: { lookup } },
          { provide: ComponentFixtureAutoDetect, useValue: true }
        ]
      })
    ).createComponent(RequestInfoComponent);

    const input = fixture.debugElement.query(By.css('[data-test=address]'))
      .nativeElement as HTMLInputElement;
    input.value = 'Diagon Alley';
    input.dispatchEvent(new Event('input'));

    const button = fixture.debugElement.query(By.css('[data-test=btn-search]'))
      .nativeElement as HTMLButtonElement;
    button.click();
    tick();

    const lookupResult = fixture.debugElement.query(By.css('[data-test=lookup-result]'))
      .nativeElement as HTMLElement;

    expect(lookupResult.textContent.trim()).toBe('Address not found');
    input.value = 'Winkelgasse';
    input.dispatchEvent(new Event('input'));
    button.click();
    tick();

    expect(lookupResult.textContent.trim()).toBe('Brochure sent');
  }));

  it('should check the snapshot', () => {
    TestBed.configureTestingModule(createModuleMetadata());
    const fixture = TestBed.createComponent(RequestInfoComponent);

    expect(fixture).toMatchSnapshot();
  });

  it('should catch an error', fakeAsync(() => {
    const lookup = () => throwError(new Error('nominatim not available'), asyncScheduler);

    TestBed.configureTestingModule(
      createModuleMetadata({
        providers: [{ provide: AddressLookuper, useValue: { lookup } }]
      })
    );

    const fixture = TestBed.createComponent(RequestInfoComponent);
    fixture.detectChanges();
    const input = fixture.debugElement.query(By.css('[data-test=address]'))
      .nativeElement as HTMLInputElement;
    input.value = 'Domgasse 5';
    input.dispatchEvent(new Event('input'));
    fixture.componentInstance.search();
    tick();
    fixture.detectChanges();

    const lookupResult = fixture.debugElement.query(By.css('[data-test=lookup-result]'))
      .nativeElement as HTMLElement;

    expect(lookupResult.textContent.trim()).toBe('Address not found');
  }));

  it('should use HttpClientTestingModule', () => {
    const moduleMetaData = createModuleMetadata({
      providers: [{ provide: ComponentFixtureAutoDetect, useValue: true }]
    });
    moduleMetaData.imports.push(HttpClientTestingModule);
    TestBed.configureTestingModule(moduleMetaData);

    const httpController = TestBed.inject(HttpTestingController);
    const fixture = TestBed.createComponent(RequestInfoComponent);

    const input = fixture.debugElement.query(By.css('[data-test=address]'))
      .nativeElement as HTMLInputElement;
    input.value = 'Hauptstrasse 5';
    input.dispatchEvent(new Event('input'));

    const button = fixture.debugElement.query(By.css('[data-test=btn-search]'))
      .nativeElement as HTMLButtonElement;
    button.click();

    const [request] = httpController.match((req) => !!req.url.match(/nominatim/));
    request.flush([true]);
    fixture.detectChanges();

    const lookupResult = fixture.debugElement.query(By.css('[data-test=lookup-result]'))
      .nativeElement as HTMLElement;
    expect(lookupResult.textContent.trim()).toBe('Brochure sent');
  });

  it('should mock components', () => {
    // tslint:disable-next-line:component-selector
    @Component({ selector: 'mat-form-field', template: '' })
    class MatFormField {}

    // tslint:disable-next-line:component-selector
    @Component({ selector: 'mat-hint', template: '' })
    class MatHint {}

    // tslint:disable-next-line:component-selector
    @Component({ selector: 'mat-icon', template: '' })
    class MatIcon {}

    // tslint:disable-next-line:component-selector
    @Component({ selector: 'mat-label', template: '' })
    class MatLabel {}

    const fixture = TestBed.configureTestingModule({
      declarations: [RequestInfoComponent, MatFormField, MatHint, MatIcon, MatLabel],
      imports: [ReactiveFormsModule],
      providers: [{ provide: AddressLookuper, useValue: null }]
    }).createComponent(RequestInfoComponent);

    fixture.detectChanges();

    expect(true).toBe(true);
  });
});
```

### 3. Test Harness

With test harness, the search is very simple. You just need:

```typescript
await harness.writeAddress('Domgasse 5');
await harness.submit();
```

### 4. Harness: Clearing the input field

**address.component.harness.ts**

```typescript
export class RequestInfoComponentHarness extends ComponentHarness {
  // ...

  async writeAddress(address: string): Promise<void> {
    const input = await this.getInput();
    await input.clear();
    return await input.sendKeys(address);
  }
}
```

### Bonus

#### 6. HttpClientTestingModule / Detroit Style

The instrumentation of the `HttpTestingController` could look like this:

```typescript
const [request] = httpController.match((req) => !!req.url.match(/nominatim/));
request.flush([true]);
fixture.detectChanges();
```

#### 7. Routing I

```typescript
it('should check confirmation in London-style', () => {
  const router = mock<Router>({
    navigateByUrl: jest.fn()
  });
  new GdpcComponent(router).consent();

  expect(router.navigateByUrl).toBeCalledWith('/holidays');
});
```

#### 8. Routing II

```typescript
it('should make sure that gdpc confirmation redirects to holidays', fakeAsync(() => {
  const fixture = TestBed.configureTestingModule({
    declarations: [GdpcComponent, RequestInfoComponent],
    imports: [
      RouterTestingModule.withRoutes(
        APP_ROUTES.filter((route) => ['gdpc', 'holidays'].includes(route.path))
      )
    ],
    schemas: [NO_ERRORS_SCHEMA]
  }).createComponent(GdpcComponent);

  const location = TestBed.inject(Location);
  fixture.detectChanges();
  fixture.ngZone.run(() => {
    const consentButton = fixture.debugElement.query(By.css('[data-test=btn-search]'))
      .nativeElement as HTMLButtonElement;
    consentButton.click();
    fixture.detectChanges();
    tick();
    expect(location.path()).toBe('/holidays');
  });
}));
```

**Bonus**: Think about an alternative where the schemas property is not required.

#### 9. Upgrade all other Unit Tests to Harnesses

**request-info.component.spec.ts**

```typescript
describe('Test Address input', () => {
  afterEach(() => expect.hasAssertions());

  const mockLookuper = (results = [true]): Partial<AddressLookuper> => ({
    lookup: jest.fn<Observable<boolean>, [string]>(() => scheduled(results, asyncScheduler))
  });

  const defaultModuleMetadata = {
    declarations: [RequestInfoComponent],
    imports: [
      MatButtonModule,
      MatFormFieldModule,
      MatIconModule,
      MatInputModule,
      NoopAnimationsModule,
      ReactiveFormsModule
    ],
    providers: [{ provide: AddressLookuper, useValue: null }]
  };

  const createFixtureAndHarness = async (customModuleMetadata: TestModuleMetadata = {}) => {
    TestBed.configureTestingModule({
      ...defaultModuleMetadata,
      ...customModuleMetadata
    });
    const fixture = TestBed.createComponent(RequestInfoComponent);
    const harness = await TestbedHarnessEnvironment.harnessForFixture(
      fixture,
      RequestInfoComponentHarness
    );

    return { fixture, harness };
  };

  it('should check if search calls service', () => {
    const lookuper = mockLookuper();
    const component = new RequestInfoComponent(
      lookuper as AddressLookuper,
      ({
        group: jest.fn()
      } as unknown) as FormBuilder
    );
    component.ngOnInit();
    component.formGroup = {
      value: { address: '' }
    } as FormGroup;
    component.lookupResult$.subscribe();

    component.search();

    expect(lookuper.lookup).toBeCalled();
  });

  it('should check if right parameters are passed to lookup service', () => {
    const lookuper = mockLookuper();
    const component = new RequestInfoComponent(
      lookuper as AddressLookuper,
      ({
        group: jest.fn()
      } as unknown) as FormBuilder
    );
    component.ngOnInit();
    component.formGroup = {
      value: { address: 'Lindenstrasse 5, de' }
    } as FormGroup;
    component.lookupResult$.subscribe();

    component.search();

    expect(lookuper.lookup).toHaveBeenCalledWith('Lindenstrasse 5, de');
  });

  it('should check the title', async () => {
    const { fixture, harness } = await createFixtureAndHarness();

    const initialValue = await harness.getTitleText();
    expect(initialValue).toBe('Request More Information');

    fixture.componentInstance.title = 'Test Title';
    return expect(harness.getTitleText()).resolves.toBe('Test Title');
  });

  it('should check input fields have right values', async () => {
    const { fixture, harness } = await createFixtureAndHarness();

    fixture.detectChanges();
    fixture.componentInstance.formGroup.patchValue({
      address: 'Hauptstraße 5'
    });

    return expect(harness.getAddressValue()).resolves.toBe('Hauptstraße 5');
  });

  it('should fail on no input', fakeAsync(async () => {
    const { fixture, harness } = await createFixtureAndHarness({
      providers: [{ provide: AddressLookuper, useValue: mockLookuper([false]) }]
    });

    fixture.componentInstance.search();
    tick();

    return expect(harness.getResult()).resolves.toBe('Address not found');
  }));

  it('should trigger search on click', async () => {
    const lookuper = mockLookuper();
    const { harness } = await createFixtureAndHarness({
      providers: [{ provide: AddressLookuper, useValue: lookuper }]
    });
    await harness.submit();

    expect(lookuper.lookup).toHaveBeenCalled();
  });

  it('should find an address', async () => {
    const lookuper = mockLookuper();
    const { harness } = await createFixtureAndHarness({
      providers: [{ provide: AddressLookuper, useValue: lookuper }]
    });

    await harness.writeAddress('Domgasse 5');
    await harness.submit();

    return expect(harness.getResult()).resolves.toBe('Brochure sent');
  });

  it('should check both not-validation and validated', async () => {
    const lookup = (address) => {
      return scheduled([address === 'Winkelgasse'], asyncScheduler);
    };
    const { harness } = await createFixtureAndHarness({
      providers: [
        { provide: AddressLookuper, useValue: { lookup } },
        { provide: ComponentFixtureAutoDetect, useValue: true }
      ]
    });

    await harness.writeAddress('Diagon Alley');
    await harness.submit();
    const result = await harness.getResult();
    expect(result).toBe('Address not found');

    await harness.writeAddress('Winkelgasse');
    await harness.submit();
    return expect(harness.getResult()).resolves.toBe('Brochure sent');
  });

  it('should check the snapshot', async () => {
    const { fixture } = await createFixtureAndHarness();
    fixture.detectChanges();

    expect(fixture).toMatchSnapshot();
  });

  it('should catch an error', async () => {
    const lookup = () => throwError(new Error('nominatim not available'), asyncScheduler);
    const { harness } = await createFixtureAndHarness({
      providers: [{ provide: AddressLookuper, useValue: { lookup } }]
    });

    await harness.writeAddress('Domgasse 5');
    await harness.submit();
    return expect(harness.getResult()).resolves.toBe('Address not found');
  });

  it('should use HttpClientTestingModule', async () => {
    const imports = [...defaultModuleMetadata.imports, HttpClientTestingModule];
    const { harness } = await createFixtureAndHarness({
      imports,
      providers: [{ provide: ComponentFixtureAutoDetect, useValue: true }]
    });
    const httpController = TestBed.inject(HttpTestingController);

    await harness.writeAddress('Hauptstrasse 5');
    await harness.submit();

    const [request] = httpController.match((req) => !!req.url.match(/nominatim/));
    request.flush([true]);
    return expect(harness.getResult()).resolves.toBe('Brochure sent');
  });

  it('should mock components', () => {
    // tslint:disable-next-line:component-selector
    @Component({ selector: 'mat-form-field', template: '' })
    class MatFormField {}

    // tslint:disable-next-line:component-selector
    @Component({ selector: 'mat-hint', template: '' })
    class MatHint {}

    // tslint:disable-next-line:component-selector
    @Component({ selector: 'mat-icon', template: '' })
    class MatIcon {}

    // tslint:disable-next-line:component-selector
    @Component({ selector: 'mat-label', template: '' })
    class MatLabel {}

    const fixture = TestBed.configureTestingModule({
      declarations: [RequestInfoComponent, MatFormField, MatHint, MatIcon, MatLabel],
      imports: [ReactiveFormsModule],
      providers: [{ provide: AddressLookuper, useValue: null }]
    }).createComponent(RequestInfoComponent);

    fixture.detectChanges();

    expect(true).toBe(true);
  });
});
```
