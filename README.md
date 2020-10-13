<div align="center">
    <h1>
        <img src="assets/heading.png" alt="Tobit.Labs™ React Project Guideline" width="750" />
    </h1>
    <p>
        Defines a consistent structure for React projects.
    </p>
</div>

---

Most topics in this guide go beyond what can be statically analyzed ESLint. For
that, we also have an
[ESLint-configuration](https://github.com/TobitSoftware/chayns-toolkit#eslint-configuration).

## Overview

-   [Components](#components)
    -   [Use Function Components](#use-function-components)
    -   [Enforce Consistent Component Structure](#enforce-consistent-component-structure)
    -   [Keep Components Small](#keep-components-small)
    -   [Use Error Boundaries](#use-error-boundaries)
    -   [Export Components as `default`](#export-components-as-default)
    -   [Avoid Uncontrolled Inputs](#avoid-uncontrolled-inputs)
    -   [Set the `displayName` Property](#set-the-displayname-property)
    -   [Use `&&` for Conditional Rendering](#use--for-conditional-rendering)
-   [Props](#props)
    -   [Destructure Props](#destructure-props)
    -   [Avoid Objects in Props](#avoid-objects-in-props)
    -   [Use Precise `PropTypes`](#use-precise-proptypes)
    -   [Extract Reusable Shapes](#extract-reusable-shapes)
-   [File Structure](#file-structure)
    -   [Represent the DOM Structure](#represent-the-dom-structure)
-   [State Management](#state-management)
    -   [Use Local State or Context for Simple State](#use-local-state-or-context-for-simple-state)
    -   [Use Redux Toolkit for Complex State](#use-redux-toolkit-for-complex-state)
-   [Styling](#styling)
    -   [Stick to the BEM Methodology](#stick-to-the-bem-methodology)
    -   [Avoid Inline-Styles](#avoid-inline-styles)
    -   [Use `clsx` for Composing Classes](#use-clsx-for-composing-classes)
-   [Utilities](#utilities)
    -   [Extract Commonly Used Utilities](#extract-commonly-used-utilities)
    -   [Do Not Export Utilities as `default`](#do-not-export-utilities-as-default)
    -   [Use Keywords for File Names](#use-keywords-for-file-names)
-   [Other](#other)
    -   [Use Tree-Shaking for `chayns-components`](#use-tree-shaking-for-chayns-components)
-   [Naming conventions](#naming-conventions)
-   [Code formatting](#code-formatting)
-   [About Clean Code](#about-clean-code)

## Components

### Use Function Components

Write components as function components using hooks. Only if the desired
functionality is not available to function components (e.g.
[Error Boundaries](https://reactjs.org/docs/error-boundaries.html)), use class
components.

```js
// good
const Button = () => {
    return /* jsx... */;
};

// avoid
class Button extends React.Component {
    // ...
}
```

> Functions are easier to understand than classes and the `this`-keyword. Hooks
> allow for extraction of stateful logic. Read more about why we use function
> components and hooks in the
> [official React documentation](https://reactjs.org/docs/hooks-intro.html#motivation).

### Enforce Consistent Component Structure

Write function components with this structure:

```js
const Button = ({ children, onClick }) => {
    // ...
};

Button.propTypes = {
    // ...
};

Button.defaultProps = {
    // ...
};

Button.displayName = 'Button';

export default Button;
```

### Keep Components Small

Keep your components as small as possible. When components grow, split them into
multiple components.

> Aim to have fewer than 200 lines of code in a component file.

### Use Error Boundaries

Use [Error Boundaries](https://reactjs.org/docs/error-boundaries.html) around
different parts of your application.

To determine where to use Error Boundaries, think about if the surrounding
components would still function and make sense, if your component would unmount.
Some examples of this:

1. **Two Column Layout**

    Must have an Error Boundary around each of the columns. If one of the
    columns crashes the other one could still be used normally.

2. **Message in Chat Application**

    Must not have an Error Boundary around each message component. A user would
    be confused if one message was missing from the chat. In this case it is
    better to unmount the whole message list.

> When React encounters an error in one of your components, it will unmount the
> tree up to the next Error Boundary or the root. That means without error
> boundaries, your whole screen will go blank when you throw an Error in one of
> your components.

### Export Components as `default`

Export components with `export default`, but always assign them to a variable
with a name. Never export an anonymous function

```js
// good
export default Button;

// bad
export { Button };

// worst
export default () => {
    // component code...
};
```

> Code splitting with
> [`React.lazy`](https://reactjs.org/docs/code-splitting.html#reactlazy) only
> works with components exported as `default`.

### Avoid Uncontrolled Inputs

Always provide the `value`-prop to `<input>`, `<textarea>` or any other
component that can be controlled.

```jsx
// good
const MyForm = () => {
    const [inputValue, setInputValue] = useState('');

    return (
        <input
            value={inputValue}
            onChange={(e) => setInputValue(e.target.value)}
        />
    );
};

// bad
const MyForm = () => {
    const [inputValue, setInputValue] = useState('');

    return (
        <input
            // value prop not set
            onChange={(e) => setInputValue(e.target.value)}
        />
    );
};
```

### Set the `displayName` Property

Specify the `displayName` property on your components. Assign it to the exact
string which is also your variable and file name.

```js
// good
const Button = () => {
    // ...
};

// ...

Button.displayName = 'Button';

export default Button;
```

> The `displayName` property is used by the React Devtools and error messages
> within React. For more information refer to
> [this](https://mariosfakiolas.com/blog/become-a-better-godfather-for-your-react-components/)
> article.

### Use `&&` for Conditional Rendering

Use the logical AND operator (`&&`) in JSX for conditional rendering.

```jsx
// good
<div>
    {todos.length > 0 && <TodoList todos={todos} />}
</div>

// bad
<div>
    {todos.length > 0 ? <TodoList todos={todos} /> : null}
</div>
```

> While this syntax provides great readability, be cautious with non-boolean
> values on the left hand of the `&&` operator. React will skip rendering for
> `null`, `undefined`, the empty string or `false`, but it will render `0` and
> `NaN`. Read more about this
> [here](https://kentcdodds.com/blog/use-ternaries-rather-than-and-and-in-jsx).

## Props

### Destructure Props

Destructure props inside the parameter parentheses for function components.

```jsx
// good
const TodoItem = ({ text, checked }) => (
    <div className="todo">
        <Checkbox className="todo__checkbox" checked={checked} />
        <div>{text}</div>
    </div>
);

// bad
const TodoItem = (props) => (
    <div className="todo">
        <Checkbox className="todo__checkbox" checked={props.checked} />
        <div>{props.text}</div>
    </div>
);
```

### Avoid Objects in Props

Pass single props rather than grouping props as an object.

```jsx
// good
const TodoItem = ({ id, title, text }) => (
    <div className="todo">
        <TodoTitle id={id} title={title} />
        <div>{text}</div>
    </div>
);

// bad
const TodoItem = ({ todo }) => (
    <div className="todo">
        <TodoTitle id={todo.id} title={todo.title} />
        <div>{todo.text}</div>
    </div>
);
```

### Use Precise `PropTypes`

Check objects passed to your component with `PropTypes.shape(...)`. Never use
`PropTypes.object` or `PropTypes.any`.

Similarily, use `PropTypes.arrayOf(...)` instead of `PropTypes.array`.

If a prop is passed as an object, it has to be checked with `PropTypes.shape`,
not `PropTypes.object` or `PropTypes.any`.

```js
// good
Todo.propTypes = {
    todo: PropTypes.shape({
        creationTime: PropTypes.number.isRequired,
        text: PropTypes.string.isRequired,
        id: PropTypes.number.isRequired,
        checked: PropTypes.bool,
    }).isRequired,
    toggleTodoChecked: PropTypes.func.isRequired,
    removeTodo: PropTypes.func.isRequired,
};

// bad
Todo.propTypes = {
    todo: PropTypes.object.isRequired,
    toggleTodoChecked: PropTypes.func.isRequired,
    removeTodo: PropTypes.func.isRequired,
};
```

> A short explanation of `PropTypes.shape` can be found
> [here](https://dev.to/cesareferrari/how-to-specify-the-shape-of-an-object-with-proptypes-3c56).

### Extract Reusable Shapes

Extract complex shapes used in multiple occasions into the
`src/constants/shapes.js`-file for increased reusability.

`shapes.js`:

```jsx
const TODO_SHAPE = {
    creationTime: PropTypes.number.isRequired,
    text: PropTypes.string.isRequired,
    id: PropTypes.number.isRequired,
    checked: PropTypes.bool,
};
```

`Todo.jsx`:

```jsx
Todo.propTypes = {
    todo: PropTypes.shape(TODO_SHAPE).isRequired,
    toggleTodoChecked: PropTypes.func.isRequired,
    removeTodo: PropTypes.func.isRequired,
};
```

## File Structure

### Represent the DOM Structure

Strive to represent the DOM tree with your file structure. That means that
nested components should live in nested folders.

Put shared components in a `shared`-folder in the `components` directory.

If there are multiple different view for an app (e.g. user-view and admin-view),
split their components into different top level folders:

```bash
components
├───admin
├───shared
└───user
```

A directory structure should look like this:

```bash
src
└───components
    │   App.jsx
    ├───add-todo
    │       addTodo.scss
    │       AddTodo.jsx
    ├───headline
    │       Headline.jsx
    ├───intro
    │       Intro.jsx
    └───todos
        │   Todos.jsx
        ├───todo
        │       Todo.jsx
        │       todo.scss
        └───todos-headline
                TodosHeadline.jsx
```

## State Management

### Use Local State or Context for Simple State

Manage simple state without external libraries. Use the `useState` or
`useReducer` hooks and pass props down.

When prop drilling becomes tedious, use React Context and the `useContext` hook.

A guide on how to manage application state without external libraries can be
found in
[this article](https://kentcdodds.com/blog/application-state-management-with-react)
by Kent C. Dodds.

### Use Redux Toolkit for Complex State

Use the [`@reduxjs/toolkit`](https://redux-toolkit.js.org/) to manage complex
state.

Separate global state into
[slices](https://redux-toolkit.js.org/api/createSlice). A slice should look like
this:

```js
import { createSlice } from '@reduxjs/toolkit';

const initialState = { value: 0 };

const counterSlice = createSlice({
    name: 'counter',
    initialState,
    reducers: {
        increment(state) {
            state.value++;
        },
        decrement(state) {
            state.value--;
        },
        incrementByAmount(state, { payload }) {
            state.value += payload;
        },
    },
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;
export const counterReducer = counterSlice.reducer;
```

> Redux Toolkit is the officially recommended way of using Redux.

## Styling

### Stick to the BEM Methodology

Styling your components with (S)CSS-files. Name your classes to follow the
[BEM methodology](https://css-tricks.com/bem-101/).

```scss
// good
.todo-item {
    &__header {
        // ...
    }

    &__checkbox {
        // ...

        &--checked {
            // ...
        }
    }
}

// bad
.todoItem {
    // ...

    .todoItemHeader {
        // ...
    }

    .todoItemCheckbox {
        // ...
    }
}
```

### Avoid Inline-Styles

Avoid inline-styles. If style values must be calculated in the context of a
component, it is fine to use inline-styles.

```jsx
// good
const TodoItem = ({ id, title, text }) => (
    <div className="todo" style={{ height: calculateHeight() }}>
        <TodoTitle id={id} title={title} />
        <div className="todo__text">{text}</div>
    </div>
);

// bad
const TodoItem = ({ id, title, text }) => (
    <div
        style={{
            margin: '8px 0',
            backgroundColor: '#a1a1a1',
        }}
    >
        <TodoTitle id={id} title={title} />
        <div style={{ fontSize: 12 }}>{text}</div>
    </div>;
);
```

### Use `clsx` for Composing Classes

Use the [clsx](https://www.npmjs.com/package/clsx) package for constructing
`className` strings.

```jsx
import clsx from 'clsx';

// ...

const locationMapClasses = clsx(
    'locations-map',
    !isMapsScriptLoaded && 'locations-map--hidden'
);
```

> The `clsx`-package provides the same API as the `classnames`-package but is
> smaller in size and
> [faster at runtime](https://github.com/lukeed/clsx/blob/HEAD/bench).

## Utilities

### Extract Commonly Used Utilities

Never export utility functions from a component file. Create a separate file for
utility functions if they are used in multiple locations.

Extract big utility functions, even if only used in one component.

> Big component files are intimidating and difficult to scan. Aim to have
> component files of 200 or fewer lines of code.

### Do Not Export Utilities as `default`

Use named exports instead of `export default` for utilities.

```jsx
// good
export function getDate() {
    // ...
}

// good
export const getDate = () => {
    // ...
};

// bad
function getDate() {
    // ...
}

export default getDate;
```

### Use Keywords for File Names

Name the files according to the topic of the contained functions. This way several functions can be implemented together in one file without needing a new file for each function. This makes it easier to find functions more quickly.

```bash
src
└───utils
    ├───logger.js           // good
    ├───initializeLogger.js // bad
    │   ...
    ├───viewport.js         // good
    └───getWindowHeight.js  // bad
```

The `viewport.js` file can for example look like this:

```jsx
// With function declaration
export function getWindowHeight() {
    // ...
}

export function getScrollPosition() {
    // ...
}

// With constant declaration
export const getWindowHeight = () => {
    // ...
};

export const getScrollPosition = () => {
    // ...
};
```

## Other

### Use Tree-Shaking for `chayns-components`

`chayns-components` have to be tree-shaken. If your tooling is not automatically
configured for this, refer to the
[tree-shaking guide](https://github.com/TobitSoftware/chayns-components/blob/master/tree-shaking.md).

## Naming conventions

Follow these naming conventions:

-   **Constants** (`UPPER_CASE`)

    ```js
    const FRONTEND_VERSION = 'development';
    ```

-   **Functions** (`camelCase`)

    Name functions as self-explanatory as possible. Names of functions that
    handle user interaction start with `handle`

    ```js
    function handleShowCategory() {
        // ...
    }
    ```

-   **Folders** (`kebab-case`)

    ```bash
        ├───add-todo
        │       addTodo.scss
        └───todos
                todos.scss
    ```

-   **Files** (`camelCase`)

    ```bash
        ├───add-todo
        │       addTodo.scss
        └───todos
                todos.scss
    ```

-   **Component Files** (`PascalCase`)

    Name component files with `PascalCase`, just like components themselves.

    ```bash
    └───components
        │   App.jsx
        ├───add-todo
        │       AddTodo.jsx
        ├───headline
        │       Headline.jsx
    ```

-   **Boolean Values**

    Prefix boolean values with `is`, `has` or `should` (e.g. `isLoading`,
    `hasTitle` or `shouldShowImage`).

## Code formatting

Format the code in your project with [Prettier](https://prettier.io/). Use the
following configuration in your `package.json`-file:

```json
{
    "...": "",
    "prettier": {
        "proseWrap": "always",
        "singleQuote": true,
        "tabWidth": 4
    },
    "...": ""
}
```

> To format files right in your editor, check out the
> [Prettier editor integrations page](https://prettier.io/docs/en/editors.html)
> or the [Webstorm guide](https://prettier.io/docs/en/webstorm.html), if
> you are using that.

> Using the same code formatter important for clean Git diffs. Read
> [this](https://prettier.io/docs/en/why-prettier.html) for more information on
> why we use Prettier.

## About Clean Code

Strive to write clean code that is readable and maintainable. If you want to
read more about clean code, check out
["Clean Code JavaScript"](https://github.com/ryanmcdermott/clean-code-javascript)
by Ryan McDermott.
