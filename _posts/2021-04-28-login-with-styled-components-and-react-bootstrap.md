---
title: "Login with styled components and react bootstrap"
date: 2021-04-17T12:59:30+01:00
categories:
  - guides
tags:
  - styled-components
  - react-bootstrap
  - react
last_modified_at: 2021-04-28T12:59:30+01:00
---

We are going to create a login page using `react bootstrap` components and providing the style with the lib `styled-components`

# Requirements
This guide assumes you already have installed in your system `node v.15.14.0` or newer.

## Setting up the environment:
We have to create a new react project for our login page.

``` sh
mkdir styled-components-loginPage
cd styled-components-loginPage
yarn create react-app my-app
cd my-app
yarn start
```
Yarn start is going to open a browser window and show you the content of your project. 
&nbsp;
&nbsp;

## Create a basic login page:
The first step is to create our login page in our project. 
As you can see, my app has these files created. 

{% include figure image_path="/assets/posts/login-page/login-tree-start.jpeg" alt="initial login tree" caption="" %}

LoginPage is going to be a component in our project. So we are going to create a folder called components in our src folder. Then we create inside components our login page file. 
As this file is going to have js extension, the first letter of the name has to be capitalize. We have to see something like this:

{% include figure image_path="/assets/posts/login-page/tree-with-login-page-created.jpeg" alt="tree with login page created" caption="" %}

In index.js you can see that we have 

``` javascript
ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);
```
Index is calling the method render from ReactDOM, and is going to render in root what we tell him to render, in this case <App />.
But we don't want that root renders <App />, we want it to render <LoginPage />.

So we have to change it

``` javascript
ReactDOM.render(
  <React.StrictMode>
    <LoginPage />
  </React.StrictMode>,
  document.getElementById('root')
);
```
Look in your browser what is showing. You should see a blank page because we don't have content in our LoginPage.js

If we did it like this, you should see this error in your browser
{% include figure image_path="/assets/posts/login-page/failed-to-compile-login-page.jpeg" alt="failed to compile login page" caption="" %}

Why is that? Because we created our LoginPage.js but index.js doesn't know that this page exist. 

Let's fix this.
In LoginPage.js we have to define our component. 

```javascript
import React from 'react'
class LoginPage extends React.Component{
    render(){
        return()
    }
}
```
In the return we are going to write what we what to show. Just to see something in our browser, let's render a simply paragraph with the text "This is a paragraph"

```javascript
class LoginPage extends React.Component{
    render(){
        return(<p>This is a paragraph</p>)
    }
}
```

If you tried to see this text in the browser, you can see that still is showing the same error. 
We have to do two more steps. First we have to export LoginPage class, so other files can use it.
We only have to add the word export before class

```javascript
export class LoginPage extends React.Component{
    render(){
        return(<p>This is a paragraph</p>)
    }
}
```
Then we have to import this component where we want to use it, in our case is index.js.
In index.js we have to see something like this:

```javascript
import { LoginPage } from './components/LoginPage'
ReactDOM.render(
  <React.StrictMode>
    <LoginPage />
  </React.StrictMode>,
  document.getElementById('root')
);
```
Great! Now we see "This a paragraph" in our browser! 

## Create content with react-bootstrap:

We managed to show the paragraph in our browser but we don't want to see that. We want to have our login page here. 
For that, we are going to create a simply structure. One row with two columns. In the left side, we are going to show our icon page. In the right side, we are going to have a h1 with a button that allows us to login.

As we are using react, the best way to create our login page is using `react-boostrap` components. Let's install `react-bootstrap` (https://react-bootstrap.github.io/). If you are running your project, you will have to press control + C to stop it and then write in you folder project this command 

```sh
yarn add react-bootstrap bootstrap
```

We have to add the following link in the index.html file that is in public folder. Or we have to add the bootstrap.css in our project.

```html 
    <link
      rel="stylesheet"
      href="https://cdn.jsdelivr.net/npm/bootstrap@4.6.0/dist/css/bootstrap.min.css"
      integrity="sha384-B0vP5xmATw1+K9KRQjQERJvTumQW0nPEzvF6L/Z6nronJ3oUOFUFpCjEUQouq2+l"
      crossorigin="anonymous"
    />  
```

Now that we have all installed we can create our row with two columns in our return statment, in LoginPage.js. For that we have to import `Row` and `Col` components from `react-bootstrap`

```javascript
import Row from 'react-bootstrap/Row'
import Col from 'react-bootstrap/Col'

export class LoginPage extends React.Component{
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

Now we have to write our content for the columns. Let's start with the first one. In my
case I am going to put our logo.

As we are going to start putting images in our project, we should create a folder for these ones. 
{% include figure image_path="/assets/posts/login-page/tree-images-folder.jpeg" alt="tree images folder" caption="" %}

As we want to show this image in our Col, we should use the `Image` component from react-bootstrap. Remember to import it 
```javascript
import Image from 'react-bootstrap/Image'

export class LoginPage extends React.Component{
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
We have to put the src for the image. As we have the image in our folder, we have to import it and call it.

```javascript
import logo from '../images/logo.png'

export class LoginPage extends React.Component{
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

Now we want to create our h1 and button for our right column. We are going to use `Button` from react-bootstrap

``` javascript
import Button from 'react-bootstrap/Button'

export class LoginPage extends React.Component{
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

## Create styles with styled-components:

### Install lib styled-components:

Now that we have our content, we want to give it some style! 
We are going to install `styled-components` lib `(https://styled-components.com/)

``` sh
yarn add styled-components
```
### Style components from react-bootstrap: 

We are going to center all the content vertically and horizontally. For this task we have to style Row component. 
First of all we have to import `styled` from 'styled-components'
Then we are going to create a constant with the first letter capitalize (if not the lib is not going to change that constant for the Row and it's not going to show the style in the browser). We assign to this constant styled(Row). We have to write it this way because we are styling a component. Then, as a string and in curly braces, we write the styles like css.
We write this constant outside of the class component to have separate funcionallity from styles. 

```javascript
const StyledRow = styled(Row)`{
    display: flex;
    align-items: center;
    text-align: center;
    height: 100vh;
    margin: 0;
}`
```

Now we have to assign this constant to the actual component that we want to style. So we replace Row for StyledRow. 

```javascript
export class LoginPage extends React.Component{
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
Let's create the style for our image. 
```javascript
const StyledImage = styled(Image)`{
    max-width: 100%;
    height: auto;
    width: 200px;
}`

export class LoginPage extends React.Component{
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
### Style tags with styled-components: 

Now let's style our h1. We have to write it a little different. H1 is a <tag> so for these cases after the word styled we write a dot and the tag that we want to style. Let's see an example. 

``` javascript 
const StyledH1 = styled.h1`{
    color: green;
    font-weight: 800;
}`

<StyledH1>Welcome!</StyledH1>
```

### Style pseudo-clases with styled-components: 

For our button it's the same case as the Row or Image. 

```javascript
const StyledButton = styled(Button)`{
    border-radius: 2rem;
    background-color: gray;
    padding: 0.3rem 2rem ;
    border: none;
    
}`
 <StyledButton>Login</StyledButton>
```
As you can see, if you put your mouse above the button, the hover action display a background color blue, but we don't want that. We want that on hover, active and focus the background color is a light gray.
This is an example of pseudo classes:

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

### Add icons from font-awesome:

Let's say that now we want an icon after the text in our button. First, we have to install `font-awesome` (https://fontawesome.com/how-to-use/on-the-web/using-with/react)

```sh 
yarn add @fortawesome/fontawesome-svg-core
yarn add @fortawesome/free-solid-svg-icons
yarn add @fortawesome/react-fontawesome
yarn add @fortawesome/free-brands-svg-icons
yarn add @fortawesome/free-regular-svg-icons
```

Now we have to import the icon that we want to use and put it in our button

``` javascript
import { FontAwesomeIcon } from '@fortawesome/react-fontawesome'
import { faSignInAlt } from "@fortawesome/free-solid-svg-icons";

<StyledButton>Login <FontAwesomeIcon icon={faSignInAlt}/></StyledButton>
```

But we want this icon to be green. 
We could create a new StyledIcon and style it but as we have to icon inside the button, we can add this style to the StyledButton that we already created. 

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
### Add responsive breakpoints in our styled components:

When we are in mobile, our logo is pretty big. We should give it a smaller width for this case. In our StyledImage we have to add @media. You have to write these specifications from highest to lowest.

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

### Reusing components with same styles: 

Let's say that we want to add a new Image below our button, for example the logo related to styled components but smaller. Do we have to create another styledImage or can we use the same one?
We want to use some of the properties that we add in StyledImage, but not all of them.
In these cases we should create a new component so we can reuse the properties.
Let's create a new component in our components folder.

{% include figure image_path="/assets/posts/login-page/tree-with-image-with-style.jpeg" alt="tree with component image with style" caption="" %}

In this component we are going to write all the information that is the same for all the images that we are going to create. 

```javascript
import React from 'react'

export class ImageWithStyle extends React.Component{
    render(){
        return()
    }
}
```
We want to return an styled image component that can render diffents images with their own properties. For that we have to declare in our StyledImage the properties. In this case: src and className. 
We should have something like this in our ImageWithStyle.js file

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
In our LoginPage we deleted all the information that we are using now in ImageWithStyle.
How we use <ImageWithStyle /> in LoginPage.js? We have to import it and call it where we want to!

```javascript
import { ImageWithStyle } from "../components/ImageWithStyle"

export class LoginPage extends React.Component{
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

We have to style these two components with the css that we want. 
In this case we are going to style our own component. You can see that we do it the same way as a react component
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

export class LoginPage extends React.Component{
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

Now we have to instance the properties that we declared in <ImageWithStyle />.
So we have to instance the src and the className. 
For the first one, we already had imported logo, and I added logoStyledComponent.

```javascript
import logo from '../images/logo.png'
import logoStyledComponent from '../images/logoStyledComponents.png'
export class LoginPage extends React.Component{
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
As you can see the images were rendered with the styled widths that we gave them, but for example, in the <StyledLogoPage /> we didn't specify it. Why is this working? Because styled-components creates classes that are added in the element that we are telling it to do it. 
<StyledLogoPage /> is the father of <ImageWithStyle />. Because of this, the second one receives from the constructor all the properties from <StyledLogoPage />. We said that styled-componets lib creates new classes for the elements. So we have to specify in the child className={this.props.className} so the style can be rendered.  
If you don't give to ImageWithStyle the className prop, the images are not going to have the styles that we define. 

### Reusing styles: 


If we focus on our right column, we can see that the components don't have space between them. We are going to add this space, creating a styled div. We can add it in each one but it's better if we create a styled constant, because if we want to change it after, we only have to do it in one place. 

``` javascript

const RightColumnSpaces = styled.div`{
    margin-bottom: 1rem;
}`

export class LoginPage extends React.Component{
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
### Show style depending of condition from properties: 

The last thing we are going to see if what happens if we want to style a component depenending of the condition of a propertie.
Let's suppose that we want that some images in our loginPage have the same background and others don't. We can create a propertie backgroundImage in the constructor of our LoginPage, and set it to true. Then, when we call our component ImageWithStyle like this or instance with style, we can give this propertie as parameter. 

```javascript
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
            </Col>
        </StyledRow>
        )
    }
}
```
and in ImageWithStyle we add the style that we want to add if this condition is true. See that now the StyledImage is inside the render method. Why is that? Because if not this class can't read {this.props.background}. This propertie comes with the super(props), so we must call it inside the render. 

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