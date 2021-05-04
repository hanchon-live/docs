We are going to create our styled component for radio buttons. 

the first step is to create our component in our components folder. 

<!-- poner imagen del tree de radiobutton -->

As you can see we are mixing LoginPage with other components that we could re-use. We should create for these type of components a folder called generics and move all the generic components to this folder. 

<!-- poner imagen del tree con folder generics -->

We'll have to change the import for ImageWithStyle because now it's inside our generics folder. In loginPage.js we are going to update this import:


Our base structure is going to be like this:
```javascript
import { ImageWithStyle } from "../components/generics/ImageWithStyle"
```
Now that we have our components organized, we are going to create the content for our RadioButton

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

We could use the react component Form.Check and give it style. The problem with this is that we have to give the style to the pseudo classes before and after for the input. If we give style to a tag that ends with a / in safari, it doesnt work. We have to give it to a start ending tag. 
input semi elemento
los otros son elementos completos. 

We pass this.props.text to the span, we the component can render any text that we want. Always try to create the components as generics as you can, so they could be re-used in diffents files. 

Let's see how it looks. We are going to add in our LoginPage a RadioButton component below our right image.

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
As you can see, the radio button has a black border, and when we check it, the background color turns blue. Let's say that we don't want that. Let's change the sytle. 

For that we have to changed the style for the label, span and input.
for the label we only are going to change the position, so our circle appears in the left side of the text. 
``` javascript
const SytledLabel = styled.label`
     {
        position: relative;
    }
`;
export class RadioButton extends React.Component {
    render(){
        return(
            <SytledLabel>
                <input type="radio" />
                <span>{this.props.text}</span>
            </SytledLabel>
        )
    }
}
```

For the input, first we have to hide the predefinided circle. For that we are going to give it ocapity 0, and we dont have that that circle bother us so we are going to put a z-index= -1.
```javascript
const StyledInput = styled.input`
     {
        opacity: 0;
        z-index: -1;
    }
`;

export class RadioButton extends React.Component {
    render(){
        return(
            <SytledLabel>
                <StyledInput type="radio" />
                <span>{this.props.text}</span>
            </SytledLabel>
        )
    }
}
```

Then we have to create our new circle with the properties that we want.
Let's create a circle that has border green. We give this style to the pseudo element before. 
We dont want content inside the radio button in the before, so we apply content: ''. We give our dimension for the circle with height and witdh. With position and left, we put the circle in the left side of the text. As we want a circle, we can play with the border-radius and transform it to a circle. Suppose that we want a square. We can only change the border-radius and we have it! Last, we have to provide the color for our border. 
We need to align vertically our text and radio button, for that we use display: flex and align-items: center. We provide margin and padding left so the radio button is separate from the text. 

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
            <SytledLabel>
                <StyledInput type="radio" />
                <StyledSpan>{this.props.text}</StyledSpan>
            </SytledLabel>
        )
    }
}
```

As you can see, between our image and our input, it appeared a space. That's because we still have the first circle (the blue one). Try changing the opacity to 1, you can see it. For that, we have to give to our inpout position:absolute. This places the blue circle over the text. We dont care about this, because it's hidden!

```javascript
const StyledInput = styled.input`
     {
        opacity: 0;
        z-index: -1;
        position: absolute;
    }
`;
```

What else do we have to do? Give the style to the element when it's checked! For that we have to create the pseudo element after in our StyledSpan and create the checked state related to the after for the input.
Let's start with our style for the input. We want that this style is applied when the input is checked and the span has the pseudo element after. We can combine these two with checked ~ span. 
We specify the content that we want to show and the color.

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
```

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