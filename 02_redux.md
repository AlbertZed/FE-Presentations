# Why Redux

* From a purely technology standpoint, I don’t feel strongly that Redux is necessarily better than other options.
* What Redux did do is popularize a lot of important design patterns that useful when constructing React applications
  * Separating view from business logic AKA dumb components vs smart components aka regular components vs containers

## Higher Order Components
(Derived from higher order functions - functional programming)

```js
import React, {Component} from 'react';
import statusStore from './statusStore'; //some reflux store

class NonSeparatedComponent extends Component {
  constructor(props) {
    super(props);
    this.state = { valueFromStore: false};
    this.onStoreChange = this.onStoreChange.bind(this);
  }

  onStoreChange(newState) {
    this.setState({valueFromStore: newState});
  }

  componentDidMount() {
    this.unsubscribe = statusStore.listen(this.onStoreChange);
  }

  componentWillUnmount() {
    this.unsubscribe();
  }

  render() {
    return <div>The value is: {this.state.valueFromStore}</div>;
  }
}

// <NonSeparatedComponent />


function StatusStoreHOC(WrappedComponent) {

  return StatusStoreComponent extends Component {
    constructor(props) {
      super(props);
      this.state = { valueFromStore: false};
      this.onStoreChange = this.onStoreChange.bind(this);
    }

    onStoreChange(newState) {
      this.setState({valueFromStore: newState});
    }

    componentDidMount() {
      this.unsubscribe = statusStore.listen(this.onStoreChange);
    }

    componentWillUnmount() {
      this.unsubscribe();
    }

    render() {
      return (<WrappedComponent valueFromStore={this.state.valueFromStore} />)
    }
  }
}

class PresentationOnlyComponent extends Component {
  render() {
    return <div>The value is: {this.props.valueFromStore}</div>;
  }
}

const PresentationWrappedComponent = StatusStoreHOC(PresentationOnlyComponent);

// Usage: <PresentationWrappedComponent />

function PresentationOnlyComponentAsFunction(props) {
  return <div>The value is: {props.valueFromStore}</div>;
}

const PresentationFunctionWrappedComponent = StatusStoreHOC(PresentationOnlyComponentAsFunction);

// Usage: <PresentationFunctionWrappedComponent />
```

### How Redux handles separating presentational and container components

```js
//import { connect } from 'react-redux';
import { loadWeather } from '../actions';
import WeatherList from '../components/weatherList';

const mapStateToProps = (state) => {
  return {
    weather: state.weather
  }
}

//my BS attempt at mocking what `connect` does
function connect(mapStateToProps, actionCreators) {
  return (WrappedComponent) => {
    return class ConnectedComponent extends Component {
      constructor() {
        //does a bunch of stuff
      }
      render() {
        return (<WrappedComponent {...props} {...mapStateToProps()} {...actionCreators} />)
      }
    }
  }
}

export default connect(mapStateToProps, { loadWeather })(WeatherList);
```
* **SHOW THE ReactReduxStarter** git branch feature/open-weather

* Redux Ecosystem
  * Reselect - **maybe show them The RNNYT Book app**
  * Middleware / Redux Thunk / Redux Promise
  * 
