If you are a develepor front end, you are familiar with bootstrap classes.

As you can see in these tutorials, we are not using them. But sometimes it should be great to use it because we are always writing the same styles all over different files.

Instead of doing that, we can create our own constant and call it in any file that we want. 
These constants are going to be globall to our application. So we should create a folder for this. Where? Well, the constant are not part of our components, so we should create our styles folder in src

<!-- imagen del styles trree -->

Let's start with the colors. In our project so far we use the colors: green, gray and black. Let's suppose that we want to change our green color for the blue one. As you may think, we have to go in each file and change it! Now because we only have three files, but imagine that we have 200. It would be really frustrating. 
There is a solution for that!
We can create our color as constant, and then call them. Then, if we have to change our green color for blue, we only have to do it in one place! 

In our styles folders, let's create a js file with the name colors.

<!-- imagen de colors trree -->

Let's define our three colors. 
```javascript
const colors = {
    black: "#333333",
    gray: "#CCCCCC",
    greenDark: "#134240"
}
```
Now that we define our colors, we have to create our constant and export them so our project can reach them and use them. 

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

This is very good too, because now we can use the colors separate. We dont have to import all the file, we only import the color that we want to use in that file. 
Let's change in our components the colors. 
In ImageWithStyle.js we have to change that background color. For that we have to import the color. Our style is inside a string. We have to read the constant. For that we have to write ${...}. Let's see an example. 
TIP: if you have the constant created, you can start writing it where you need it, and it will appear and option to auto import it. If not, you will have to write it in the top of our file. 
```javascript
import { black } from '../../styles/colors'
const bgBlack = `{
    background-color: ${black}
}`

```
And that's it! You can try changing all the colors!

As we did it with colors, we can do it with other styles. In our StyledRow in LoginPage.js we have display:flex and align-items:center. We have these two in the span of RadioButton.js too. 
These properties are very useful because they align our content vertically. We are going to use them a LOT in our projects. Let's create anothe file for this kind of styling, for example, displays.js.

<!-- imagen del tree con displays -->

Now let's create our style. As we are going to use it directly as we are doing so far in our styledComponents, we are going to create the constant we the same format. Es un string, no?

```javascript
export const alignVertically = `{
    display: flex;
    align-items: center
}`
```

Then we are going to use this in LoginPage and RadioButton. 
Remember always to import them!
In RadioButton.js:
```javascript
import { alignVertically } from '../../styles/displays';
const StyledSpan = styled.span`
     {
        ${alignVertically};
        margin-left: 20px;
        padding-left: 7px;
    }
`
```
In LoginPage.js
```javascript
import { alignVertically } from '../styles/displays';
const StyledRow = styled(Row)`{
    ${alignVertically};
    text-align: center;
    height: 100vh;
    margin: 0;
}`

```
As you can see, the imports are not the same. Why is that? Because LoginPage is not in the same level as RadioButton, so for the last one we have to move outside generics. 