🚨 This is still in development.

---

# 🍷 Dripsy

A **dead simple**, **responsive** design system for Expo / React Native Web. Heavily inspired by React's [`theme-ui`](https://theme-ui.com/home).

```jsx
<Text
  sx={{
    fontSize: [14, 16, 20], // 14 on mobile, 16 on tablet, 20 on desktop
    color: ['primary', null, 'accent'], // `primary` on mobile & tablet, `accent` on desktop
  }}
>
  Responsive font size?? 🤯
</Text>
```

# Features

- Responsive styles
- Works with Expo
- Works with Next.js / server-side rendering!
- Full theme support
- Custom theme variants
- TypeScript support (TypeScript theme support is in the works too)
- Insanely simple API (themed, responsive designs in one line!)
- Works with Animated/Reanimated values

# 🤔 Why?

**Build once, deploy everywhere,** is a great philosophy made possible by Expo Web/React Native Web. A large impediment is responsive design.

React Native doesn't have media queries for styles, and trying to micmick it with JS turns into `useState` hell with a ton of conditionals (as you'll see [below](#Before-&-After).)

While React Native has some nice component libraries, it lacks responsive styles that respond to theme changes.

No longer. The goal of this library is to let you go from idea -> universal, themed styles without much effort.

There have been many discussions about what responsive design should look like in React Native. After trying many, many different ways, I'm convinced this is the best.

# 👀 What does Dripsy look like?

## Create a theme!

```js
export default {
  colors: {
    text: '#000',
    background: '#fff',
    primary: 'tomato',
  },
  spacing: [10, 12, 14],
};
```

## ...and build a beautiful, responsive UI

```jsx
<Text
  sx={{
    color: 'primary',
    padding: [1, 3], // [10px, 14px] from theme!
  }}
>
  Themed color!
</Text>
```

_Todo: make the theme values show up in TS types for intelliesense._

# 🏆 Before & After

## Before Dripsy ☹️

This is what it took to make _one_ responsive style without Dripsy...

```jsx
import { useState } from 'react';
import { View } from 'react-native';

const ResponsiveBox = () => {
  const [screenWidth, setScreenWidth] = useState(
    Dimensions.get('window').width
  );

  useEffect(() => {
    const onResize = (event) => {
      setScreenWidth(event.window.width);
    };
    Dimensions.addEventListener('change', onResize);

    return () => Dimensions.removeEventListener('change', onResize);
  }, []);

  let width = '100%';
  if (screenWidth > 700) {
    width = '50%';
  }

  return <View style={{ width }} />;
};
```

A big issue with using JS-only breakpoints like that is that it won't work on SSR apps using Expo + Next.js. The "solution" would be to lazy load the component, but then you lose the SEO benefits of Next.js. With Dripsy, SSR works fine!

## With Dripsy 🤩

```jsx
import { View } from '@nandorojo/dripsy';

const ResponsiveBox = () => {
  return <View sx={{ width: ['100%', '50%'] }} />;
};
```

# 🙉 Installation

```sh
yarn add @nandorojo/dripsy

# or
npm i @nandorojo/dripsy
```

# 🛠 Set up

Technically, you don't have to do anything else!

However, you'll likely want to create a custom theme.

## For Next.js apps

If you're using the expo + next.js integration, there's one extra step.

### 1. Install dependencies

```sh
yarn add next-compose-plugins next-transpile-modules
```

Edit your `next.config.js` file to look like this:

```js
const withPlugins = require('next-compose-plugins');
const withTM = require('next-transpile-modules')([
  '@nandorojo/dripsy',
  // you can add other packages here that need transpiling
]);

const { withExpo } = require('@expo/next-adapter');

module.exports = withPlugins(
  [withTM],
  withExpo({
    projectRoot: __dirname,
  })
);
```

That's it! Btw, if you're using Expo + Next.js, check out my library, [expo-next-react-navigation](https://github.com/nandorojo/expo-next-react-navigation).

## Custom theme

Wrap your entire app with the `ThemeProvider`, and pass it a `theme` object. Make sure you create your theme outside of the component to avoid re-renders.

`App.js`

```jsx
import { ThemeProvider } from '@nandorojo/dripsy';

const theme = {
  colors: {
    text: '#000',
    background: '#fff',
    primary: 'tomato',
  },
  fonts: {
    body: 'system-ui, sans-serif',
    heading: '"Avenir Next", sans-serif',
  },
  spacing: [10, 12, 14],
};

export default function App() {
  return (
    <ThemeProvider theme={theme}>
      {/* Your app code goes here! */}
    </ThemeProvider>
  );
}
```

Follow the [docs from `theme-ui`](https://theme-ui.com/theme-spec) to see how to theme your app – we use the same API as them.

My personal preference is to have the entire theme object in one file.

Example theme:

_All theme values are optional. You don't have to use them if you don't want._

Wrap your entire app with the `ThemeProvider`, and pass it a `theme` object.

`App.js`

```jsx
const theme = {
  colors: {
    text: '#000',
    background: '#fff',
    primary: 'tomato',
  },
  fonts: {
    body: 'system-ui, sans-serif',
    heading: '"Avenir Next", sans-serif',
  },
  spacing: [10, 12, 14],
};

export default function App() {
  return (
    <ThemeProvider theme={theme}>
      {/* Your app code goes here! */}
    </ThemeProvider>
  );
}
```

### Expo + Next.js

If you're using the Expo + Next.js integration, you'll have to [follow the steps](https://docs.expo.io/guides/using-styled-components/#usage-with-nextjs) to use styled components with Next.js + Expo.

- However, there is a bug with this integration that I haven't figured out how to fix yet.

## Usage

Change your imports from `react-native` to `@nandorojo/dripsy`

```diff
- import { View } from 'react-native'
+ import { View } from '@nandorojo/dripsy'
```

# API

## `createThemedComponent`

Currently, a bunch of the React Native components are supported. That said, I haven't added them all. If you want to add one, go to `src/components` and add one and submit a PR.

Or, you can use the `createThemedComponent` function in your own app.

```jsx
import { createThemedComponent } from '@nandorojo/dripsy';
import { View } from 'react-native'

const CustomView = createThemedComponent(View, {
  defaultStyle: {
    flex: 1
  }
})
```

# How it works

First, this library is super inspired by `theme-ui`, and uses many of its low-level functions and methodologies.

Practically speaking, this library uses the `Dimensions` api on Android & iOS, and uses actual CSS breakpoints on web. The CSS breakpoints are made possible by `styled-components`. This means that you get actually-native web breakpoints. That matters, because server-size rendered apps will have startup issues if you use JS-based media queries that require React to rehydrate on when it opens.

On Native, there is nothing too fancy going on. We track the screen width, generate styles based on the current width using a mobile-first approach, and return the regular React Native components. But it just feels like magic!

## Contributing

This is a really new project. I'd love your help and contributions.

Under the hood, this library uses `styled-components/native`.

## TODO

- Add theme variants for certain element types
- Add typing for theme values
  - Maybe add support for custom theme values to be typed too, with a CLI that edits the NPM package? 🧐
    - It could have a `postinstall` script in package.json.
- Add SSR support (for the Expo + Next.js integration.) Currently, the responsive style doesn't rehydrate on the client when the page loads.

## License

MIT
