---
title: "Radio buttons with styled components (Safari compatible)"
date: 2021-05-09T19:18:30+01:00
categories:
  - guides
tags:
  - styled-components
  - radio button
  - safari 
last_modified_at: 2021-05-10T17:40:30+01:00
---

We are going to create our styled component for radio buttons.

{% include toc icon="cog" title="Content" %}

# Create the radioButton file

Let's create the `radioButton` file

```sh
touch RadioButton.js
```

Let's create the content for the `RadioButton.js`

``` react
import React from 'react'

export class RadioButton extends React.Component {
    render(){
        return(
            <label>
                <input type="radio" />
                <span>{this.props.text}</span>
            </label>
        )
    }
}
```
*NOTE: there is a bug when you are using Form.Check that does not renders the input correctly in Safari, so we use this custom solution*

# Add RadioButton to a component

Let's add the `RadioButton` to the `LandingPage.js`, we are using the `LandingPage` as an example from the [last guide](/guides/landing-with-styled-components-and-react-bootstrap/) *LandingPage with styled-components*.

 
``` react
import { RadioButton } from '../components/generics/RadioButton'

export class LandingPage extends React.Component{
    ...
    render(){
        return(
        ...
            <RightColumnSpaces>
                <RadioButton text="This is a radio button" />
            </RightColumnSpaces>
        ...
        )
    }
}
```

# Style RadioButton with styled-components

Let's change the style of the `radio button`.

## Style for label: 

Change the `position`, so the circle appears in the left side of the text. 

``` react
const StyledLabel = styled.label`
     {
        position: relative;
    }
`;

export class RadioButton extends React.Component {
    render(){
        return(
            <StyledLabel>
                <input type="radio" />
                <span>{this.props.text}</span>
            </StyledLabel>
        )
    }
}
```

## Style for input:

- Hide the predefined circle: `opacity: 0` and `z-index: -1`
- Remove space from the predefine circle: `position: absolute`
- Give style to the checked state: add the icon that you want to display (specify the `font-family` and the `background-color`)

``` react
const StyledInput = styled.input`
     {
        opacity: 0;
        z-index: -1;
        position: absolute;
        &:checked ~ span::after {
            font-family: "FontAwesome";
            content: "\f00c";
            background: green;
        }
    }
`;

export class RadioButton extends React.Component {
    render(){
        return(
            <StyledLabel>
                <StyledInput type="radio" />
                <span>{this.props.text}</span>
            </StyledLabel>
        )
    }
}
```

*NOTE: in `ìndex.html` you must add the fontawesome css file to use the fontawesome family as an attribute.*

## Style for span: 

Create the `circle` with the new styles. To align vertically the `text` and the `radio button` we apply `display: flex` and `align-items:center`.

- Give style to the pseudo element **before**: 
  - Empty content: `content: ""`.
  - Dimensions: `width` and `height`.
  - Position in the left side of the text: `absolute` and `left: 0`.
  - Circle: `border-radius: 100%`.

``` react 
const StyledSpan = styled.span`
     {
        display: flex;
        margin-left: 20px;
        padding-left: 7px;
        align-items: center;
        &::before {
            content: "";
            position: absolute;
            border-radius: 100%;
            width: 20px;
            height: 20px;
            left: 0;
            border: 1px solid green;
        }
    }
`;

export class RadioButton extends React.Component {
    render(){
        return(
            <StyledLabel>
                <StyledInput type="radio" />
                <StyledSpan>{this.props.text}</StyledSpan>
            </StyledLabel>
        )
    }
}
```

- Give style to the pseudo element **after**: 
  - Dimensions: `width` and `height`.
  - Position  `absolute`.
  - Circle: `border-radius: 100%`.
  - Center circle: `left: 3px`.


```react
const StyledSpan = styled.span`
     {
        display: flex;
        margin-left: 20px;
        padding-left: 7px;
        align-items: center;
        &::before {
            content: "";
            position: absolute;
            border-radius: 100%;
            width: 20px;
            height: 20px;
            left: 0;
            border: 1px solid green;
        }
        &::after {
            position: absolute;
            width: 14px;
            height: 14px;
            left: 3px;
            border-radius: 100%;

        }
    }
`;
```