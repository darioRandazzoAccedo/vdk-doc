# VDK Web Navigation Component
A navigation and focus handling library

## Docs
Repository URL: Source code location
Package Documentation + Stories
Install

`npm install @accedo/vdkweb-navigation@latest`

## Usage
The recommendation is to see the Storybook examples, both for this package and related, like @accedo/vdkweb-tv-ui or @accedo/vdkweb-epg.

This is how you can make a React component to support navigation:

```
import { withFocus } from '@accedo/vdkweb-navigation';

const CompA = ({ isFocused }) => (
    <div>{isFocused ? 'Focused!' : 'Waiting'}</div>
);

export default withFocus(CompA);
```

Then, you can specify the navigation id of this component:

```
render() {
  return (
    <div>
      <CompA nav={{ id: 'first', nextdown: 'second' }}/>
      <CompA nav={{ id: 'second', nextdown: 'third', nextup: 'first' }}/>
      <CompA nav={{ id: 'third', nextup: 'second' }}/>
    </div>
  )
}
```

Normally you will not need to manually specify the ids but use helpers to generate these from the componets structure, or define the top id of a component.

Then you can specify the first focused element:

```
import { focusManager } from '@accedo/vdkweb-navigation';

// ...

focusManager.changeFocus('first');
```

Finally, you can listen to key events in the app:

```
import { focusManager } from '@accedo/vdkweb-navigation';

xdk.environment.addEventListener(xdk.environment.SYSTEM.KEYDOWN, ({ id }) => {
    switch (id) {
        case UP.id:
        case DOWN.id:
        case LEFT.id:
        case RIGHT.id: {
            const direction = `next${id.substring(KEY_EVENT_PREFIX.length)}`;
            focusManager.navigateFocus(direction);
            break;
        }
        case OK.id:
            focusManager.click();
            break;
        default:
            break;
    }
});
```

## API

### focusManager

#### focusManager.changeFocus(focusId: string): void
Use this to change the existing focused element. If using withFocus, this will forward the focus if necessary, and will rerender just the unfocused and new focused components.

#### focusManager.navigateFocus(direction: string): void
This will move the focus in a certain direction, and when using withFocus it will be able to calculate the next focused element.

#### focusManager.listenToFocusChanged(fn: (data, previousData): void): removeListenerFn
You can use this in some components to be notified when the focus change. Remember to unlisten (using the same function reference) on componentWillUnmount: focusManager.unlistenToFocusChanged(fn).

##### Example #1 of use with vanilla javascript

```
import { focusManager } from '@accedo/vdkweb-navigation';

/**
 * @typedef {Object} FocusChangedData
 * @property {string} currentFocus
 */

/**
 * @param {FocusChangedData} data
 * @param {FocusChangedData} previousData
 */
const handleFocusChanged = (data, previousData) => {
    const { currentFocus } = data;
    console.log('currentFocus ', currentFocus);
}

focusManager.listenToFocusChanged(handleFocusChanged);
```

##### Example #2 of use with React

```
import React, { useEffect } from 'react';
import { focusManager } from '@accedo/vdkweb-navigation';

/**
 * @typedef {Object} FocusChangedData
 * @property {string} currentFocus
*/

const UIComponent = () => {
    useEffect(() => {
        /**
         * @param {FocusChangedData} data
         * @param {FocusChangedData} previousData
        */
        const handleFocusChanged = (data, previousData) => {
            const { currentFocus } = data;
            console.log('currentFocus ', currentFocus);
        }

        focusManager.listenToFocusChanged(handleFocusChanged);

        return () => {
            focusManager.unlistenToFocusChanged(handleFocusChanged);
        };
    }, []);

    return (...)
}
```

#### focusManager.isFocused(id: string): boolean
Checks that the passed id is the current focus. It will not validate the passed id in favour of performance.

#### focusManager.isChildFocused(id: string): boolean
Checks that the currently focused element is a child of the passed focus id. The child doesn't need to be a direct child, but could be several layers below.

For example, for a Root > Container > Child (trail ['Child', 'Container', 'Root']), focusManager.changeFocus('Child); focusManager.isChildFocused('Root') would return true.

#### focusManager.isValidFocusId(id: string): boolean
Checks that the passed id is a valid focus id: a non-empty string.

#### focusManager.listenToTrailBuilt(fn: (data, previousData)): removeListenerFn
Called whenever the trail array has finished building.

#### focusManager.setPersistTrail(persistTrail: boolean): void
Enable the persist trail feature for the focus in order to allow to keep and retrieve the trail (array of Focusable parent components) of the Focus.

#### focusManager.getTrail(): array
Returns the trail array if the persistTrail is enabled (using setPersistTrail).

### navigationService
The app will normally not need to use this directly, except when required to access specific navigation state.

#### navigationService.getData(): data
This will return the navigation state object. For performance reasons it will not be cloned but it should not be mutated directly.

#### navigationService.listenToFocusEvent(opts: { id: string, fn: (data, previousData): void,  shouldWarnOnExistingId: bool }): removeListenerFn

### utils
Collection of Higher-Order Components for React.

#### withFocus
Provides navigation features by defining a nav prop which will be used to (via the focusManager and navigationService) update the navigation and notify the necessary components.

##### nav prop structure
Property	Description
id	String. Must have. Unique focus id for the nav system.
nextup	String. Declares the next up component id.
nextdown	String. Declares the next down component id.
nextleft	String. Declares the next left component id.
nextright	String. Declares the next right component id.
parent	String. Declares the parent id.
skip	Boolean. Allows skipping when navigating the focus. Please check "Skip Behaviour" section for more details.
forwardFocus	String. A focus id from another component to forward focus when focusing the current id.
useLastFocus	Boolean. Whether to remember the last focus status when leave the component which has focusable children.
internal	Object.
internal.nextup	Function. Called when navigating with nextup.
internal.nextdown	Function. Called when navigating with nextdown.
internal.nextleft	Function. Called when navigating with nextleft.
internal.nextright	Function. Called when navigating with nextright.
directionReassign	Object. Used to remap the direction of next up/down/left/right. Mainly used for switching right to left app mode.
When passing an internal function, they will be called instead of using the respective navigation value (e.g. internal.nextup takes preference over nextup: 'foo'). When returning true in an internal function, the navigation is stopped. This allows for custom behavior like changing the focus dynamically.

#### Skip Behaviour
When setting skip: true inside the nav, for example in nav={{ id: 'idValue', nextright: 'idValue2', skip: true }}, it is important to know the expected behaviour:

When there are several consecutive skipped components, none of them will be focused in a single press
When the skipped component is the last in the direction (including other parents), the navigation will be stopped in the last non-skipped component
A potentially skipped component should NOT be focused directly with focusManager.changeFocus('...')
When skipping a container, and the focus not yet in any of its children components, all the children components would be skipped on navigation
Please check the Storybook stories or the test cases for withFocus to see the current behaviours
This is a recent feature, please comment any issue and we will add more tests for edge cases
withForwardFocus
Higher-Order Component that wraps a component to always forward the focus to a certain id when this component is mounted.

#### withLayout
Experimental Higher-Order Component, this can be useful to define large and flat layouts via a string definition. However, for complex layouts we recomment to not use this.

#### withFocusDOM
Function that adds focus handling to DOM elements (no extra dependencies). It contains the same unit tests as withFocus and a Storybook story.

## Migration from v1 / v2 to v3
In versions v1 and v2 the navigation would work via redux. In v3 the focusManager and navigationService were introduced in order to minimize the calls on key press.

The migration is possible and recommended if want to improve performance. The steps are:

Remove the usage of the redux actions:
A good place to start: grep -ri vdkweb-navigation src | grep -i actions
Replace these with direct usage of focusManager (e.g. focusManage.changeFocus('foo'))
Remove the usage of the redux state:
Look for navigation: grep -ri modules.navigation src
Selectors usage grep -ri selectors.getnavigationstate src
Look for the usage of the currentFocus: grep -ir props.currentfocus src
Replace with functions from the focusManager (e.g. focusManager.getCurrentFocus())
If you need to listen to focus changes from other components, you can use: focusManager.listenToFocusChanged