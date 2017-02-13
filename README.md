# ld-redux

[![npm version](https://img.shields.io/npm/v/ld-redux.svg?style=flat-square)](https://www.npmjs.com/package/ld-redux) [![npm downloads](https://img.shields.io/npm/dm/ld-redux.svg?style=flat-square)](https://www.npmjs.com/package/ld-redux) [![npm](https://img.shields.io/npm/dt/ld-redux.svg?style=flat-square)](https://www.npmjs.com/package/ld-redux) [![npm](https://img.shields.io/npm/l/ld-redux.svg?style=flat-square)](https://www.npmjs.com/package/ld-redux) 

> **A library to integrate launch darkly with react redux** :clap:

[Launch Darkly](https://launchdarkly.com/faq.html) is a great tool for feature flagging and a/b testing. It has a fully capable [client-side javascript sdk](https://github.com/launchdarkly/js-client), so why this package? 

If you use react redux and you want to store your feature flags as part of your redux state, this package will do that for you. It does the heavy lifting of:

 * Fetching your flags from launch darkly.
 * Storing it in your redux state.
 * Camel casing your keys so you can use them in code with the dot operator. The keys by default are dash separated so you can't do this out of the box with the [official sdk](https://github.com/launchdarkly/js-client).
 * Server Sent Event works as well so your app will respond live to feature flag changes without the users having to refresh the browser!

## Installation

yarn add ld-redux

or the old school way

npm i --save ld-redux

## Quickstart

1. In your client bootstrap, initialise the launch darkly client by invoking initLD method:

    ```javascript
    import createStore from '<your-project>/store';
    import ldRedux from 'ld-redux';
    
    // standard redux createStore
    const store = createStore();
    
    // Pass redux store to ld-redux so it can store flags in redux state
    ldRedux.init('yourClientSideId', store);
    
    render(
      <Provider store={store}>
        <Router routes={routes} history={browserHistory}/>
      </Provider>,
      document.getElementById('reactDiv')
    );
    ```

2. Include ldReducer as one of the reducers in your app:

    ```javascript
    import { combineReducers } from 'redux';
    import ldRedux from 'ld-redux';
    import reducers from '<your-project>/reducers'; 
   
    export default combineReducers({
      ...reducers,
      LD: ldRedux.reducer(), // Note: key must be upper case LD
    });
    ```

3. Subscribe to flags in your redux container:
    
    ```javascript
    import {connect} from 'react-redux';
    import ldRedux, {ldConnect} from 'ld-redux';
    import * as yourActions from '<your-project>/actions/yourActions';
    
    // These must be the keys you set up in launch darkly dashboard (kebab-lower-cased)
    const homeFlags = {'feature-flag-key': false};
    
    const mapStateToProps = (state) => {
      // Use getFlags to access flags from LD state
      const flags = ldRedux.getFlags(state, homeFlags);
    
      return {
        ...flags,
      };
    };
    
    // Use ldConnect to connect your component to the feature flags it needs
    @connect(mapStateToProps, yourActions)
    @ldConnect(homeFlags)
    export default class HomeContainer extends Component {
      render() {
        return <HomeComponent {...this.props} />;
      }
    };
    ```
    
4. Finally in your component, your feature flags are available from props in camelCase:

    ```javascript
    import React, {Component} from 'react';
    
    export default class Home extends Component {
      render() {
        return (
          <div>
            {
              /* featureFlagKey is camelCased & injected by ld-redux */
              this.props.featureFlagKey ?
                <div>
                  <p>Welcome to feature toggling!</p>
                </div>
                :
                'nothing'
            }
          </div>
        );
      }
    }
    ```

## Advanced API
### window.ldClient
The ldRedux.init method (see step 1) initialises the js sdk and stores the resultant object in window.ldClient. You can use 
this object to access the [official sdk methods](https://github.com/launchdarkly/js-client) directly. For example, you can do things like: 

```javascript
// track goals
window.ldClient.track('add to cart');

// change user context
window.ldClient.identify({key: 'someUserId'});
```

For more info on changing user context, see the [official documentation](http://docs.launchdarkly.com/docs/js-sdk-reference#section-changing-the-user-context).

### isLDReady 
This is a boolean flag in LD reducer which gets set to true when the sdk has finished loading. You can subscribe to this state if you 
need to perform custom operations on component load. By default, the ldRedux.getFlags method injects this flag implicitly so if you follow
step 3 above, you'll find that isLDReady is already available as props to your component.


## Example
Check the [example](https://github.com/yusinto/ld-redux/tree/master/example) for a fully working spa with react, redux and react-router. 
