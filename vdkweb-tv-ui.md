# VDK Web TV UI
A library with React components to build UI for TV devices.

## nav object

```

/**
 * @typedef {Object} ThemeObject
 * @property {string} buttonContainer Classname for the button container.
 * @property {string} button Classname for the button element.
 * @property {string} buttonFocused Classname for the button element when it is in focus state.
 * @property {string} buttonActive Classname for the button element when it is in active state
*/


/**
 * @typedef {Object} NavObject
 * @property {string} id Must have. Unique focus id for the nav system.
 * @property {string} nextup Declares the next up component id.
 * @property {string} nextdown Declares the next down component id.
 * @property {string} nextleft Declares the next left component id.
 * @property {string} nextright Declares the next right component id.
 * @property {string} parent Declares the parent id.
 * @property {boolean} parent Declares the parent id.
 * @property {boolean} skip Allows skipping when navigating the focus.
 * @property {string} forwardFocus A focus id from another component to forward focus when focusing the current id.
 * @property {boolean} useLastFocus Whether to remember the last focus status when leave the component which has focusable children.
 */
```

## FocusDiv usage

Focus Div is a simple HTML `<div>` element with the knowledge of spatial navigation by wrapping a styleable `<div>` element with the `withFocus` high-order function from `@accedo/vdkweb-navigation`. The major use case is to receive the nav property object from the parent, modify the properties when necessary, and pass it on to the children element(s) if needed.

```
/**
 * @typedef {Object} FocusDivProps
 * @property {Object} style
 * @property {string} className
 * @property {Object} children
 * @property {string} dir
 * @property {NavObject} nav
 */

/**
 * Focus Div is a simple HTML `<div>` element with the knowledge of spatial
 * navigation by wrapping a styleable `<div>` element with the withFocus high-order
 * function from `@accedo/vdkweb-navigation`.
 *
 * The major use case is to receive the nav property object from the parent,
 * modify the properties when necessary, and pass it on to the children element(s)
 * if needed.
 *
 * @param {FocusDivProps} data
 * @returns {React.ReactElement} The FocusDiv Element
 */
const FocusDiv = (data) => {
  return (...)
}
```

A FocusDiv component can contain multiple children (e.g. FocusButton, which is defined below).

## FocusButton

```
/**
 * @typedef {Object} FocusButtonProps
 * @property {Object} style
 * @property {string} className
 * @property {Object} children
 * @property {boolean} isActive
 * @property {NavObject} nav
 * @property {Function} onClick
 * @property {ThemeObject} theme
 */

/**
 * Focus Button is an HTML5 button wrapped inside a `<div>` element. This set up
 * allows the Focus Button not only handle traditional mouse click, but also
 * provides spatial navigation, an essential functionality for application on CTV.
 *
 * @param {FocusButtonProps} data
 * @returns {React.ReactElement} The FocusButton Element
 */
const FocusButton = (data) => {
  return (...)
}
 ```

Focus Button is an HTML5 button wrapped inside a `<div>` element. This set up allows the Focus Button not only handle traditional mouse click, but also provides spatial navigation, an essential functionality for application on CTV.

### Example #1 of use - Simple FocusButton inside a react component.

```

const ExampleFocusButtonComponent = () => {
  return (
    <FocusButton
      nav={{
        id: 'example_focus_button',
        parent: 'example_parent_focus_button',
      }}
      theme={{
        buttonContainer: style.focusButtonContainer,
        button: style.menuButton,
        buttonFocused: style.focusMenuButtonFocused,
        buttonActive: style.focusButtonActive
      }}
      onClick={() => {
        console.log('Example Focus Button pressed!');
      }}
    >
      {Press me!}
  </FocusButton>);
}

```

### Example #2 of use - FocusDiv containing 2 FocusButton with a horizontal navigation layout

```
const ExampleHorizontalButtons = () => {
  return (
    <FocusDiv
      nav={{
        id: 'container',
        forwardFocus: 'button_1',
        useLastFocus: true
      }}
      className={style.buttonsContainer}
      dir={dir}
    >
        <FocusButton
          nav={{
            id: 'button_1',
            nextright: 'button_2'
            parent: 'container'
          }}
          theme={{
            buttonContainer: style.focusButtonContainer,
            button: style.menuButton,
            buttonFocused: style.focusMenuButtonFocused,
            buttonActive: style.focusButtonActive
          }}
          onClick={() => {console.log('Pressed Button 1')}}
        >
          {Press Button 1}
        </FocusButton>

        <FocusButton
          nav={{
            id: 'button_2',
            nextleft: 'button_1'
            parent: 'container'
          }}
          theme={{
            buttonContainer: style.focusButtonContainer,
            button: style.menuButton,
            buttonFocused: style.focusMenuButtonFocused,
            buttonActive: style.focusButtonActive
          }}
          onClick={() => {console.log('Pressed Button 2')}}
        >
          {Press Button 2}
        </FocusButton>
    </FocusDiv>
  );
}


```