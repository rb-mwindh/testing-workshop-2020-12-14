# Angular Workshop: Testing / Component Tests - Functional

- [Angular Workshop: Testing / Component Tests - Functional](#angular-workshop-testing--component-tests---functional)
  - [Exercises](#exercises)
    - [1. Querying the DOM](#1-querying-the-dom)
    - [2. DOM & Mocking Services](#2-dom--mocking-services)
    - [3. DOM Interaction](#3-dom-interaction)
    - [4. Full Lookup](#4-full-lookup)
    - [5. Snapshotting](#5-snapshotting)
    - [Bonus](#bonus)
      - [6. Mock HttpClient](#6-mock-httpclient)
      - [7. Error Handler](#7-error-handler)
      - [8. Double Search](#8-double-search)
      - [9. Component Test as Unit Test](#9-component-test-as-unit-test)
  - [Solutions](#solutions)
    - [1. Querying the DOM](#1-querying-the-dom-1)
    - [2. DOM and Mocking Services](#2-dom-and-mocking-services)
    - [4. Full Lookup](#4-full-lookup-1)
    - [Bonus](#bonus-1)
      - [6. Mock HttpClient](#6-mock-httpclient-1)
      - [7. Error Handler](#7-error-handler-1)
      - [8. Double Search](#8-double-search-1)

In this lab, we will integrate the address lookuper into the request-info component.

**holidays/request-info/request-info.component.ts**:

Inject the addressLookuper service into the component:

```typescript
export class RequestInfoComponent implements OnInit {
  // ...

  constructor(private lookuper: AddressLookuper, private formBuilder: FormBuilder) {}

  // ...
}
```

The `lookupResult$` observable should use the new serivce:

```typescript
export class RequestInfoComponent implements OnInit {
  // ...

  ngOnInit(): void {
    // ...

    this.lookupResult$ = this.submitter$.pipe(
      switchMap(() =>
        this.lookuper.lookup(this.formGroup.value.address).pipe(catchError(() => of(false)))
      ),
      map((found) => (found ? 'Brochure sent' : 'Address not found'))
    );
  }

  // ..
}
```

## Exercises

### 1. Querying the DOM

Write two tests that query the DOM.

One test should check if the title can be changed and the other one should set a default value for the form field.

This one misses change detection. Find the right place and add it.

```typescript
it('should check the title', () => {
  const fixture = TestBed.configureTestingModule({
    declarations: [RequestInfoComponent],
    imports: [ReactiveFormsModule],
    providers: [{ provide: AddressLookuper, useValue: null }]
  }).createComponent(RequestInfoComponent);
  fixture.detectChanges();

  const title = fixture.debugElement.query(By.css('h2')).nativeElement as HTMLElement;
  expect(title.textContent).toBe('Request More Information');

  fixture.componentInstance.title = 'Test Title';

  expect(title.textContent).toBe('Test Title');
});
```

You will have to find out the right css selector for the input field and also add the change detection.

```typescript
it('should check input fields have right values', () => {
  TestBed.configureTestingModule({
    declarations: [RequestInfoComponent],
    imports: [ReactiveFormsModule],
    providers: [{ provide: AddressLookuper, useValue: null }]
  });
  const fixture = TestBed.createComponent(RequestInfoComponent);
  const component = fixture.componentInstance;
  fixture.detectChanges();

  component.formGroup.patchValue({
    address: 'Hauptstraße 5'
  });
  const address = fixture.debugElement.query(By.css('add-your-selector-here'))
    .nativeElement as HTMLInputElement;

  expect(address.value).toBe('Hauptstraße 5');
});
```

### 2. DOM & Mocking Services

Mock the Lookup service so that it returns no results. Verify that it shows the right message.

```typescript
it('should fail on no input', fakeAsync(() => {
  const fixture = TestBed.configureTestingModule({
    declarations: [RequestInfoComponent],
    imports: [ReactiveFormsModule],
    providers: [{ provide: AddressLookuper, useValue: mockLookuper([false]) }]
  }).createComponent(RequestInfoComponent);

  fixture.detectChanges();
  fixture.componentInstance.search();
  tick();
  fixture.detectChanges();
  const lookupResult = fixture.debugElement.query(By.css('[data-test=lookup-result]'))
    .nativeElement as HTMLElement;
}));
```

### 3. DOM Interaction

Interact with the DOM by clicking on the submit button.

```typescript
it('should trigger search on click', fakeAsync(() => {
  const lookuper = mockLookuper();
  const fixture = TestBed.configureTestingModule({
    declarations: [RequestInfoComponent],
    imports: [ReactiveFormsModule],
    providers: [{ provide: AddressLookuper, useValue: lookuper }]
  }).createComponent(RequestInfoComponent);

  fixture.detectChanges();
  const button = fixture.debugElement.query(By.css('[data-test=btn-search]'))
    .nativeElement as HTMLButtonElement;
  button.click();
  tick();

  expect(lookuper.lookup).toHaveBeenCalled();
}));
```

### 4. Full Lookup

Write a test where the lookup method returns an observable which runs asynchronously.

Find out where to add the `tick()` and `fixture.detectChanges()` methods.

You will have to use the **fakeAsync** and **tick** methods.

```typescript
it('should find an address', fakeAsync(() => {
  const lookuper = mockLookuper();
  const fixture = TestBed.configureTestingModule({
    declarations: [RequestInfoComponent],
    imports: [ReactiveFormsModule],
    providers: [{ provide: AddressLookuper, useValue: lookuper }]
  }).createComponent(RequestInfoComponent);

  const input = fixture.debugElement.query(By.css('[data-test=address]'))
    .nativeElement as HTMLInputElement;
  input.value = 'Hauptstrasse';
  input.dispatchEvent(new Event('input'));

  const button = fixture.debugElement.query(By.css('[data-test=btn-search]'))
    .nativeElement as HTMLButtonElement;
  button.click();

  const lookupResult = fixture.debugElement.query(By.css('[data-test=lookup-result]'))
    .nativeElement as HTMLElement;

  expect(lookupResult.textContent.trim()).toBe('Brochure sent');
}));
```

### 5. Snapshotting

Write a test with Jest's snapshotting feature. Play a little bit with the interactive update feature by changing some small things in the template and try the interactive update via the CLI.

```typescript
it('should check the snapshot', () => {
  TestBed.configureTestingModule({
    declarations: [RequestInfoComponent],
    imports: [ReactiveFormsModule],
    providers: [{ provide: AddressLookuper, useValue: null }]
  });
  const fixture = TestBed.createComponent(RequestInfoComponent);
  expect(fixture).toMatchSnapshot();
});
```

### Bonus

#### 6. Mock HttpClient

In this test, we want to include the implementation of the AddressLookuper and mock the HttpClient instead.

Create a new file **shared/nominatim-interceptor.spec.ts**:

```typescript
it('should mock the HttpClient', fakeAsync(() => {
  // insert an appropriate mock for the HttpClient here

  TestBed.configureTestingModule({
    declarations: [RequestInfoComponent],
    imports: [ReactiveFormsModule],
    providers: [
      { provide: ComponentFixtureAutoDetect, useValue: true },
      { provide: HttpClient, useValue: http }
    ]
  });

  const fixture = TestBed.createComponent(RequestInfoComponent);

  const input = fixture.debugElement.query(By.css('[data-test=address]'))
    .nativeElement as HTMLInputElement;
  input.value = 'Domgasse 5';
  input.dispatchEvent(new Event('input'));

  const button = fixture.debugElement.query(By.css('[data-test=btn-search]'))
    .nativeElement as HTMLButtonElement;
  button.click();
  tick();

  const lookupResult = fixture.debugElement.query(By.css('[data-test=lookup-result]'))
    .nativeElement as HTMLElement;
  expect(lookupResult.textContent.trim()).toBe('Brochure sent');
}));
```

#### 7. Error Handler

When the lookuper is throwing an exception, we want to show that the invalidation failed. The unit test below fails due to a buggy implementation in the component (not the unit test) itself. Fix the bug.

```typescript
it('should catch an error', fakeAsync(() => {
  const lookup = () => throwError(new Error('nominatim not available'), asyncScheduler);
  TestBed.configureTestingModule({
    declarations: [RequestInfoComponent],
    imports: [ReactiveFormsModule],
    providers: [{ provide: AddressLookuper, useValue: { lookup } }]
  });

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
```

#### 8. Double Search

Write a test where you do two search queries sequentially. The first one should end with an invalid address, the second with a valid one.

#### 9. Component Test as Unit Test

We can still run some basic unit tests against the component.

First task is to create a factory method for the `AddressLookuper`.

```typescript
const mockLookuper = (results = [true]): Partial<AddressLookuper> => ({
  lookup: jest.fn<Observable<boolean>, [string]>(() => scheduled(results, asyncScheduler))
});
```

Since we will be dealing with asynchronity, we enable the check for assertions:

```typescript
afterEach(() => expect.hasAssertions());
```

Write a test that verifies that the lookuper service is actually called and another one that makes sure that it is called with the right parameters.

```typescript
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
```

## Solutions

### 1. Querying the DOM

- `fixture.detectChanges()` has to be called after
- The css selector is `input[data-test=address]`

### 2. DOM and Mocking Services

The assertion should be:

```typescript
expect(lookupResult.textContent.trim()).toBe('Address not found');
```

### 4. Full Lookup

```typescript
it('should find an address', fakeAsync(() => {
  const lookuper = mockLookuper();
  const fixture = TestBed.configureTestingModule({
    declarations: [RequestInfoComponent],
    imports: [ReactiveFormsModule],
    providers: [{ provide: AddressLookuper, useValue: lookuper }]
  }).createComponent(RequestInfoComponent);

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
```

### Bonus

#### 6. Mock HttpClient

A mock which also checks against the input:

```typescript
const http = {
  get: jest.fn<Observable<string[]>, [string, { params: HttpParams }]>((url, { params }) => {
    let returnValue = [[]];
    if (
      url === 'https://nominatim.openstreetmap.org/search.php' &&
      params.get('q') === 'Domgasse 5'
    ) {
      returnValue = [['Domgasse 5, 1010 Wien']];
    }
    return scheduled(returnValue, asyncScheduler);
  })
};
```

#### 7. Error Handler

The `lookupResult$` does not catch the error from the lookuper service. In **address.component.ts**, change it to:

```typescript
this.lookupResult$ = this.submitter$.pipe(
  switchMap(() =>
    this.lookuper.lookup(this.formGroup.value.address).pipe(catchError(() => of(false)))
  ),
  map((found) => (found ? 'Brochure sent' : 'Address not found'))
);
```

#### 8. Double Search

```typescript
it('should check both not-validation and validated', fakeAsync(() => {
  const lookup = (address) => scheduled([address === 'Winkelgasse'], asyncScheduler);

  const fixture = TestBed.configureTestingModule({
    declarations: [RequestInfoComponent],
    imports: [ReactiveFormsModule],
    providers: [
      { provide: AddressLookuper, useValue: { lookup } },
      { provide: ComponentFixtureAutoDetect, useValue: true }
    ]
  }).createComponent(RequestInfoComponent);

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
```
