---
title: "Constants styles"
date: 2021-05-09T20:18:30+01:00
categories:
  - tips
tags:
  - styled-components
  - classes 
  - constants
last_modified_at: 2022-04-28T12:59:30+01:00
---

In this tutorial we are going to `create` our own `styled constant` so we can use them all over different `files`.

We are using the `LandingPage` as an example from the [last guide](/guides/landing-with-styled-components-and-react-bootstrap/) *LandingPage with styled-components*.

{% include toc icon="cog" title="Content" %}


# Create a folder for the styled constants

Let's create a folder for the styled constants (`src/styles`)

```sh
cd src
mkdir styles
```
## Create a file for all the colors:

So far, in our project, we used the `colors` *green*, *gray* and *black*. 

Let's imagine that we want to `replace` our *green* color for a *blue* one. 

Instead of replacing all the *green* colors white the *blue* ones in each `file`, we can `create` a `constant` and then change it in only one place. 

- Create a `colors file` in the `styles` folder:

```sh
touch styles.js
```

- Define the colors: 

```javascript
const colors = {
    black: "#333333",
    gray: "#CCCCCC",
    greenDark: "#134240"
}
```

- Create the `constants` and `export` them:

```javascript
const colors = {
    black: "#333333",
    gray: "#CCCCCC",
    greenDark: "#134240",
    yellow: "yellow",
}

export const black = colors.black;

export const gray = colors.gray;

export const green = colors.green

export const yellow = colors.yellow
```
*NOTE: this way you only import the constant that you need in each file.*

- Import the `constants` and use them:

In `ImageWithStyle.js`, change the `background color`. 

```javascript
import { black } from '../../styles/colors'
const bgBlack = `{
    background-color: ${black}
}`

```
*NOTE: if you have the constant created, you can start writing it in the file where you have to used it, and it will appear an option to auto import it.* 

And that's it! You can try changing all the colors!

## Create a file for general styles:

`StyledRow` in `LoginPage.js` has the following properties:  `display:flex` and `align-items:center`. These two also are defined in the `span` of `RadioButton.js`. 

These `properties` are very useful because they `align` our content `vertically`. We are going to use them a lot in our projects. 

- Let's create the file `generalStyles.js`.

```sh
touch generalStyles.js
```

- Create the `constants` and `export` them:

```javascript
export const alignVertically = `{
    display: flex;
    align-items: center
}`;
```

- Import the `constants` and use them:
  - In `RadioButton.js`:
```javascript
import { alignVertically } from '../../styles/generalStyles';
const StyledSpan = styled.span`{
      ${alignVertically};
      margin-left: 20px;
      padding-left: 7px;
}
`;
```
  - In `LoginPage.js`:
```javascript
import { alignVertically } from '../styles/generalStyles';
const StyledRow = styled(Row)`{
      ${alignVertically};
      text-align: center;
      height: 100vh;
      margin: 0;
}`;
```
