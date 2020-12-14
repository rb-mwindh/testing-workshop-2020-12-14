# Angular Workshop: Testing / Unit Tests

- [Angular Workshop: Testing / Unit Tests](#angular-workshop-testing--unit-tests)
  - [Exercises](#exercises)
    - [1. Test Observable](#1-test-observable)
    - [2. Ensure that we have expects in each test.](#2-ensure-that-we-have-expects-in-each-test)
    - [3. Test Http Injection](#3-test-http-injection)
    - [4. Test Mock](#4-test-mock)
    - [5. No internal check against addresses](#5-no-internal-check-against-addresses)
    - [Bonus](#bonus)
      - [6. Http Interceptor](#6-http-interceptor)
      - [7. Show Test Coverage](#7-show-test-coverage)
      - [8. Enforce Test Coverage Thresholds](#8-enforce-test-coverage-thresholds)
      - [9. Street Names with Spaces](#9-street-names-with-spaces)
  - [Solution](#solution)
    - [1. Test Observable](#1-test-observable-1)
    - [3. Test Http Injection](#3-test-http-injection-1)
    - [4. Test Mock](#4-test-mock-1)
    - [5. No internal check against addresses](#5-no-internal-check-against-addresses-1)
    - [Bonus](#bonus-1)
      - [6. Http Interceptor](#6-http-interceptor-1)
      - [10. Street Names with Spaces](#10-street-names-with-spaces)

We already know, that we load the dataset via an http service. We have identified Nominatim for that. We know how the endpoint works and what it returns.

In this lab, we are going to upgrade our lookuper to use that service. It should also demonstrate how much we can achieve with plain unit tests.

## Exercises

### 1. Test Observable

Our address supplier and lookup method will need to return an Observable.

```typescript
it.each<any>([
  ['Domgasse 5, 1010 Wien', false],
  ['Domgasse 15, 1010 Wien', true]
])(
  'should require an addressSupplier as Observable:  %s',
  (query: string, expected: boolean, done: jest.DoneCallback) => {
    const addressSupplier = () => scheduled([['Domgasse 15, 1010 Wien']], asyncScheduler);

    const lookuper = new AddressLookuper(addressSupplier);
    lookuper.lookup(query).subscribe((result) => {
      expect(result).toBe(expected);
      done();
    });
  }
);
```

### 2. Ensure that we have expects in each test.

If you did not use the done callback pattern and did not follow strictly the first-fail approach, your expects might never run. We ensure that by adding an afterEach.

```typescript
afterEach(() => expect.hasAssertions());
```

### 3. Test Http Injection

Let's get serious.

We have identified nominatim as the geo service we want to use. It offers a free HTTP interface.

The address lookup will use Angular's `HttpClient` to fetch the adresses. We will have to mock that.

No need for additional tests. We just need to change the existing ones.

```typescript
it.each<any>([
  ['Domgasse 5, 1010 Wien', false],
  ['Domgasse 15, 1010 Wien', true]
])('should use HttpClient: %s', (query: string, expected: boolean, done: jest.DoneCallback) => {
  const lookuper = new AddressLookuper(httpClient);
  lookuper.lookup(query).subscribe((result) => {
    expect(result).toBe(expected);
    done();
  });
});
```

### 4. Test Mock

No refactoring this time. 😀

We want to make sure, that the right parameters are passed to the `HttpClient`. We create a further unit test to verify that.

```typescript
it('should call nominatim with the right parameters', () => {
  const get = jest.fn<Observable<string[]>, [string, { params: HttpParams }]>(() =>
    scheduled([[]], asyncScheduler)
  );
  const lookuper = new AddressLookuper(({ get } as unknown) as HttpClient);

  lookuper.lookup('Domgasse 5');

  const [url, { params }] = get.mock.calls[0];
  expect(url).toBe('https://nominatim.openstreetmap.org/search.php');
  const paramsMap = fromPairs(params.keys().map((key) => [key, params.get(key)]));
  expect(paramsMap).toMatchObject({
    format: 'jsonv2',
    q: 'Domgasse 5'
  });
});
```

### 5. No internal check against addresses

Since Nomination already checks for us if the address exists or not, we need to make sure that the lookuper just checks if an empty result is returned or not.

You can safely remove the test **should work with short input and long output**.

Make sure that this test is the first one in the file as it shows the default use case.

```typescript
it.each<any>([
  ['Domgasse 5, 1010 Wien', [], false],
  ['Domgasse 15, 1010 Wien', [''], true],
  ['Domgasse 1, 1010 Wien', [1], true],
  ['Domgasse 2, 1010 Wien', ['hit1', 'hit2'], true]
])(
  'should use HttpClient: %s',
  (query: string, addresses: string[], expected: boolean, done: jest.DoneCallback) => {
    const addresses$ = scheduled([addresses], asyncScheduler);
    const httpClient = ({
      get: jest.fn<Observable<string[]>, [string]>(() => addresses$)
    } as unknown) as HttpClient;

    const lookuper = new AddressLookuper(httpClient);

    lookuper.lookup(query).subscribe((result) => {
      expect(result).toBe(expected);
      done();
    });
  }
);
```

### Bonus

#### 6. Http Interceptor

- We create an Http Interceptor that adds a new http header when the nominatim endpoint is called. The header's key is `NominatimId` and the value `0129`.

```typescript
@Injectable({
  providedIn: 'root'
})
export class NominatimInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    if (req.url.match(/nominatim\.openstreetmap\.org/)) {
      return next.handle(req.clone({ setHeaders: { NominatimId: '0129' } }));
    }

    return next.handle(req);
  }
}
```

- Write two unit tests. One verifying that the header is set and one where it is not.

#### 7. Show Test Coverage

Run Jest with enabled test coverage and check its output. At the moment it should be 100%.

```bash
npx jest --collect-coverage
```

#### 8. Enforce Test Coverage Thresholds

Require a global test coverage rate of 100%. Add some function the address-lookuper, and add the following configuration to `jest.config.js`.

```js
module.exports = {
  // append this to the jest.config.js. Don't replace it
  coverageThreshold: {
    global: {
      branches: 100,
      functions: 100,
      lines: 100,
      statements: 100
    }
  }
};
```

Now run again jest with the collect coverage flag and make sure that it fails.

#### 9. Street Names with Spaces

We should allow street names with empty spaces. They are very common actually.

Additionally, since the parse is a very simple function. Just one input and output, we can replace all our tests with one single `it.each`.

```typescript
it.each([
  ['Domgasse 5', { street: 'Domgasse', streetNumber: '5' }],
  ['Domgasse 5, 1010 Wien', { street: 'Domgasse', streetNumber: '5', city: 'Wien', zip: '1010' }],
  [
    'Tiefer Graben 25, 1010 Wien',
    {
      street: 'Tiefer Graben',
      streetNumber: '25',
      zip: '1010',
      city: 'Wien'
    }
  ]
])('should parse %s', (query, expected) => {
  expect(parseAddress(query)).toEqual(expected);
});
```

## Solution

### 1. Test Observable

Both implementation and unit tests need to be refactored:

**address-lookuper.service.ts**

```typescript
export class AddressLookuper {
  constructor(private addressesSupplier: () => Observable<string[]>) {}

  lookup(query: string): Observable<boolean> {
    const address = parseAddress(query);
    if (!address.streetNumber) {
      throw new Error('Address without street number');
    }

    return this.addressesSupplier().pipe(
      map((addresses) => !!addresses.find((a) => a.startsWith(query)))
    );
  }
}
```

**addresss-lookuper.service.spec.ts**

```typescript
describe('AddressLookupService', () => {
  afterEach(() => expect.hasAssertions());
  const addressSupplier: () => Observable<string[]> = () =>
    scheduled([['Domgasse 15, 1010 Wien']], asyncScheduler);

  it('should throw an error if no street number is given', () => {
    const lookuper = new AddressLookuper(addressSupplier);

    expect(() => lookuper.lookup('Domgasse')).toThrow('Address without street number');
  });

  it.each<any>([
    ['Domgasse 5, 1010 Wien', false],
    ['Domgasse 15, 1010 Wien', true]
  ])(
    'should require an addressSupplier as Observable:  %s',
    (query: string, expected: boolean, done: jest.DoneCallback) => {
      const lookuper = new AddressLookuper(addressSupplier);
      lookuper.lookup(query).subscribe((result) => {
        expect(result).toBe(expected);
        done();
      });
    }
  );

  it('should work with short input and long output', (done) => {
    const lookuper = new AddressLookuper(addressSupplier);

    lookuper.lookup('Domgasse 15').subscribe((result) => {
      expect(result).toBe(true);
      done();
    });
  });
});
```

### 3. Test Http Injection

**addresss-lookuper.service.ts**

```typescript
export class AddressLookuper {
  constructor(private httpClient: HttpClient) {}

  lookup(query: string): Observable<boolean> {
    const address = parseAddress(query);
    if (!address.streetNumber) {
      throw new Error('Address without street number');
    }

    return this.httpClient
      .get<string[]>('')
      .pipe(map((addresses) => !!addresses.find((a) => a.startsWith(query))));
  }
}
```

**addresss-lookuper.service.spec.ts**

```typescript
describe('AddressLookupService', () => {
  afterEach(() => expect.hasAssertions());

  const addresses$ = scheduled([['Domgasse 15, 1010 Wien']], asyncScheduler);
  const httpClient = ({
    get: jest.fn<Observable<string[]>, [string]>(() => addresses$)
  } as unknown) as HttpClient;

  it('should throw an error if no street number is given', () => {
    const lookuper = new AddressLookuper(httpClient);

    expect(() => lookuper.lookup('Domgasse')).toThrow('Address without street number');
  });

  it.each<any>([
    ['Domgasse 5, 1010 Wien', false],
    ['Domgasse 15, 1010 Wien', true]
  ])('should use HttpClient:  %s', (query: string, expected: boolean, done: jest.DoneCallback) => {
    const lookuper = new AddressLookuper(httpClient);
    lookuper.lookup(query).subscribe((result) => {
      expect(result).toBe(expected);
      done();
    });
  });

  it('should work with short input and long output', (done) => {
    const lookuper = new AddressLookuper(httpClient);

    lookuper.lookup('Domgasse 15').subscribe((result) => {
      expect(result).toBe(true);
      done();
    });
  });
});
```

### 4. Test Mock

Extend the http call in the service with:

```typescript
return this.httpClient
  .get<string[]>('https://nominatim.openstreetmap.org/search.php', {
    params: new HttpParams().append('format', 'jsonv2').append('q', query)
  })
  .pipe(map((addresses) => !!addresses.find((a) => a.startsWith(query))));
```

### 5. No internal check against addresses

Change the pipe of the http call to following:

```typescript
return (
  this.httpClient
    //get call with options and url
    .pipe(map((addresses) => !!addresses.length))
);
```

### Bonus

#### 6. Http Interceptor

```typescript
describe('NominatimService', () => {
  it('should add the header', () => {
    const clonedReq = {} as HttpRequest<any>;
    const req = ({
      url: 'https://nominatim.openstreetmap.org/search',
      clone: jest.fn<HttpRequest<any>, [{ setHeaders: { [key: string]: string } }]>(
        ({ setHeaders }) => {
          const key = 'NominatimId';
          return setHeaders[key] === '0129' ? clonedReq : null;
        }
      )
    } as unknown) as HttpRequest<any>;
    const next = { handle: jest.fn() };

    new NominatimInterceptor().intercept(req, next);

    expect(next.handle).toBeCalledWith(clonedReq);
  });

  it('should not add the header', () => {
    const req = ({
      url: 'https://maps.google.com/search',
      clone: jest.fn<HttpRequest<any>, any>(() => null)
    } as unknown) as HttpRequest<any>;
    const next = { handle: jest.fn() };

    new NominatimInterceptor().intercept(req, next);

    expect(next.handle).toBeCalledWith(req);
  });
});
```

#### 10. Street Names with Spaces

```typescript
export function parseAddress(query: string): Address {
  const shortPattern = /^([\w\s]+)\s(\d+)$/;
  const longPattern = /^([\w\s]+)\s(\d+),\s(\d+)\s(\w+)$/;

  if (query.match(shortPattern)) {
    const [, street, streetNumber] = query.match(shortPattern);
    return { street, streetNumber };
  } else if (query.match(longPattern)) {
    const [, street, streetNumber, zip, city] = query.match(longPattern);
    return { street, streetNumber, zip, city };
  }
  return { street: '', streetNumber: '' };
}
```
