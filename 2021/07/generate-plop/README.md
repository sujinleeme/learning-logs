# Generate React Component with plop.js

I built a react component generator with Plop.js. Plop lets you define your own parameterized code templates, which can be injected into your source code to generate boilerplate code that you would otherwise have to write yourself.

If I run `yarn generate`, it generates a file set for component.

```
❯ yarn generate
yarn run v1.22.10
$ yarn plop --plopfile ./generators/plopfile.js
$ /Users/sujin.lee/Documents/React-App/node_modules/.bin/plop --plopfile ./generators/plopfile.js
? What is your component name? button
? What is your component type? atoms
✔  ++ /Users/sujin.lee/Documents/React-App/src/ui/components/atoms/button/index.ts
✔  ++ /Users/sujin.lee/Documents/React-App/src/ui/components/atoms/button/button.tsx
✔  ++ /Users/sujin.lee/Documents/React-App/src/ui/components/atoms/button/button.styles.ts
✔  ++ /Users/sujin.lee/Documents/React-App/src/ui/components/atoms/button/button.stories.tsx
✔  ++ /Users/sujin.lee/Documents/React-App/src/ui/components/atoms/button/button.test.tsx
```

## How to config plop

Install plop as a dev-dependency

```
yarn add -D plop
```

package.json

```
{
  ...
  "scripts": {
    "generate": "yarn plop --plopfile ./generators/plopfile.js"
  }
}
```

```
── generators
  ├── plopfile.js
  └── templates
      ├── Component.tsx.hbs
      ├── index.ts.hbs
      ├── stories.tsx.hbs
      ├── styles.ts.hbs
      └── test.tsx.hbs
```

index.ts.hbs

```hbs
export * from './{{dashCase name}}';
```

component.tsx.hbs

```hbs
import * as S from './{{dashCase name}}.styles'

export interface {{pascalCase name}}Props {

export const {{pascalCase name}}:React.FC<{{pascalCase name}}Props}> = () => (
  <S.Wrapper>
    <h1>{{pascalCase name}}</h1>
  </S.Wrapper>
)
```

stories.tsx.hbs

```hbs
import { withDesign } from 'storybook-addon-designs'
import { Story, Meta } from '@storybook/react/types-6-0'

import { {{pascalCase name}}, {{pascalCase name}}Props } from '.'

export default {
  title: '{{pascalCase name}}',
  component: {{pascalCase name}},
  decorators: [withDesign],
  argType: {},
  parameters: {
    design: {
      type: 'figma',
      url: ''
    }
  }
} as Meta

export const Default: Story<{{pascalCase name}}Props> = (args) => <{{pascalCase name}} {...args} />
```

style.ts.hbs

```
import styled from 'styled-components'

export const Wrapper = styled.main``
```

test.tsx.hbs

```
import { render, screen } from '@testing-library/react'

import { {{pascalCase name}} } from '.'

describe('<{{pascalCase name}} />', () => {
  it('should render the heading', () => {
    const { container } = render(<{{pascalCase name}} />)

    expect(screen.getByRole('heading', { name: /Test/i })).toBeInTheDocument()

    expect(container.firstChild).toMatchSnapshot()
  })
})

```

plopfile.js

```js
module.exports = (plop) => {
  plop.setGenerator('component', {
    description: 'Create a component',
    prompts: [
      {
        type: 'input',
        name: 'name',
        message: 'What is your component name?',
      },
      {
        type: 'list',
        name: 'type',
        message: 'What is your component type?',
        choices: ['atoms', 'molecules'],
      },
    ],
    actions: [
      {
        type: 'add',
        path: '../src/ui/components/{{type}}/{{dashCase name}}/index.ts',
        templateFile: 'templates/index.ts.hbs',
      },
      {
        type: 'add',
        path: '../src/ui/components/{{type}}/{{dashCase name}}/{{dashCase name}}.tsx',
        templateFile: 'templates/component.tsx.hbs',
      },
      {
        type: 'add',
        path: '../src/ui/components/{{type}}/{{dashCase name}}/{{dashCase name}}.styles.ts',
        templateFile: 'templates/styles.ts.hbs',
      },
      {
        type: 'add',
        path: '../src/ui/components/{{type}}/{{dashCase name}}/{{dashCase name}}.stories.tsx',
        templateFile: 'templates/stories.tsx.hbs',
      },
      {
        type: 'add',
        path: '../src/ui/components/{{type}}/{{dashCase name}}/{{dashCase name}}.test.tsx',
        templateFile: 'templates/test.tsx.hbs',
      },
    ],
  });
};
```
