---
title: "Radio buttons with styled components (Safari compatible)"
date: 2021-05-09T19:18:30+01:00
categories:
  - guides
tags:
  - styled-components
  - radio button
  - safari 
last_modified_at: 2021-05-09T19:18:30+01:00
---

We are going to create our styled component for radio buttons. 

# Create generic folder to organize the files

As you can see we are mixing `LandingPage` with other components that we could re-use. 

Let's create a folder called `generics` and move all the generic components to this folder. 

```sh
cd components
mkdir generics
```

# Create the radioButton file

Let's create the `radioButton` file

```sh
touch RadioButton.js
```

We'll have to change the import for `ImageWithStyle` because now it's inside the `generics` folder. 

Update the import of `ImageWithStyle` in `ladingPage.js`.

```javascript
import { ImageWithStyle } from "../components/generics/ImageWithStyle"
```
Now that we have our components organized, let's create the content for the `RadioButton.js`

```javascript
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

# Add RadioButton in LandingPage.js

Let's add the `RadioButton` in our `LandingPage.js`

```javascript
import { RadioButton } from '../components/generics/RadioButton'

export class LoginPage extends React.Component{
    constructor(props){
        super(props);
        this.backgroundImage = true;
    }
    render(){
        return(
        <StyledRow>
            <Col xs={6}>
                <StyledLogoPage src={logo} background={this.backgroundImage}/>
            </Col>
            <Col xs={6}>
                <RightColumnSpaces>
                    <StyledH1>Welcome!</StyledH1>
                </RightColumnSpaces>
                <RightColumnSpaces>
                    <StyledButton>Login <FontAwesomeIcon icon={faSignInAlt}/></StyledButton>
                </RightColumnSpaces>
                <RightColumnSpaces>
                    <StyledLogoStyledComp src={logoStyledComponent}/>
                </RightColumnSpaces>
                <RightColumnSpaces>
                    <RadioButton text="This is a radio button" />
                </RightColumnSpaces>
            </Col>
        </StyledRow>
        )
    }
}
```

# Style RadioButton with styled-components

Let's change the style for the `radio button`.

## Style for label: 

Change the `position`, so the circle appears in the left side of the text. 

``` javascript
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
- Give style to the checked state: add the icon that you want to display (specify the `font-family` and the `background-color`

```javascript
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

*NOTE: in `ìndex.html` you must add the fontawesome css file to use the fontawesome family as an attribute*

## Style for span: 

Create the circle with the new styles. To align vertically the text and the radio button we apply `display: flex` and `align-items:center`.
- Give style to the pseudo element before: 
  - Empty content: `content: ""`
  - Dimensions: `width` and `height`
  - Position in the left side of the text: `absolute` and `left: 0`
  - Circle: `border-radius: 100%`. If we want and square we put this property at 0.

``` javascript 
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

- Give style to the pseudo element after: 
  - Dimensions: `width` and `height`
  - Position  `absolute`
  - Circle: `border-radius: 100%`. If we want and square we put this property at 0.
  - Center circle: `left: 3px`

Then we have to add in our styledSpan the psuedo element after. We specify the sizes that we want, the position, and we give again the border-radius so it's rounded. 
```javascript
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