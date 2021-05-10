We are going to use the loginPage example and add routes.

You can clone the reporsitory or keep working in your project.

We are going to create a new component, called Dashboard, so we can move through pages. 
You will have something like this: 
{% include figure image_path="/assets/posts/routes-react/tree-with-dashboard.jpeg" alt="tree with dashboard" caption="" %}

Let's create a some simply content for our dashboard
```javascript
import React from 'react'
import styled from 'styled-components'
import Button from 'react-bootstrap/Button'

const StyledDiv = styled.div`{
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: lightGrey;
    height: 100vh;
}`

export class Dashboard extends React.Component{
    render(){
        return (
        <StyledDiv>
            <h1>This is the dashboard page!</h1>
            <Button variant="secondary">Back</Button>
        </StyledDiv>
    )}
}
```

The first step is to install [`react-router-dom`](https://reactrouter.com/web/guides/quick-start). This library is used to create the routing in react apps. 

```sh
yarn add react-router-dom
```

Our App.js file is going to be the one that manages our routing page. We are going to delete the content and create the routes. 
We should have this:

```javascript 
function App() {
  return (
    
  );
}
export default App;
```

We are going to use the components BrowserRouter, Switch and route from `react-router-dom`

<!-- 
The Route 

- Strict: this is going to render the component only if at the end of the path it appears a slash. Ex: /home/ === /home/

Sensitive: Si le pasamos true vamos a tener en cuenta las mayúsculas y las minúsculas de nuestras rutas. Ej: /Home === /Home

Component: Le pasamos un componente para renderizar solo cuando la ubicación coincide. En este caso el componente se monta y se desmonta no se actualiza.

Render: Le pasamos una función para montar el componente en línea. -->
The BrowserRouter gives properties to our component so we can access to the navegation history, do redirections, etc. This component is better for dynamics request (if we want to serve statics request we use HashRouter). 

Switch: this component allows us to render only the first Route child or Redirect that matches with our path. When a <Switch> is rendered, it searches through it's children <Route> elements to find one whose path matches the current URL. When it finds one, it renders that <Route> and ignores all others. This means that you should put <Route>s with more specific (typically longer) paths before less-specific ones. If no <Route> matches, the <Switch> renders nothing (null).

Route: this component renders some UI when its path matches the current URL.
This component has some properties:
- Path: the route where we have to render our component. We can pass a string or array string.
- Exact: this is going to render the component only if the path specify is exactly the same. Ex: /home === /home.

``` javascript
import { BrowserRouter as Router, Switch, Route } from "react-router-dom";
import { LoginPage } from './components/LoginPage'

function App() {
  return (
    <Router>
      <Switch>
        <Route exact path={"/"}>
            <LoginPage />
        </Route>
      </Switch>
    </Router>
  );
}

export default App;
```

Let's add our route for Dashboard.js. For that we should add another <Route > component with the path and compoonent that we want to render. 
``` javascript
import { Dashboard } from './components/Dashboard'

function App() {
  return (
    <Router>
      <Switch>
        <Route exact path={"/"}>
            <LoginPage />
        </Route>
        <Route exact path={"/dashboard"}>
            <Dashboard />
        </Route>
      </Switch>
    </Router>
  );
}

export default App;
```
Let's say that when we click the login button, we want to go to the dashboard.
For that we have to go to our LoginPage.js and in our button we have to add the href with the path that we created. 
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
                    <StyledButton href={"/dashboard"}>Login <FontAwesomeIcon icon={faSignInAlt}/></StyledButton>
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
As you can see we write "/dashboard" im LoginPage and in App.js. Writing like this it could give us some problems, because now we want that our redirect has the name dashboard, but this name can change. And maybe then we wnat to be case senstive. In that case we'll have to change in each file that we wrote it.
We dont want that. So we are going to create a new folder for routes. 

{% include figure image_path="/assets/posts/routes-react/routes-tree.jpeg" alt="tree with routes" caption="" %}