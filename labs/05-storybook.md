# Angular Workshop: Testing / End-to-End Tests

- [Angular Workshop: Testing / End-to-End Tests](#angular-workshop-testing--end-to-end-tests)
  - [Exercises](#exercises)
    - [1. Setup a Holiday Card Component](#1-setup-a-holiday-card-component)
    - [2. Run Storybook](#2-run-storybook)
    - [3. Adding HolidayCard to Storybook](#3-adding-holidaycard-to-storybook)
    - [4. Add RequestInfo Story](#4-add-requestinfo-story)
    - [5. Snapshot all Stories](#5-snapshot-all-stories)
    - [Bonus:](#bonus)
      - [5. Holiday Card with Controls](#5-holiday-card-with-controls)
  - [Solutions](#solutions)
    - [4. Add RequestInfo Story](#4-add-requestinfo-story-1)

In this lab, we are going to improve the display of our holiday offers. To make them better "storybookable", we need to stick to the container / presentation strategy.

So let's first create a presentation component that represents a holiday card.

## Exercises

### 1. Setup a Holiday Card Component

1. Generate Component Holiday Card

```
npx ng g c holidays/holidayCard
```

2. Extract the presentational parts from holidays.component to the new one:

**holiday-card.component.html**

```html
<mat-card>
  <mat-card-header>
    <mat-card-title>{{ holiday.title }}</mat-card-title>
    <mat-card-subtitle>{{ holiday.teaser }}</mat-card-subtitle>
  </mat-card-header>
  <img [alt]="holiday.title" [src]="holiday.imageUrl" mat-card-image />
  <mat-card-content> {{ holiday.description }} </mat-card-content>
  <mat-card-actions>
    <a [routerLink]="['.', 'request-info', holiday.id]" mat-raised-button>More Information</a>
  </mat-card-actions>
</mat-card>
```

**holiday-card.component.ts**

```typescript
@Component({
  selector: 'app-holiday-card',
  templateUrl: './holiday-card.component.html',
  styleUrls: ['./holiday-card.component.scss']
})
export class HolidayCardComponent {
  @Input() holiday: Holiday;
}
```

**holiday-card.component.scss**

```scss
mat-card {
  mat-card-actions {
    text-align: center;
  }
}
```

3. Adapt the holidays

**holidays.component.html**

```html
<div class="container">
  <ng-container *ngrxLet="holidays$; let holidays">
    <app-holiday-card *ngFor="let holiday of holidays" [holiday]="holiday"></app-holiday-card>
  </ng-container>
</div>
```

**holidays.component.scss**

```scss
div.container {
  display: flex;
  justify-content: space-evenly;
  flex-wrap: wrap;

  app-holiday-card {
    display: flex;
    flex: 1 0 232px;
    margin: 2em;
  }
}
```

4. Verify the holidays page is still showing the holidays by running the vr tests. It will fail. Verify that the visual changes are only minimal. Then update the screenshots.

### 2. Run Storybook

1. Storybook is very easy to setup. Just run

```bash
npx start-storybook -p 6006
```

2. A browser window should open automatically where you see already some examples. Pick the button story. Play a little bit with it and also study the source code in the `src/stories` folder.

3. Optionally hide the pre-generated stories by replacing changing the second value of the stories properties in `.storybook/main.js` to `../src/app/**/*.stories.@(js|jsx|ts|tsx)`.

### 3. Adding HolidayCard to Storybook

1. Create the stories file.

**holidays/holiday-card.component.stories.ts**

```typescript
export default {
  title: 'Eternal/HolidayCard',
  component: HolidayCardComponent,
  decorators: [moduleMetadata({ imports: [MatButtonModule, MatCardModule, RouterTestingModule] })],
  argTypes: { holiday: { name: 'Holiday', description: 'JSON object for the Holiday' } }
} as Meta;

const Template: Story<HolidayCardComponent> = (args: HolidayCardComponent) => {
  return {
    component: HolidayCardComponent,
    props: args
  };
};

export const Default = Template.bind({});
Default.args = {
  holiday: {
    id: 1,
    title: 'Holiday',
    description: 'A holiday',
    imageUrl: 'https://eternal-app.s3.eu-central-1.amazonaws.com/assets/AngkorWatSmall.jpg'
  }
};
```

2. Set maximum width

Storybook reveels, that the component spans over the whole width. That's too much.

Limit the maximum width to 400px in **holiday-card.component.scss**.

```scss
mat-card {
  max-width: 400px;

  mat-card-actions {
    text-align: center;
  }
}
```

### 4. Add RequestInfo Story

Create the for the RequestInfo component on your own. Since it does not have any `@Input`, the code should be much shorter.

### 5. Snapshot all Stories

We can now snapshot all our stories via Puppeteer with one single test:

**/src/stories/storyshot.spec.js**

```typescript
import initStoryshots from '@storybook/addon-storyshots';
import { imageSnapshot } from '@storybook/addon-storyshots-puppeteer';

initStoryshots({ suite: 'Image storyshots', test: imageSnapshot() });
```

### Bonus:

#### 5. Holiday Card with Controls

This is a variation of the holiday card where we provide controls so that the user could change the values:

**holidays/holiday-card.component.custom.stories.ts**

```typescript
const urls = [
  'https://eternal-app.s3.eu-central-1.amazonaws.com/assets/AngkorWatSmall.jpg',
  'https://eternal-app.s3.eu-central-1.amazonaws.com/assets/AustrianRushSmall.jpg',
  'https://eternal-app.s3.eu-central-1.amazonaws.com/assets/ChinaSmall.jpg'
];

export default {
  title: 'Eternal/Customisable HolidayCard',
  component: HolidayCardComponent,
  decorators: [moduleMetadata({ imports: [MatButtonModule, MatCardModule, RouterTestingModule] })],
  argTypes: {
    title: { control: 'text', defaultValue: 'THE eternal Holiday' },
    description: { control: 'text', defaultValue: 'Very nice Holiday' },
    teaser: { control: 'text', defaultValue: 'Buy it' },
    imageUrl: {
      control: {
        type: 'select',
        options: urls
      },
      defaultValue: urls[0]
    },
    holiday: {
      table: {
        disable: true
      }
    }
  }
} as Meta;

type HolidayCardArgs = HolidayCardComponent & {
  title: string;
  description: string;
  teaser: string;
  imageUrl: string;
};

const Template: Story<HolidayCardComponent> = (args: HolidayCardArgs) => {
  return {
    component: HolidayCardComponent,
    props: {
      ...args,
      holiday: {
        title: args.title,
        description: args.description,
        teaser: args.teaser,
        imageUrl: args.imageUrl
      }
    }
  };
};

export const Holiday = Template.bind({});
```

## Solutions

### 4. Add RequestInfo Story

```typescript
export default {
  title: 'Eternal/RequestInfo',
  component: RequestInfoComponent,
  decorators: [
    moduleMetadata({
      imports: [
        BrowserAnimationsModule,
        ReactiveFormsModule,
        HttpClientModule,
        MatButtonModule,
        MatIconModule,
        MatInputModule
      ]
    })
  ]
} as Meta;

const Template: Story<RequestInfoComponent> = (args: RequestInfoComponent) => ({
  component: RequestInfoComponent,
  props: args
});

export const Normal = Template.bind({});
```
