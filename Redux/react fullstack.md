# React Full Stack

## Dependencies

### Client

- **Framework** - React
- **State Container** - Redux, react-redux
- **Router** - react-router, react-router-redux (Custom the rootReducer to support Immutable)
- **Action Creator** - redux-actions
- **Side Effect** - redux-saga
- **Immutable State** - Immutable
- **Reducer Immutable** - redux-immutable
- **Persist Immutable Store** - redux-persist-immutable
- **Selector** - reselect
- **State Normalize** - normalizr
- **Dynimic Import** - react-loadable
- **UI** - Ant Design
- **Form**- redux-form

### Server
- **Framework** - express, [koa2]
- **Database ODM** - mongoose
- **Session** - expresss-session
- **Authorization** - passport, passport-local, [passport-jwt]

## Develop Dependencies

- **ES2015** - Babel
- **Flow & Bundle** - Webpack
- **Lint** - eslint
- **Test** - Jest, supertest
- **Type** - Flow
- **Process Manager** nodemon, pm2

## How to use normalizr with Redux

State would look like:

```js
{
  entities: {
    plans: {
      1: {title: 'A', exercises: [1, 2, 3]},
      2: {title: 'B', exercises: [5, 1, 2]}
     },
    exercises: {
      1: {title: 'exe1'},
      2: {title: 'exe2'},
      3: {title: 'exe3'}
    }
  },
  currentPlans: [1, 2]
}
```

reducers might look like:

```js
import merge from 'lodash/object/merge';

const exercises = (state = {}, action) => {
  switch (action.type) {
  case 'CREATE_EXERCISE':
    return {
      ...state,
      [action.id]: {
        ...action.exercise
      }
    };
  case 'UPDATE_EXERCISE':
    return {
      ...state,
      [action.id]: {
        ...state[action.id],
        ...action.exercise
      }
    };
  default:
    if (action.entities && action.entities.exercises) {
      return merge({}, state, action.entities.exercises);
    }
    return state;
  }
}

const plans = (state = {}, action) => {
  switch (action.type) {
  case 'CREATE_PLAN':
    return {
      ...state,
      [action.id]: {
        ...action.plan
      }
    };
  case 'UPDATE_PLAN':
    return {
      ...state,
      [action.id]: {
        ...state[action.id],
        ...action.plan
      }
    };
  default:
    if (action.entities && action.entities.plans) {
      return merge({}, state, action.entities.plans);
    }
    return state;
  }
}

const entities = combineReducers({
  plans,
  exercises
});

const currentPlans = (state = [], action) {
  switch (action.type) {
  case 'CREATE_PLAN':
    return [...state, action.id];
  default:
    return state;
  }
}

const reducer = combineReducers({
  entities,
  currentPlans
});
```

**First, note that the state is normalized. We never have entities inside other entities. Instead, they refer to each other by IDs. So whenever some object changes, there is just a single place where it needs to be updated.**