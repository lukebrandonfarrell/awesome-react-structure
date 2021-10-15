# 12 Rules for React

# Files and Structure

### 1. Flat Structure Divided by Type

The structure of your react native application should be divided by type. Here is a scalable example structure:

```jsx
/components
    /tests
    /styles
       my-component-style.js
    my-component.js

/pages
    /tests
	  /styles
       home-page-style.js
    home-page-style.js

/utils
   alert-with-exit-app.js
   get-distance-between-points.js
   set-android-icon.js
   use-component-size.js

/config
   images.js
   main-tab-strings.js
   debug-config.js

/theme
   metrics.js
   global-styles.js
```

As you can see this structure is extensible. The files in the '/theme' folder could also be move to the '/config' folder. This is also true for the '/utils' folder, if you had 200 files which contained custom react hooks, you could make a folder new called '/hooks'.

It is recommended that you don't go beyond two folders for splitting up components, even if you have thousands of components (**developers don't search via folders, they use hot keys and search**), this is because these folders have sub-folders for '/styles' and '/tests', creating more folders is extra overhead.

The example above splits components into '/pages' and '/components' to provide a high level overview of the application. Each root page is accessible via the '/pages' folder. Make sure to suffix each root page with either `-page` or `-screen`. For smaller application, you can put pages into the '/components' folder and delete '/pages'... but having a high level overview of the structure of your application and the available pages is useful at the application grows in size.

> Why? Dividing features by domain can cause friction by creating overhead when trying to classify new components. For most projects dividing by type is the best approach.

**Note on** **Index.js Exporting**

When using a flat structure, you do not need to use absolute imports. A flat structure works well with relative imports... absolute imports break IDE intelligence. You should consider using an `index.js` in each folder e.g. '/components', '/utils' etc. This allows you to export files and import them with a single statement:

```jsx
import {
  alertWithExitApp,
  getDistanceBetweenPoints,
  useComponentSize
} from "../utils";
```

This does require some additional overhead to maintain the exports in the `index.js` files, but the benefits of simplified imports, usually outweighs the maintenance overhead.

### 2. Single Export per File

Each function and component should be split into its own separated file. Same with config and theme folders above, each file should only include a single object configuration. e.g. the `images.js` contains an object requiring all the images for the application. Each file should match the name of the exported function, component or configuration e.g. `function getDistanceBetweenPoints` should be in a file name `get-distance-between-points.js`.

> Why? This makes the codebase easier to navigate when looking for components, functions and objects. It also reduces the chance of duplicate code when files follow verbose naming conventions.

### 3. Use Named Exports

Export all function, components, and configurations as named exports. e.g. `import { MyComponent } from "./components/MyComponent"`. Not default.

> Why? Exporting named exports stops the same function, component, configuration acquiring different names across your codebase, reducing complexity and overhead.

# Components

### 4. Always use Functional Components (As of React 16.8)

With the introduction of hooks in React 16.8 you should always use functional components with hooks over class components.

> Why? 

1. In a functional component you can declare your variables, process props, and declare external state once, and use it across your component without having to bind those values to the component or state.

2. There is less complexity involved as there is no overhead thinking about binding functions to context 'this' and lifecycle methods.

### 5. Declare Component Variables in the Same Place

Component level state and variables should always be declared at the top of your functional component. Comments can be used to separate common groups of variables to aid readability, but not necessary. Below is a real-world example: 

```jsx
const EditProfileScreen = ({ componentId }) => {
  /** Redux */
  const isBiometricByDefault = useSelector(state => getIsBiometricByDefault(state));
  const { 
		isLoading: isSubmitting, 
		errors 
	} = useSelector(state => getNetworkRequestByType(state, UPDATE_USER_REQUEST));
  const {
    authToken,
    name,
    email,
    birthday,
    mobile,
    marketingEmails,
    postcode
  } = useSelector(state => getUser(state));
  const dispatch = useDispatch();
  /** Hooks */
  const formikRef = useRef(null);
  const [biometricType, biometricTypeAsString] = useBiometricType();
  const [confirmPasswordError, setConfirmPasswordError] = useState(false);
  /** Variables */
  const firstName = getFirstNameFromName(name);
  const lastName = getLastNameFromName(name);
  const mobilePrefix = getMobilePrefixFromString(mobile);
  const mobileNumber = getMobileNumberFromString(mobile);
  /** Effects */
  useEffect(() => ...
```

> Why? This stops redeclaration of variables across large components and helps reduce code by giving a top level overview of all data and processing requirements and that component.

### 6. Extract Callbacks into Nested Functions

Callbacks in your `useEffects` and components (e.g. `<MyComponent onPress={...}`) should be extracted into functions at the bottom of your component:

```jsx
useEffect(() => onFirstNameChanged(), [firstName])

return (
	 <MyComponent onPress={onPressMyComponent}
   ...

function onPressMyComponent() ...
function onFirstNameChanged() ...
```

> Why? This keeps components organised and readable. Just like putting your clothes in your wardrobe, you don't just leave everything around on the floor... don't do the same with your application.

### 7. Don't Nest Components Inside Components

Use `<Components />` instead of render functions e.g. `renderComponent`, when splitting larger components into smaller pieces. This one is extremely simple... don't put JSX inside variables or functions for conditional rendering. Use a component with a descriptive name.

`renderBottomFooterConditions() { ...` ❌

`my-component-bottom-footer-conditions.js -> <MyComponentBottomFooterConditions` ✅

Using 'render' function is a sneaky way of getting around the '2. Single Export per File' principle...

...*I see what you are doing 👀and I'm not impressed.*

> Why? React is made with components for composition. When something is complex enough that an inline JSX ternary is not sufficient and extra processing and complex conditionals require it to be extracted into its own function, just make a new component.

### 8. Flatten Component Props

In most cases, flatten objects before passing them to your components via props. There are some cases where you would want to pass an object as a component property e.g. *"you have a common set of data processing logic which needs to happen in multiple places across your application... [See Example](https://medium.com/@lukebrandonfarrell/component-composition-for-dummies-cd94515a360e)"*. When using a type system with JavaScript e.g. TypeScript, this principle can be more flexible... it is still recommended to flatten props.

> Why? 

1. Working with objects is more complex and error prone than single value variables; booleans, numbers, strings.

2. Passing an object to a component couples it to that object data structure, in most cases, you want your components more flexible than that.

### 9. Separate Styles from Components

Styles should be separated into a separate file. Following this guide your styles must exist in a `/styles` folder at the root level of your component folder. If you are using multiple folders to split components (e.g. components / containers) then each will have a separate styles folder.

 

> Why? Separating styles increases component code readability and promotes reuse of style in your application.

### 10. Follow a Naming Convention

Follow naming conventions for your project, you can have your own naming conventions, but here is a default convention:

- File and folder names should be written in kebab-case: `my-new-component.js`
- Constants which are declared outside the component should be start-case: `IconSources`
- Images which are imported into the component should be start-case:  `import ArrowImage from 'assets/arrow.png';`
- Functions should be written in camel-case: `getDistanceBetweenPoints`
- Variables and functions inside components should be camel-case: `const activeBrandId ...`
- Components should be named with start-case: `<MyComponent />`
- Default and named imports from third party libraries should follow the project naming conventions: `import { SetPlatformIcon: setPlatformIcon } from 'rn-native-icon'`

> Why? Keeps everything tidy.

### 10. Use Descriptive Naming

Variable and function names should be descriptive instead of generic:

`let messageForTheWarningBanner = messages?.message_for_banner` ✅

`let message = messages?.message_for_banner` ❌

Further example for deconstructing:

```jsx
const { name: companyName } = useSelector(state => getOrganisation(state));
const { email: currentEmail } = useSelector(state => getUser(state));
```

> Why? It makes code easier to understand.

# Comments

'How' and 'Why' did we get here!?

Please... comment your code. If you do not comment your code then below explains how to comment code. Comments drastically increase the robustness of your project, not commenting code is like selling a 'build it yourself' bookshelf with no instructions. 

### 11. Explain the 'Why' not the 'How'

Comments are used to explain the 'why' behind the code. The 'how' can be inferred from looking at the code. Talk about the reasons behind your choices for writing that portion of code. Here is a comment for a single variable, in a real-world application:

```jsx
/**
 * Used so the rebrand alert does not appear multiple times,
 * as if the user exits the app, this will trigger new user data
 * to be fetched, which will trigger new organisation data to be
 * fetched, which will lead to multiple FETCH_ORGANISATION_RESPONSE
 * and multiple instances of this generator function waiting at
 * `call(waitUntilStack...`.
 */
let isRebrandAlertShowed = false;
```

> Why? The 'how' can be inferred from the code... the 'why' is forever lost.

### 12. Breakdown a Programs Flow with Comments

Comments can be used to break down the control flow of a program to make it more readable. Note: that 'why' comments should be used instead when applicable, these sort of comments 'help the how' but do not replace the 'why':

```jsx
// Used to close the array in our JavaScript file
iconsJavascriptContent = iconsJavascriptContent + "]";

// We use object assign, this takes the existing icons, our primary icon, and icons folder and updates them
plistObject["CFBundleIcons"]["CFBundleAlternateIcons"] = 
    Object.assign(plistObject["CFBundleIcons"]["CFBundleAlternateIcons"], PRIMARY_ICON, icons);

// Build the plist object back into an XML list
const result = plist.build(plistObject);

// Write the modified plist file back to source
try {
    fs.writeFileSync(PLIST_PATH, result);
    terminal.white(`\n The plist file ${PLIST_PATH} was updated with new app icons. \n`);
} catch(e) {
    return terminal.red(err);
}

// Write to the JavaScript file used in the app to keep track of our active icons
try {
    fs.writeFileSync(JS_FILE_PATH, iconsJavascriptContent);
} catch(e) {
    return terminal.red(err);
}
```

> Why? Increases readability for less experienced developers who can't immediately interpret the 'how'.
