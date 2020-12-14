# Angular Workshop: Testing / End-to-End Tests

- [Angular Workshop: Testing / End-to-End Tests](#angular-workshop-testing--end-to-end-tests)
  - [Exercises](#exercises)
    - [1. Make a video recording](#1-make-a-video-recording)
    - [2. Run on Firefox](#2-run-on-firefox)
    - [3. Run in Different Resolutions](#3-run-in-different-resolutions)
    - [4. Page Object Model for Sidemenu](#4-page-object-model-for-sidemenu)
    - [5. Customer Form POM](#5-customer-form-pom)
    - [6. Stub endpoints](#6-stub-endpoints)
    - [7. Component E2E with Storybook](#7-component-e2e-with-storybook)
  - [Solutions](#solutions)
    - [3. Run in Different Resolutions](#3-run-in-different-resolutions-1)
    - [4. Page Object Model for Sidemenu](#4-page-object-model-for-sidemenu-1)
    - [5. Customer Form POM](#5-customer-form-pom-1)

## Exercises

### 1. Make a video recording

Easiest exercise of all. Just run `npx cypress run` and open `/cypress/videos` folder.

### 2. Run on Firefox

Also very easy. Just run `npx cypress run --browser firefox`.

It is very likely that your a single e2e fails. Ignore it for now and move on with the other tests.

### 3. Run in Different Resolutions

Cypress has pre-defined resolutions which can be run by `cy.viewport("preset")`. Run the "should count the entities" with following presets:

- ipad-2
- ipad-mini
- iphone-6
- samsung-s10

### 4. Page Object Model for Sidemenu

Create a Page Object Model for the sidemenu and use it in all the tests.

### 5. Customer Form POM

The `should add a new person` should be runnable by following code:

```typescript
it('should add a new person', () => {
  cy.visit('');
  sidemenu.click('Customers');
  cy.get('a').contains('Add Customer').click();
  customer.setFirstname('Tom');
  customer.setLastname('Lincoln');
  customer.setCountry('USA');
  customer.setBirthday(new Date(1995, 9, 12));
  customer.submit();

  cy.get('div').contains('Tom Lincoln');
});
```

Ensure that there exists a POM `customer` with the called methods.

### 6. Stub endpoints

```typescript
it.only('should stub the holidays', () => {
  cy.server();
  cy.route({
    method: 'GET',
    url: /holidays\.json$/,
    response: {
      holidays: [
        {
          title: 'Unicorn',
          teaser: 'You found One Piece',
          imageUrl: 'https://eternal-app.s3.eu-central-1.amazonaws.com/assets/OnePiece.png',
          description: 'Congratulations, you finally found Unicorn. Roger would be proud of you.'
        }
      ]
    }
  });
  cy.visit('');
  cy.get('mat-drawer a').contains('Holidays').click();
  cy.get('mat-card-title').contains('Unicorn');
});
```

### 7. Component E2E with Storybook

Let's combine Cypress and Storybook to E2E-test components in isolation.

Create a new file in **/cypress/integration/storybook.spec.ts**:

```typescript
describe('Component Tests', () => {
  it('should mock', () => {
    cy.server();
    cy.route({
      method: 'GET',
      url: /nominatim/,
      response: [[true]]
    });

    cy.visit('http://localhost:6006/iframe.html?id=eternal-requestinfo');
    cy.get('[data-test=address]').type('Winkelgasse 5');
    cy.get('[data-test=btn-search').click();
    cy.contains('Brochure sent');
  });

  it('Domgasse 5 succeeds', () => {
    cy.visit('http://localhost:6006/iframe.html?id=eternal-requestinfo');
    cy.get('[data-test=address]').type('Domgasse 5');
    cy.get('[data-test=btn-search').click();
    cy.contains('Brochure sent');
  });

  it('Domgasse does not succeed', () => {
    cy.visit('http://localhost:6006/iframe.html?id=eternal-requestinfo');
    cy.get('[data-test=address]').type('Domgasse');
    cy.get('[data-test=btn-search').click();
    cy.contains('Brochure sent');
  });
});
```

You will find out that the last test fails. Did we miss something? 🤔

There is no solution for this task. If time is short, see it as your homework. Good luck! 👍

## Solutions

### 3. Run in Different Resolutions

```typescript
context('entries count', () =>
  ['ipad-2', 'ipad-mini', 'iphone-6', 'samsung-s10'].forEach((resolution: ViewportPreset) => {
    it(`should count the entries for ${resolution}`, () => {
      cy.viewport(resolution);
      cy.visit('');
      sidemenu.click('Customers');
      cy.get('div.row:not(.header)').should('have.length', 22);
    });
  })
);
```

### 4. Page Object Model for Sidemenu

```typescript
class Sidemenu {
  click(name: 'Customers' | 'Holidays'): Chainable {
    return cy.get('mat-drawer a').contains(name).click();
  }
}

export const sidemenu = new Sidemenu();
```

### 5. Customer Form POM

**pom/customer.ts**

```typescript
import Chainable = Cypress.Chainable;

class Customer {
  setFirstname(firstname: string): Chainable {
    return cy.get('input:first').clear().type(firstname);
  }

  setLastname(lastname: string): Chainable {
    return cy.get('input:eq(1)').clear().type(lastname);
  }

  setCountry(country: string): Chainable {
    return cy.get('mat-select').click().get('mat-option').contains(country).click();
  }

  setBirthday(date: Date): Chainable {
    return cy.get('input:eq(2)').clear().type(Cypress.moment(date).format('DD.MM.yyyy'));
  }

  submit(): Chainable {
    return cy.get('button[type=submit]').click();
  }
}

export const customer = new Customer();
```
