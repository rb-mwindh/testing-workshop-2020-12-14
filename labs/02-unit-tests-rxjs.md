# Angular Workshop: Testing / Unit Tests - RxJs

- [Angular Workshop: Testing / Unit Tests - RxJs](#angular-workshop-testing--unit-tests---rxjs)
  - [Exercises](#exercises)
    - [1. Multiply by 2](#1-multiply-by-2)
    - [2. Unsubscription](#2-unsubscription)
    - [3. combineLatest](#3-combinelatest)
    - [4. Query Counter](#4-query-counter)
    - [5. Custom Operator: filterTruthy](#5-custom-operator-filtertruthy)
    - [6. Asynchronicity](#6-asynchronicity)
    - [7. Proofing Higher Order Observable](#7-proofing-higher-order-observable)
    - [8. ngrx Reducer](#8-ngrx-reducer)
    - [9. ngrx Selectors](#9-ngrx-selectors)
    - [10. ngrx Effect](#10-ngrx-effect)
  - [Solution](#solution)
    - [1. Multiply by 2](#1-multiply-by-2-1)
    - [2. Unsubscription](#2-unsubscription-1)
    - [3. combineLatest](#3-combinelatest-1)
    - [4. Query Counter](#4-query-counter-1)
    - [5. Custom Operator: filterTruthy](#5-custom-operator-filtertruthy-1)
    - [6. Asynchronity](#6-asynchronity)
    - [7. Proofing Higher Order Observable](#7-proofing-higher-order-observable-1)
      - [7.1 switchMap](#71-switchmap)
      - [7.1 mergeMap](#71-mergemap)
      - [7.1 concatMap](#71-concatmap)
      - [7.1 exhaustMap](#71-exhaustmap)
    - [8. Reducer](#8-reducer)

If not explicitly excluded, all test should be done with the `rxjs-marbles` library.

## Exercises

### 1. Multiply by 2

The observable below emits numbers. Create a set of pipe operator(s) where each number is multiplied and verify that via a unit test.

```typescript
m.cold('--a-b-c', { a: 2, b: 10, c: 25 });
```

### 2. Unsubscription

```typescript
m.cold('abcde', {
  a: 'Hauptstraße 3',
  b: '',
  c: 'Domgasse 5',
  d: 'Kärntnerring 12',
  e: 'Praterstern 2'
});
```

The observable should complete, when `Domgasse 5` is emitted.

**Hint**: `first` or `takeWhile` made be useful.

### 3. combineLatest

Check if the `combineLatest` operator works as specified.

We have two observable that, where we want to add their emitted values.

```typescript
const s1 = m.cold('-a', { a: 1 });
const s2 = m.cold('a-', { a: 2 });
```

If combineLatest works as expected, the new observable should only emit one value which is 3.

### 4. Query Counter

```typescript
let queryCounter = 0;
const source = m.cold('d 2ms p 2ms h', {
  d: 'Domgasse 5',
  p: 'Praterstern',
  h: 'Herrengasse 12'
});
```

We should filter only queries where a streetNumber is passed. Per valid query, we increate the `queryCounter`. We have 2 valid queries.

**Hint**: You have to implement that via a sideffect.

### 5. Custom Operator: filterTruthy

Create an operator filterTruthy that does what its name says.

```typescript
m.cold('abcdef', {
  a: null,
  b: undefined,
  c: false,
  d: '',
  e: 0,
  f: 1
});
```

Only the number 1 will get through the filter.

**Hint**:

```typescript
const filterTruthy = (observable: Observable<unknown>) =>
  observable.pipe(...);
```

You see how to create a new operator above. The filtering should be do inside the pipe.

### 6. Asynchronicity

For this exercise, you CANNOT use `rxjs-marbles`.

```typescript
const lookuper = (query) =>
  new Promise<boolean>((resolve) => {
    resolve(query === 'Domgasse 5');
  });

const source = of('Praterstern', 'Domgasse 5');
const hits = [];
```

Create an observable from `source` that is using the `lookuper` method. It should store the resolved values of `lookuper` into hits. Verify that `hits` has following value: `[false, true]`.

**Hint**: A higher order operator like `concatMap` or `mergeMap` works also against Promises.

### 7. Proofing Higher Order Observable

Write tests for all 4 major higher order observable and proof their behaviour:

- switchMap
- mergeMap
- concatMap
- exhaustMap

The test structure could look like this

```typescript
test(
  'switchMap',
  marbles((m) => {
    const source: Observable<string> = m.cold('pd', { p: 'Praterstern', d: 'Domgasse 5' });
    const dest: Observable<boolean> = source.pipe(
      switchMap((query) => m.cold('---b', { b: query === 'Domgasse 5' }))
    );

    m.expect(dest).toBeObservable('insert your marble');
  })
);
```

### 8. ngrx Reducer

Let's test the reducer. Very easy actually.

```typescript
it('should add the holidays on findHolidaySuccess', () => {
  const holidays = [
    { id: 1, title: 'Pyramids' },
    { id: 2, title: 'Tower Bridge' }
  ] as Holiday[];

  const state = holidaysReducer(
    { holidays: [] },
    holidaysActions.findHolidaysSuccess({ holidays })
  );

  expect(state).toEqual({ holidays });
});
```

Write two other tests, one testing that the action `findHoliday` doesn't change anyhting and another that a `findHolidaySuccess` overwrites pre-existing holidays.

### 9. ngrx Selectors

Even easier and simpler than to test the reducer.

```typescript
it('should select the holidays', () => {
  const state: HolidaysState = {
    holidays: [
      { id: 1, title: 'Pyramids' },
      { id: 2, title: 'Tower Bridge' }
    ] as Holiday[]
  };

  expect(fromHolidays.get.projector(state)).toEqual([
    { id: 1, title: 'Pyramids' },
    { id: 2, title: 'Tower Bridge' }
  ]);
});
```

### 10. ngrx Effect

In order to test our effects, we create a new file **holiday.effects.spec.ts**. The properties in an effect are nothing more than normal Observables and we can test also test them as we do with all others.

```typescript
it(
  'should test find$',
  marbles((m) => {
    const httpClient = {
      get: () => m.cold('---a', { a: { holidays: [{ id: 1 }] } })
    };
    const actions$ = m.cold('a', { a: holidaysActions.findHolidays() });

    const effect = new HolidaysEffects(actions$, (httpClient as unknown) as HttpClient);
    m.expect(effect.find$).toBeObservable('3ms a', {
      a: holidaysActions.findHolidaysSuccess({ holidays: [({ id: 1 } as unknown) as Holiday] })
    });
  })
);
```

## Solution

### 1. Multiply by 2

```typescript
test(
  'multiply by 2',
  marbles((m) => {
    const source = m.cold('--a-b-c', { a: 2, b: 10, c: 25 });

    const destination = source.pipe(map((n) => n * 2));

    m.expect(destination).toBeObservable('--x-y-z', {
      x: 4,
      y: 20,
      z: 50
    });
  })
);
```

### 2. Unsubscription

```typescript
test(
  'unsubscription',
  marbles((m) => {
    const source = m.cold('abcde', {
      a: 'Hauptstraße 3',
      b: '',
      c: 'Domgasse 5',
      d: 'Kärntnerring 12',
      e: 'Praterstern 2'
    });

    const destination = source.pipe(
      filter((address) => address === 'Domgasse 5'),
      first()
    );

    m.expect(destination).toBeObservable('--(a|)', { a: 'Domgasse 5' });
  })
);
```

### 3. combineLatest

```typescript
test(
  'combine Latest',
  marbles((m) => {
    const s1 = m.cold('-a', { a: 1 });
    const s2 = m.cold('a-', { a: 2 });

    const dest = combineLatest([s1, s2]).pipe(map(([a, b]) => a + b));
    m.expect(dest).toBeObservable('-a', { a: 3 });
  })
);
```

### 4. Query Counter

```typescript
test(
  'query counter',
  marbles((m) => {
    let searchCounter = 0;
    const source = m.cold('d 2ms p 2ms h', {
      d: 'Domgasse 5',
      p: 'Praterstern',
      h: 'Herrengasse 12'
    });
    const destination = source.pipe(
      map((address) => address.match(/(.+)\s(\d+)$/)),
      filter((matcher) => !!matcher),
      tap(() => searchCounter++),
      map(([, street, streetNumber]) => ({
        street,
        streetNumber
      }))
    );
    m.expect(destination).toBeObservable('d 5ms h', {
      d: { street: 'Domgasse', streetNumber: '5' },
      h: { street: 'Herrengasse', streetNumber: '12' }
    });
    m.flush();
    expect(searchCounter).toBe(2);
  })
);
```

### 5. Custom Operator: filterTruthy

```typescript
test(
  'operator filterTruthy',
  marbles((m) => {
    const source = m.cold('abcdef', {
      a: null,
      b: undefined,
      c: false,
      d: '',
      e: 0,
      f: 1
    });

    const filterTruthy = (observable: Observable<unknown>) =>
      observable.pipe(filter((data) => !!data));
    const destination = source.pipe(filterTruthy);
    m.expect(destination).toBeObservable('-----f', { f: 1 });
  })
);
```

### 6. Asynchronity

```typescript
test('asynchronity', (done) => {
  const lookuper = (query) =>
    new Promise<boolean>((resolve) => {
      resolve(query === 'Domgasse 5');
    });

  const source = of('Praterstern', 'Domgasse 5');
  const hits = [];
  source.pipe(concatMap((query) => lookuper(query))).subscribe((isHit) => {
    hits.push(isHit);

    if (hits.length === 2) {
      expect(hits).toEqual([false, true]);
      done();
    }
  });
});
```

### 7. Proofing Higher Order Observable

#### 7.1 switchMap

```typescript
test(
  'switchMap',
  marbles((m) => {
    const source: Observable<string> = m.cold('pd', { p: 'Praterstern', d: 'Domgasse 5' });
    const dest: Observable<boolean> = source.pipe(
      switchMap((query) => m.cold('---b', { b: query === 'Domgasse 5' }))
    );

    m.expect(dest).toBeObservable('----t', { t: true });
  })
);
```

#### 7.1 mergeMap

```typescript
test(
  'mergeMap',
  marbles((m) => {
    const source: Observable<string> = m.cold('pd', { p: 'Praterstern', d: 'Domgasse 5' });
    const dest: Observable<boolean> = source.pipe(
      mergeMap((query) => m.cold('---b', { b: query === 'Domgasse 5' }))
    );

    m.expect(dest).toBeObservable('---ft', { f: false, t: true });
  })
);
```

#### 7.1 concatMap

```typescript
test(
  'concatMap',
  marbles((m) => {
    const source: Observable<string> = m.cold('pd', { p: 'Praterstern', d: 'Domgasse 5' });
    const dest: Observable<boolean> = source.pipe(
      concatMap((query) => m.cold('---b|', { b: query === 'Domgasse 5' }))
    );

    m.expect(dest).toBeObservable('---f---t', { f: false, t: true });
  })
);
```

#### 7.1 exhaustMap

```typescript
test(
  'exhaustMap',
  marbles((m) => {
    const source: Observable<string> = m.cold('pd', { p: 'Praterstern', d: 'Domgasse 5' });
    const dest: Observable<boolean> = source.pipe(
      exhaustMap((query) => m.cold('---b', { b: query === 'Domgasse 5' }))
    );

    m.expect(dest).toBeObservable('---f', { f: false, t: true });
  })
);
```

### 8. Reducer

```typescript
it('should be no state change on findHoliday', () => {
  const state = holidaysReducer({ holidays: [] }, holidaysActions.findHolidays());

  expect(state).toEqual({ holidays: [] });
});
```

```typescript
it('should replace existing holidays on findHolidaySuccess', () => {
  const initialState = { holidays: [{ id: 1, title: 'Pyramids' }] as Holiday[] };

  const state = holidaysReducer(
    initialState,
    holidaysActions.findHolidaysSuccess({
      holidays: [{ id: 2, title: 'Tower Bridge' } as Holiday]
    })
  );

  expect(state.holidays).toEqual([{ id: 2, title: 'Tower Bridge' }]);
});
```
