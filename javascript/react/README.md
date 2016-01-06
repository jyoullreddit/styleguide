# reddit React/JSX Style Guide

*A mostly reasonable approach to React and JSX*

## Table of Contents

  1. [Basic Rules](#basic-rules)
  1. [Class vs `React.createClass`](#class-vs-reactcreateclass)
  1. [Naming](#naming)
  1. [Declaration](#declaration)
  1. [Alignment](#alignment)
  1. [Spacing](#spacing)
  1. [Props](#props)
  1. [Parentheses](#parentheses)
  1. [Tags](#tags)
  1. [Methods](#methods)
  1. [Ordering](#ordering)
  1. [`isMounted`](#ismounted)

## Basic Rules

  - Always use JSX syntax.
  - Do not use `React.createElement` unless you're initializing the app from a file that is not JSX.
  - Prefer stateless components. There are many benefits to this approach including performance optimizations.

## Class vs `React.createClass`

  - Use `class extends React.Component` not `React.createClass`

  eslint rules: [`react/prefer-es6-class`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/prefer-es6-class.md).

    ```javascript
    // bad
    const Listing = React.createClass({
      render() {
        return <div />;
      }
    });

    // good
    class Listing extends React.Component {
      render() {
        return <div />;
      }
    }
    ```

## Naming

  - **Extensions**: Use `.jsx` extension for React components.
  - **Filename**: Use lowercase for filenames. E.g., `reservationcard.jsx`.
  - **Reference Naming**: Use PascalCase for React components and camelCase for their instances.

  eslint rules: [`react/jsx-pascal-case`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-pascal-case.md).

    ```javascript
    // bad
    import reservationCard from './ReservationCard';

    // good
    import ReservationCard from './ReservationCard';

    // bad
    const ReservationItem = <ReservationCard />;

    // good
    const reservationItem = <ReservationCard />;
    ```

  - **Component Naming**: Use the filename as the component name. For example, `ReservationCard.jsx` should have a reference name of `ReservationCard`. However, for root components of a directory, use `index.jsx` as the filename and use the directory name as the component name:

    ```javascript
    // bad
    import Footer from './Footer/Footer';

    // bad
    import Footer from './Footer/index';

    // good
    import Footer from './Footer';
    ```

## Declaration

  - Do not use `displayName` for naming components. Instead, name the component by reference.

    ```javascript
    // bad
    export default React.createClass({
      displayName: 'ReservationCard',
      // stuff goes here
    });

    // good
    export default class ReservationCard extends React.Component {
    }
    ```

## Alignment

  - Follow these alignment styles for JSX syntax

  eslint rules: [`react/jsx-closing-bracket-location`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-closing-bracket-location.md).

    ```javascript
    // bad
    <Foo superLongParam='bar'
         anotherSuperLongParam='baz' />

    // good
    <Foo
      superLongParam='bar'
      anotherSuperLongParam='baz'
    />

    // if props fit in one line then keep it on the same line
    <Foo bar="bar" />

    // children get indented normally
    <Foo
      superLongParam='bar'
      anotherSuperLongParam='baz'
    >
      <Spazz />
    </Foo>
    ```

## Spacing

  - Always include a single space in your self-closing tag.

    ```javascript
    // bad
    <Foo/>

    // very bad
    <Foo                 />

    // bad
    <Foo
     />

    // good
    <Foo />
    ```

## Props

  - Always use camelCase for prop names.

    ```javascript
    // bad
    <Foo
      UserName='hello'
      phone_number={ 12345678 }
    />

    // good
    <Foo
      userName='hello'
      phoneNumber={ 12345678 }
    />
    ```

  - Add space inside curly braces. As with normal Javascript Objects.

  ```javascript
  //bad
  <Foo
    foo={bar}
  />

  //good
  <Foo
    foo={ bar }
  />
  ```

  - Do not omit the value of the prop when it is explicitly `true`.

  eslint rules: [`react/jsx-boolean-value`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-boolean-value.md).

    ```javascript
    // good
    <Foo
      hidden={ true }
    />

    // bad
    <Foo
      hidden
    />
    ```

## Parentheses

  - Aways wrap JSX tags in parentheses.

  eslint rules: [`react/wrap-multilines`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/wrap-multilines.md).

    ```javascript
    // bad
    render() {
      return <MyComponent className='long body' foo='bar'>
               <MyChild />
             </MyComponent>;
    }

    // good
    render() {
      return (
        <MyComponent className='long body' foo='bar'>
          <MyChild />
        </MyComponent>
      );
    }

    ```

## Tags

  - Always self-close tags that have no children.

  eslint rules: [`react/self-closing-comp`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/self-closing-comp.md).

    ```javascript
    // bad
    <Foo className='stuff'></Foo>

    // good
    <Foo className='stuff' />
    ```

  - If your component has multi-line properties, close its tag on a new line.

  eslint rules: [`react/jsx-closing-bracket-location`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-closing-bracket-location.md).

    ```javascript
    // bad
    <Foo
      bar='bar'
      baz='baz' />

    // good
    <Foo
      bar='bar'
      baz='baz'
    />
    ```

## Methods

  - Bind event handlers for the render method in the constructor.

  > Why? A bind call in a the render path creates a brand new function on every single render.

  eslint rules: [`react/jsx-no-bind`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-no-bind.md).

    ```javascript
    // bad
    class extends React.Component {
      onClickDiv() {
        // do stuff
      }

      render() {
        return <div onClick={this.onClickDiv.bind(this)} />
      }
    }

    // good
    class extends React.Component {
      constructor(props) {
        super(props);

        this.onClickDiv = this.onClickDiv.bind(this);
      }

      onClickDiv() {
        // do stuff
      }

      render() {
        return <div onClick={this.onClickDiv} />
      }
    }
    ```

## Ordering

  - Ordering for `class extends React.Component`:

  1. `constructor`
  1. optional `static` methods
  1. `getChildContext`
  1. `componentWillMount`
  1. `componentDidMount`
  1. `componentWillReceiveProps`
  1. `shouldComponentUpdate`
  1. `componentWillUpdate`
  1. `componentDidUpdate`
  1. `componentWillUnmount`
  1. *clickHandlers or eventHandlers* like `onClickSubmit()` or `onChangeDescription()`
  1. *getter methods for `render`* like `getSelectReason()` or `getFooterContent()`
  1. *Optional render methods* like `renderNavigation()` or `renderProfilePicture()`
  1. `render`

  eslint rules: [`react/sort-comp`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/sort-comp.md).

**[⬆ back to top](#table-of-contents)**
