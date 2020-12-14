# Angular Workshop: Testing / Unit Tests

- [Angular Workshop: Testing / Unit Tests](#angular-workshop-testing--unit-tests)
  - [Exercises](#exercises)
    - [1. Sanity Test](#1-sanity-test)
    - [2. Lookup Method](#2-lookup-method)
    - [3. Dummy Data](#3-dummy-data)
    - [4. Basic Validation](#4-basic-validation)
    - [5. The Need for a Parser](#5-the-need-for-a-parser)
    - [6. Support for German Address Format](#6-support-for-german-address-format)
    - [7. Provide DataSet for Addresses](#7-provide-dataset-for-addresses)
    - [8. Dealing with short and long notation](#8-dealing-with-short-and-long-notation)
    - [9. Address DataSet as function](#9-address-dataset-as-function)
    - [10. First Major Refactoring: Extracting Parser](#10-first-major-refactoring-extracting-parser)
    - [Bonus: 11. Be Creative](#bonus-11-be-creative)
  - [Solutions](#solutions)
    - [1. Sanity Test](#1-sanity-test-1)
    - [2. Lookup Method](#2-lookup-method-1)
    - [3. Dummy Data](#3-dummy-data-1)
    - [4. Basic Validation](#4-basic-validation-1)
    - [5. The Need for a Parser](#5-the-need-for-a-parser-1)
    - [6. Support for German Address Format](#6-support-for-german-address-format-1)
    - [7. Provide DataSet for Addresses](#7-provide-dataset-for-addresses-1)
    - [8. Dealing with short and long notation](#8-dealing-with-short-and-long-notation-1)
    - [9. Address DataSet as function](#9-address-dataset-as-function-1)
    - [10. First Major Refactoring: Extracting Parser](#10-first-major-refactoring-extracting-parser-1)

In this lab, you will write basic Unit Tests in a TDD-style. Only the code for the tests will be provided. You have to come up with the actual implementation.

We have the general requirement to verify the address our customers type in.

## Exercises

### 1. Sanity Test

First of all, we need to make sure that we have a class that will do the lookup.

Create a new file under **shared/address-lookuper.spec.ts**.

```typescript
describe('Address Lookuper', () => {
  it('should instantiate the lookuper', () => {
    const lookuper = new AddressLookuper();
    expect(lookuper).toBeDefined();
  });
});
```

Start jest in watch mode, make sure the test runs and fails. Then make it turn green.

### 2. Lookup Method

Just having a class will not be enough. We also require a lookup method that returns us a boolean when call it.

Add following test to the existing test suite:

```typescript
it('should provide a lookup method', () => {
  const lookuper = new AddressLookuper();
  const result = lookuper.lookup('');

  expect(result).toBe(true);
});
```

### 3. Dummy Data

The next step is that we want to make sure that it dosn't always return true. So it should only return true, if the address to lookup is only **Domgasse 5**.

```typescript
it('should return true on Domgasse 5', () => {
  const lookuper = new AddressLookuper();
  const result = lookuper.lookup('Domgasse 5');
  expect(result).toBe(true);
});

it('should return false on Domgasse 15', () => {
  const lookuper = new AddressLookuper();
  const result = lookuper.lookup('Domgasse 15');
  expect(result).toBe(false);
});
```

At this stage, the former test will fail. You will probably not need it.

### 4. Basic Validation

We need to make sure that we get at least a street and its number to search for. Otherwise an error should be thrown.

```typescript
it('should throw an error if no street number is given', () => {
  const lookuper = new AddressLookuper();

  expect(() => lookuper.lookup('Domgasse')).toThrow('Address without street number');
});
```

### 5. The Need for a Parser

We realise that we have to think a little more about validation and come up with the idea of a parser.

```typescript
it('should provide a parse method', () => {
  const lookuper = new AddressLookuper();
  const address = lookuper.parse('Domgasse 5');
  expect(address).toEqual({ street: 'Domgasse', streetNumber: '5' });
});
```

\* Hint: The regular expression for that is `/^([\w\s]+)\s(\d+)$/`.

### 6. Support for German Address Format

Our parser should also be able to parse full addresses with city and zip.

```typescript
it('should parse a German address format with city and zip', () => {
  const lookuper = new AddressLookuper();
  const address = lookuper.parse('Domgasse 5, 1010 Wien');
  expect(address).toEqual({
    street: 'Domgasse',
    streetNumber: '5',
    city: 'Wien',
    zip: '1010'
  });
});
```

\* Hint: The regular expression is `/^([\w\s]+)\s(\d+),\s(\d+)\s(\w+)$/`.

### 7. Provide DataSet for Addresses

It should be possible to pass on a set of address via the constructor.

```typescript
it('should allow addresses in constructor', () => {
  const addresses = ['Domgasse 15, 1010 Wien'];
  const lookuper = new AddressLookuper(addresses);

  expect(lookuper.lookup('Domgasse 5, 1010 Wien')).toBe(false);
  expect(lookuper.lookup('Domgasse 15, 1010 Wien')).toBe(true);
});
```

You will see that the former tests will fail. We don't want to introduce breaking changes. Make sure that all tests run fine.

### 8. Dealing with short and long notation

The lookuper should be able to provide the address even, if the dataset has long notations and the search short.

```typescript
it('should work with short input and long output', () => {
  const addresses = ['Domgasse 15, 1010 Wien'];
  const lookuper = new AddressLookuper(addresses);

  expect(lookuper.lookup('Domgasse 15')).toBe(true);
});
```

### 9. Address DataSet as function

We can expect in that the address dataset will be be fetched by some Geo service. So we should change the data type from an array to a function that returns the array.

We don't create new tests for this, but change the two existing.

```typescript
it('should allow addresses in constructor', () => {
  const addresses = ['Domgasse 15, 1010 Wien'];
  const lookuper = new AddressLookuper(() => addresses);

  expect(lookuper.lookup('Domgasse 5, 1010 Wien')).toBe(false);
  expect(lookuper.lookup('Domgasse 15, 1010 Wien')).toBe(true);
});

it('should work with short input and long output', () => {
  const addresses = ['Domgasse 15, 1010 Wien'];
  const lookuper = new AddressLookuper(() => addresses);

  expect(lookuper.lookup('Domgasse 15')).toBe(true);
});
```

### 10. First Major Refactoring: Extracting Parser

We realise that the latest changes did not affect the parser at all. The parser is a function on its own and is only a dependency for the actual lookup. We extract the parser implementation in its own file. The same goes for the test. We would end up having a **parse-address.ts** and a **parse-address.spec.ts**.

We also extract the `Address` interface in its own file. Parser and Lookup depend on it.

We end up having two small units with clear responsibilities and test coverage. We can build upon that.

### Bonus: 11. Be Creative

If you are already finished, you can improve the parser. I am sure that you can find some caess where the current code fill fail. Just think of cities that have a space in their name or a street number with a letter... Be creative!!!

## Solutions

### 1. Sanity Test

Just create a new file **address-lookuper.service.ts** in the same directory of the unit test and export a class.

```typescript
export class AddressLookuper {}
```

### 2. Lookup Method

Add following method:

```typescript
export class AddressLookuper {
  lookup(address: string): boolean {
    return true;
  }
}
```

### 3. Dummy Data

Let's keep it simple and just do a check against a static value:

```typescript
export class AddressLookuper {
  lookup(address: string): boolean {
    return address === 'Domgasse 5';
  }
}
```

### 4. Basic Validation

```typescript
export class AddressLookuper {
  lookup(address: string): boolean {
    if (!address.match(/\d+$/) {
      throw new Error("Address without street number");
    }
    return address === 'Domgasse 5';
  }
}
```

### 5. The Need for a Parser

```typescript
export interface Address {
  street: string;
  streetNumber: string;
}

export class AddressLookuper {
  // ... former code

  parse(address: string): Address {
    const match = address.match(/^([\w\s]+)\s(\d+)$/);
    if (match) {
      return { street: match[1], streetNumber: match[2] };
    }
    return null;
  }
}
```

### 6. Support for German Address Format

```typescript
export interface Address {
  street: string;
  streetNumber: string;
  zip?: string;
  city?: string;
}

export class AddressLookuper {
  lookup(address: string): boolean {
    if (this.parse(address) === null) {
      throw new Error('Address without street number');
    }

    return address === 'Domgasse 5';
  }

  parse(address: string): Address {
    const shortPattern = /^([\w\s]+)\s(\d+)$/;
    const longPattern = /^([\w\s]+)\s(\d+),\s(\d+)\s(\w+)$/;
    let match: string[] = address.match(shortPattern);

    if (match) {
      return { street: match[1], streetNumber: match[2] };
    } else {
      match = address.match(longPattern);
      if (match) {
        return {
          street: match[1],
          streetNumber: match[2],
          zip: match[3],
          city: match[4]
        };
      }
    }

    return null;
  }
}
```

### 7. Provide DataSet for Addresses

```typescript
export class AddressLookuper {
  constructor(private addresses = ['Domgasse 5']) {}

  lookup(address: string): boolean {
    if (this.parse(address) === null) {
      throw new Error('Address without street number');
    }

    return this.addresses[0].startsWith(address);
  }

  parse(address: string): Address {
    const shortPattern = /^([\w\s]+)\s(\d+)$/;
    const longPattern = /^([\w\s]+)\s(\d+),\s(\d+)\s(\w+)$/;
    let match: string[] = address.match(shortPattern);

    if (match) {
      return { street: match[1], streetNumber: match[2] };
    } else {
      match = address.match(longPattern);
      if (match) {
        return {
          street: match[1],
          streetNumber: match[2],
          zip: match[3],
          city: match[4]
        };
      }
    }

    return null;
  }

  // ... parse method
}
```

### 8. Dealing with short and long notation

```typescript
export class AddressLookuper {
  constructor(private addresses = ['Domgasse 5']) {}

  lookup(address: string): boolean {
    if (this.parse(address) === null) {
      throw new Error('Address without street number');
    }

    return this.addresses[0].startsWith(address);
  }

  // ... parse method
}
```

### 9. Address DataSet as function

```typescript
export class AddressLookuper {
  constructor(private addresses = () => ['Domgasse 5']) {}

  lookup(address: string): boolean {
    if (this.parse(address) === null) {
      throw new Error('Address without street number');
    }

    return this.addresses()[0].startsWith(address);
  }

  // ... parse method
}
```

### 10. First Major Refactoring: Extracting Parser

**address.ts**

```typescript
export interface Address {
  street: string;
  streetNumber: string;
  zip?: string;
  city?: string;
}
```

**parse-address.ts**

```typescript
import { Address } from './address';

export function parseAddress(query: string): Address {
  const shortPattern = /^(\w+)\s(\d+)$/;
  const longPattern = /^(\w+)\s(\d+),\s(\d+)\s(\w+)$/;

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

**address-lookuper.ts**

```typescript
import { parseAddress } from './parse-address';

export class AddressLookuper {
  constructor(private addresses: () => string[] = () => ['Domgasse 5']) {}

  lookup(query: string): boolean {
    const address = parseAddress(query);
    if (!address.streetNumber) {
      throw new Error('Address without street number');
    }

    return !!this.addresses().find((a) => a.startsWith(query));
  }
}
```
