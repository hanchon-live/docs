---
title: "Routes React"
date: 2021-05-09T20:18:30+01:00
categories:
  - guides
tags:
  - routes 
  - react-router-dom
  - react
last_modified_at: 2022-04-28T12:59:30+01:00
---

Let's create the `routes` for oura react project. 

We are using the `LandingPage` as an example from the [last guide](/guides/landing-with-styled-components-and-react-bootstrap/) *LandingPage with styled-components*.

{% include toc icon="cog" title="Content" %}


I already created the `Dashboard` component in `components` folder.

# Install react-router-dom

Let's install [react-router-dom](https://reactrouter.com/web/guides/quick-start). 

This `library` is used to create the `routing` in `react apps`. 

```sh
yarn add react-router-dom
```

# Create Route for LandingPage in App.js

Let's create our `routing` in `App.js` file.

We should see something like this:

```react 
function App() {
  return (
    
  );
}
export default App;
```

##  BrowserRouter component from react-router-dom:

`BrowserRouter` allows us to access to the `navigation history`, do `redirections`, etc. 

This component is better for `dynamics request` (if you want to serve `statics request` you should use `HashRouter`). 

```react
import { BrowserRouter as Router } from "react-router-dom";

function App() {
  return (
    <Router>
    </Router>
  );
}

export default App;
```
##  Switch component from react-router-dom:

`Switch` allows us to render `only` the` first Route child` or `Redirect` that `matches` with our path. 

When a `<Switch>` is rendered, it `searches` through it's children `<Route> elements `to find one whose path `matches` the `current URL`. 

When it finds one, it `renders` that `<Route>` and ignores all others. 

This means that you should put `<Route>s` with more `specific` (typically longer) `paths` before less-specific ones. If no `<Route>` matches, the `<Switch>` renders nothing (null).

``` react
import { Switch } from "react-router-dom";
function App() {
  ...
    <Router>
      <Switch>
      </Switch>
    </Router>  
  ...
}
export default App;
```
##  Route component from react-router-dom:

`Route` renders some UI when its `path` `matches` the `current URL`.

This component has some properties:

- Path: the route where we have to `render` our component. We can pass a string or array string.
- Exact: this is going to render the component `only` if the `path` specify is `exactly` the `same`. Example: /home === /home.

``` react
import { Route } from "react-router-dom";
import { LoginPage } from './components/LoginPage'

function App() {
  ...
    <Switch>
      <Route exact path={"/"}>
        <LoginPage />
      </Route>
    </Switch>
  ...
}

export default App;
```
# Create Route for Dashboard in App.js

Let's create the `route` for `Dashboard` by adding another `<Route>` component with the `path` and the `component` to render.

``` react
import { Dashboard } from './components/Dashboard'

function App() {
  ...
    <Route key="home" exact path={"/"}>
        <LoginPage />
    </Route>
    <Route key="dashboard" exact path={"/dashboard"}>
        <Dashboard />
    </Route>
  ...
  );
}

export default App;
```

# Redirect to URL when click a button

Let's go to the `Dashboard` page when we click in the `login button` (in `LoginPage.js`) and to the `LoginPage` when we click on `back button` (in `Dashboard.js`)

In `LoginPage.js`, let's add `href` with the `path` that we created.
```react
export class LoginPage extends React.Component{
    render(){
        return(
        ...
          <RightColumnSpaces>
              <StyledButton href={"/dashboard"}>Login <FontAwesomeIcon icon={faSignInAlt}/></StyledButton>
          </RightColumnSpaces>
        ...
        )
    }
}
```

## Create constants for paths:

We wrote `"/dashboard"` in `LoginPage.js` and in `App.js`. 

If we have the same text used in several files, we should create constants. This way you avoid writing problems. 

### Create file for Routes constants:

Let's create the file for our `constants` (src/routes/Routes.js). 

``` sh
cd src
mkdir routes
touch Routes.js
```

### Define constants:

In `routes.js` let's create a constant and export it.
```javascript
const routes = {
  home: '/',
  dashboard: '/dashboard',
}

export default routes;
```
### Use the constants in the files:

Let's use our constants. First import the constants and then replace them.

#### In App.js:
import Routes from "./routes/routes";
```react

function App() {
  ...
    <Route key="home" exact path={Routes.home}>
        <LoginPage />
    </Route>
    <Route key="dashboard" exact path={Routes.dashboard}>
        <Dashboard />
    </Route>
  ...
  );
}

export default App;
```
#### In Landing.js:
```react
import Routes from "../routes/routes";
export class LandingPage extends React.Component{
  ...
   <RightColumnSpaces>
      <StyledButton href={Routes.dashboard}>Login <FontAwesomeIcon icon={faSignInAlt}/></StyledButton>
    </RightColumnSpaces>
  ...
}
```

#### In Dashboard.js:
```react
import Routes from '../routes/routes'
export class Dashboard extends React.Component{
  ...
   <Button href={Routes.home} variant="secondary">Back</Button>
  ...
}
```