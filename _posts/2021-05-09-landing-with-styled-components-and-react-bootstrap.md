---
title: "Landing page with styled-components and react-bootstrap"
date: 2021-05-09T22:17:30+01:00
categories:
  - guides
tags:
  - styled-components
  - react-bootstrap
  - react
last_modified_at: 2021-05-09T22:17:30+01:00
---

We are going to create a **Landing page** using [react-bootstrap](https://react-bootstrap.github.io/) components and [styled-components](https://styled-components.com/).

{% include figure image_path="/assets/posts/landing-page/header.png" alt="React bootstrap styled components" caption="" %}
{% include toc icon="cog" title="Content" %}


# Requirements
This guide assumes you already have installed in your system [node v.15.14.0](https://github.com/nvm-sh/nvm) or newer.

# Setting up the environment
We have to create a new `react-app` for our landing page.

``` sh
mkdir styled-components-landingPage
cd styled-components-landingPage
yarn create react-app my-app
cd my-app
yarn start
```
After running `yarn start` a new browser window will be opened with your project. 
&nbsp;
&nbsp;

# Create a basic landing page
## Create the LandingPage file:

After running `yarn create react-app`, let's start creating a new folder for all our components:

``` sh 
cd src
mkdir components
```

Inside our `components` folder create a `LandingPage.js` file. We are capitalizing this file name, so it matches the `JavaScript` class that we are going to create.

``` sh 
cd components
touch LandingPage.js
```

## Call our LandingPage instead of the example App:
The `index.js` file should have this code:

``` javascript
ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);
```

Index is calling the method `render` from `ReactDOM`. This method set the `html` code for the element with the id `root`. 

The example just renders the component called `App`. We don't want to render `<App />`, we want it to render `<LandingPage />` to test our project.

So we have to change it:

``` javascript
ReactDOM.render(
  <React.StrictMode>
    <LandingPage />
  </React.StrictMode>,
  document.getElementById('root')
);
```

If you check our project in the browser, you may think that it'll render a blank page because the `LandingPage.js` file is empty.

React checks if everything is ok when is trying to compile the code, so it'll tell us that our component is not defined:

{% include figure image_path="/assets/posts/landing-page/failed-to-compile-landing-page.jpeg" alt="failed to compile landing page" caption="" %}

## Creating a react component:

Let's fix the last error, in `LandingPage.js` define the new component:

```javascript
import React from 'react'
export class LandingPage extends React.Component{
    render(){
        return()
    }
}
```
*NOTE: The class must be `exported` by adding the `export` word before `class`, so it can be used by the `index.js` file.*

The content that we want to display in the browser must be the `return` of the `render` function.

For example, let's render a simply paragraph with the text *This is a paragraph*.

``` react
export class LandingPage extends React.Component{
    render(){
        return(<p>This is a paragraph</p>)
    }
}
```

## Use a react component:

To import this component in the `index.js` file, just a line should be added.

In `index.js` the `LandingPage` is imported using relative paths.

```javascript
import { LandingPage } from './components/LandingPage'
ReactDOM.render(
  <React.StrictMode>
    <LandingPage />
  </React.StrictMode>,
  document.getElementById('root')
);
```
Great! Now we see "This a paragraph" in our browser! 
{% include figure image_path="/assets/posts/landing-page/Screenshot from 2021-05-04 17-15-22.png" alt="this is a paragraph image" caption="" %}

# Create content with react-bootstrap
## Install react-bootstrap

Let's create the content for the landing page instead of the paragraph: one `row` with two `columns`. In the first column, we are going to show a logo. In the second one, a h1 with a button that allows us to login.
For that, let's install [react-bootstrap](https://react-bootstrap.github.io/)

```sh
yarn add react-bootstrap bootstrap
```
And add the following link in `index.html` (this file is in `public folder`).

```html 
    <link
      rel="stylesheet"
      href="https://cdn.jsdelivr.net/npm/bootstrap@4.6.0/dist/css/bootstrap.min.css"
      integrity="sha384-B0vP5xmATw1+K9KRQjQERJvTumQW0nPEzvF6L/Z6nronJ3oUOFUFpCjEUQouq2+l"
      crossorigin="anonymous"
    />  
```
## Create components with react-bootstrap
### Add Row and Col components

To create these components in the `LoginPage.js` file, we only have to import them from `react-bootstrap` and add them in the return statment. 

```javascript
import Row from 'react-bootstrap/Row'
import Col from 'react-bootstrap/Col'

export class LandingPage extends React.Component{
    render(){
        return(
          <Row>
            <Col xs={6}></Col>
            <Col xs={6}></Col>
          </Row>
        )
    }
}
```
### Add Image component

Let's add the logo in the first column. First, let's create an images folder for our project.

```sh
cd ..
mkdir images
```
For `images`, `react-bootstrap` has a specific component. 

```javascript
import Image from 'react-bootstrap/Image'

export class LandingPage extends React.Component{
    render(){
        return(
        <Row>
            <Col xs={6}>
                <Image src="" />
            </Col>
            <Col xs={6}></Col>
        </Row>
        )
    }
}
```
Let's import the image logo. In `src`, the logo has to be called between `{}` (that way the code can read the value for the variable)

```javascript
import logo from '../images/logo.png'

export class LandingPage extends React.Component{
    render(){
        return(
        <Row>
            <Col xs={6}>
                <Image src={logo} />
            </Col>
            <Col xs={6}></Col>
        </Row>
        )
    }
}
```

In the browser you should see something like this:
{% include figure image_path="/assets/posts/landing-page/first-col-with-Image.jpeg" alt="first col with logo" caption="" %}

### Add Button component

Let's add the h1 and button in the second column. Let's add the `button` component from `react-bootstrap`.

``` javascript
import Button from 'react-bootstrap/Button'

export class LandingPage extends React.Component{
    render(){
        return(
        <Row>
            <Col xs={6}>
                <Image src={logo} />
            </Col>
            <Col xs={6}>
                <h1>Welcome!</h1>
                <Button>Login</Button>
                <Image />
            </Col>
        </Row>
        )
    }
}
```
In the browser you should see something like this:
{% include figure image_path="/assets/posts/landing-page/second-col-with-content.jpeg" alt="second col with content" caption="" %}

## Create styles with styled-components

### Install lib styled-components

Let's install [styled-components](https://styled-components.com/) to style the content.

``` sh
yarn add styled-components
```

### Style Row with styled-components

Let's center the content of the `row` vertically and horizontally. 

- Import `styled` from 'styled-components'

```javascrtipt
import styled from 'styled-components'
```
- Create a constant. The name must be capitalize. We are going to pass a string value to the styled function. 

```javascript
const StyledRow = styled(Row)`{
    display: flex;
    align-items: center;
    text-align: center;
    height: 100vh;
    margin: 0;
}`
```
*NOTE: The code inside `styled` has to represent the component. It could be a `html element` or a `react component`.*

- Replace the `component` for the constant created.

```javascript
export class LandingPage extends React.Component{
    render(){
        return(
        <StyledRow>
            <Col xs={6}>
                <Image src={logo} />
            </Col>
            <Col xs={6}>
                <h1>Welcome!</h1>
                <Button>Login</Button>
            </Col>
        </StyledRow>
        )
    }
}
```
In the browser you should see something like this:
{% include figure image_path="/assets/posts/landing-page/styledRow.jpeg" alt="styled Row" caption="" %}

### Style Image with styled-components

Let's create the style for our image. 

- Create a constant. The name must be capitalize. We are going to pass a string value to the styled function. 


```javascript
const StyledImage = styled(Image)`{
    max-width: 100%;
    height: auto;
    width: 200px;
}`

export class LandingPage extends React.Component{
    render(){
        return(
        <StyledRow>
            <Col xs={6}>
                <StyledImage src={logo} />
            </Col>
            <Col xs={6}>
                <h1>Welcome!</h1>
                <Button>Login</Button>
            </Col>
        </StyledRow>
        )
    }
}
```
### Style html elements with styled-components

- Create a constant. The name must be capitalize. We are going to pass a string value to the styled function. 
As the code inside styled is an html elment, we write it with a dot.

``` javascript 
const StyledH1 = styled.h1`{
    color: green;
    font-weight: 800;
}`

<StyledH1>Welcome!</StyledH1>
```
In the browser you should see something like this:
{% include figure image_path="/assets/posts/landing-page/h1-with-style.jpeg" alt="h1 with style" caption="" %}

### Style pseudo-classes with styled-components: 

- Create a constant. The name must be capitalize. We are going to pass a string value to the styled function. 

```javascript
const StyledButton = styled(Button)`{
    border-radius: 2rem;
    background-color: gray;
    padding: 0.3rem 2rem ;
    border: none;
    
}`
 <StyledButton>Login</StyledButton>
```
In the browser you should see something like this:
{% include figure image_path="/assets/posts/landing-page/styled-button.jpeg" alt="style for button" caption="" %}

The `hover` action displays a blue background. Let's apply a light gray color to the pseudo classes `hover`, `active` and `focus`.

``` javascript
const StyledButton = styled(Button)`{
    border-radius: 2rem;
    background-color: gray;
    padding: 0.3rem 2rem ;
    border: none;
    &:hover, 
    :not(:disabled):not(.disabled):active, 
    :focus,
    :not(:disabled):not(.disabled):active:focus{
        color: black;
        background-color: #f5f5f5;
        box-shadow: none;    
    
    }
}`
```

In the browser you should see something like this:
{% include figure image_path="/assets/posts/landing-page/button-with-hover-activated.jpeg" alt="button with pseudo classes styled" caption="" %}

### Add icons from font-awesome

Let's add an icon after the text in our button. 

- Install [font-awesome](https://fontawesome.com/how-to-use/on-the-web/using-with/react)

```sh 
yarn add @fortawesome/fontawesome-svg-core
yarn add @fortawesome/free-solid-svg-icons
yarn add @fortawesome/react-fontawesome
yarn add @fortawesome/free-brands-svg-icons
yarn add @fortawesome/free-regular-svg-icons
```

- Import the icon that you want and add it in the button

``` javascript
import { FontAwesomeIcon } from '@fortawesome/react-fontawesome'
import { faSignInAlt } from "@fortawesome/free-solid-svg-icons";

<StyledButton>Login <FontAwesomeIcon icon={faSignInAlt}/></StyledButton>
```

- Style the icon with a yellow color. 
As the icon is a svg inside the button, we can style it in the button itself. 

```javascript
const StyledButton = styled(Button)`{
    border-radius: 2rem;
    background-color: gray;
    padding: 0.3rem 2rem ;
    border: none;
    &:hover, 
    :not(:disabled):not(.disabled):active, 
    :focus,
    :not(:disabled):not(.disabled):active:focus{
        color: black;
        background-color: #f5f5f5;
        box-shadow: none;    
    }
    svg{
        color:yellow;
    }
}`
```
In the browser you should see something like this:
{% include figure image_path="/assets/posts/landing-page/button-with-fontAwesome-icon.jpeg" alt="button with font awesome icon styled" caption="" %}

### Add responsive breakpoints with styled components

In `mobile`, let's give a smaller width to our logo.

Add in `StyledImage` `@media`.

```javascript
const StyledImage = styled(Image)`{
  max-width: 100%;
  height: auto;
  width: 200px;
  @media (max-width: 576px){
      width:100px;
  }
}`
```

*NOTE: write the responsive breakpoints from highest to lowest*

In the browser you should see something like this:
{% include figure image_path="/assets/posts/landing-page/image-responsive.jpeg" alt="image responsive" caption="" %}

### Reusing components with same styles: 

Let's add a new Image below our button, for example the logo related to styled components but smaller. 

For this, we want to use some of the properties that we add in StyledImage, but not all of them.

In these cases we should create a `new component` so we can `reuse` the properties.

- Create a `new component` in our components folder.

```sh
cd components
touch ImageWithStyle.js
```

- Import `React`, `Image` and `styled` components.

- Create the `styled constant` for the `Image` component

- Specify the `src` and `className` properties using the class `props`. 
This allows us to use the component in several scenarios.

```javascript
import React from 'react'
import Image from 'react-bootstrap/Image'
import styled from 'styled-components'

const StyledImage = styled(Image)`{
    max-width: 100%;
    height: auto;
}`

export class ImageWithStyle extends React.Component{
    render(){
        return(
            <StyledImage src={this.props.src} className={this.props.className} />
        )
    }
}
```

- In `LandingPage.js` delete the information that we are using now in `ImageWithStyle`.

- Import `ImageWithStyle` in `LandingPage.js` call it in the first column

```javascript
import { ImageWithStyle } from "../components/ImageWithStyle"

export class LandingPage extends React.Component{
    render(){
        return(
        <StyledRow>
            <Col xs={6}>
                <ImageWithStyle />
            </Col>
            <Col xs={6}>
                <StyledH1>Welcome!</StyledH1>
                <StyledButton>Login <FontAwesomeIcon icon={faSignInAlt}/></StyledButton>
                <ImageWithStyle />
            </Col>
        </StyledRow>
        )
    }
}
```

- Create the constants to style `ImageWithStyle`.

```javascript 
const StyledLogoPage = styled(ImageWithStyle)`{
    width: 200px;
    @media (max-width: 576px){
         width:100px;
    }
}`

const StyledLogoStyledComp = styled(ImageWithStyle)`{
    width:80px;
}`

export class LandingPage extends React.Component{
    render(){
        return(
        <StyledRow>
            <Col xs={6}>
                <StyledLogoPage />
            </Col>
            <Col xs={6}>
                <StyledH1>Welcome!</StyledH1>
                <StyledButton>Login <FontAwesomeIcon icon={faSignInAlt}/></StyledButton>
                <StyledLogoStyledComp />
            </Col>
        </StyledRow>
        )
    }
}

```

- Instance the properties declared in `ImageWithStyle`.

- Import the images for `src` property.

```javascript
import logo from '../images/logo.png'
import logoStyledComponent from '../images/logoStyledComponents.png'
export class LandingPage extends React.Component{
    render(){
        return(
        <StyledRow>
            <Col xs={6}>
                <StyledLogoPage src={logo} />
            </Col>
            <Col xs={6}>
                <StyledH1>Welcome!</StyledH1>
                <StyledButton>Login <FontAwesomeIcon icon={faSignInAlt}/></StyledButton>
                <StyledLogoStyledComp src={logoStyledComponent}/>
            </Col>
        </StyledRow>
        )
    }
}
```
*NOTE: you don't have to specify className property because `styled-components` creates the prop className and it will automatically propagate to the children.* 

### Reusing styles 

Let's add some space between the components of the second column by creating a `styled div`.

``` javascript

const RightColumnSpaces = styled.div`{
    margin-bottom: 1rem;
}`

export class LandingPage extends React.Component{
    render(){
        return(
        <StyledRow>
            <Col xs={6}>
                <StyledLogoPage src={logo} />
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
            </Col>
        </StyledRow>
        )
    }
}
```

In the browser you should see something like this:
{% include figure image_path="/assets/posts/landing-page/space-between-components.jpeg" alt="space between components" caption="" %}

### Show style depending of condition from properties

Let's style a component depending of the `condition` of a property. Suppose that we want that some images in our `landingPage` have the same background and others don't. 

- Create the property `backgroundImage` in the constructor of our `LandingPage`, and set it as `true`. 

- Pass as a parameter the variable created in `ImageWithStyle` component.

```javascript
export class LandingPage extends React.Component{
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
            </Col>
        </StyledRow>
        )
    }
}
```
- In `ImageWithStyle.js`, add the style if this condition is true. 

*NOTE: the constant with the style is inside the class because we have to read props.*

```javascript 
const bgBlack = `{
    background-color: black
}`

export class ImageWithStyle extends React.Component{
    render(){
        const StyledImage = styled(Image)`{
            max-width: 100%;
            height: auto;
            ${this.props.background && bgBlack}
        }`
        return(
            <StyledImage src={this.props.src} className={this.props.className}/>
        )
    }
}
```

In the browser you should see something like this:
{% include figure image_path="/assets/posts/landing-page/background-for-some-images.jpeg" alt="background-for-some-images" caption="" %}
