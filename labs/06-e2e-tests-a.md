# Angular Workshop: Testing / End-to-End Tests

- [Angular Workshop: Testing / End-to-End Tests](#angular-workshop-testing--end-to-end-tests)
  - [Exercises](#exercises)
    - [1. Setup](#1-setup)
    - [2. Add a sanity check](#2-add-a-sanity-check)
    - [3. Make an implicit Subject Assertion](#3-make-an-implicit-subject-assertion)
    - [4. Test via an explicit Subject Assertion](#4-test-via-an-explicit-subject-assertion)
    - [5. Count the listed customers](#5-count-the-listed-customers)
    - [6. Edit a customer's firstname](#6-edit-a-customers-firstname)
    - [7. Add a new customer](#7-add-a-new-customer)
    - [8. Request More Information for Angkor Wat](#8-request-more-information-for-angkor-wat)
    - [Bonus](#bonus)
      - [9. Ensure that the web application loads within a second](#9-ensure-that-the-web-application-loads-within-a-second)
      - [10. Split up tests](#10-split-up-tests)
  - [Solutions](#solutions)
    - [8. Request More Information for Angkor Wat](#8-request-more-information-for-angkor-wat-1)

## Exercises

### 1. Setup

1. Run Cypress:

```bash
npx cypress open
```

2. Add a **tsconfig.json** to generated **cypress** folder:

```json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["es5", "dom"],
    "types": ["cypress"]
  },
  "include": ["**/*.ts"]
}
```

3. Add the baseUrl to your **cypress.json**:

```json
{ "baseUrl": "http://localhost:4200" }
```

### 2. Add a sanity check

Create a new `e2e.spec.ts` file and add the following first test:

```typescript
it('should do a sanity check', () => {
  cy.visit('');
});
```

### 3. Make an implicit Subject Assertion

We check if the mat-drawer tag has a link which contains the text Holidays

```typescript
it('should do an implicit subject assertion', () => {
  cy.visit('');
  cy.get('mat-drawer a:first').should('have.text', 'Holidays');
});
```

### 4. Test via an explicit Subject Assertion

We check again the link for text Holidays. This time, we also check against the class and the link itself.

```typescript
it('should do an explicit subject assertion', () => {
  cy.visit('');
  cy.get('mat-drawer a:first').should(($a) => {
    expect($a).to.have.text('Holidays');
    expect($a).to.have.class('mat-raised-button');
    expect($a).to.have.attr('href', '/holidays');
  });
});
```

### 5. Count the listed customers

Go to the customers list and count the rows:

```typescript
it('should count the entries', () => {
  cy.visit('');
  cy.get('mat-drawer a').contains('Customers').click();
  cy.get('div.row:not(.header)').should('have.length', 22);
});
```

### 6. Edit a customer's firstname

Rename Latitia to Laetitia via the form

```typescript
it('should rename Latitia to Laetitia', () => {
  cy.visit('');
  cy.get('mat-drawer a').contains('Customers').click();
  cy.get('div').contains('Latitia Bellitissa').siblings('.edit').click();
  cy.get('input:first').clear().type('Laetitia');
  cy.get('button[type=submit]').click();

  cy.get('div').contains('Bellitissa').should('have.text', 'Laetitia Bellitissa');
});
```

### 7. Add a new customer

Add a new customer and check if he appears on the listing page.

```typescript
it.only('should add a new person', () => {
  cy.visit('');
  cy.get('mat-drawer a').contains('Customers').click();
  cy.get('mat-drawer-content a').contains('Add Customer').click();
  cy.get('input:first').type('Tom');
  cy.get('input:eq(1)').type('Lincoln');
  cy.get('mat-select').click().get('mat-option').contains('USA').click();
  cy.get('input:eq(2)').type('12.10.1995');
  cy.get('button[type=submit]').click();
  cy.get('div').contains('Tom Lincoln');
});
```

### 8. Request More Information for Angkor Wat

Write a test that clicks on the Angkor Wat holiday's "More Information" button and verifies that the input "Domgasse 5" shows "Brochure sent".

### Bonus

#### 9. Ensure that the web application loads within a second

```typescript
it('should load page below 1 second', () => {
  cy.visit('/', {
    onBeforeLoad: (win) => {
      win.performance.mark('start-loading');
    },
    onLoad: (win) => {
      win.performance.mark('end-loading');
    }
  })
    .its('performance')
    .then((p) => {
      p.measure('pageLoad', 'start-loading', 'end-loading');
      const measure = p.getEntriesByName('pageLoad')[0];
      expect(measure.duration).to.be.most(1000);
    });
});
```

#### 10. Split up tests

Split up the test into 3 different spec files:

- customers.spec.ts
- holidays.spec.ts
- misc.spec.ts

## Solutions

### 8. Request More Information for Angkor Wat

```typescript
it('should request more information for Holiday Angkor Wat', () => {
  cy.visit('');
  cy.get('mat-drawer a').contains('Holidays').click();
  cy.get('app-holiday-card').contains('Angkor Wat').parents('app-holiday-card').find('a').click();
  cy.get('[data-test=address').type('Domgasse 5');
  cy.get('[data-test=btn-search]').click();
  cy.get('[data-test=lookup-result]').contains('Brochure sent');
});
```
