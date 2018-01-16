---
layout: post
title:      "Moofie with ReactJS & Rails"
date:       2018-01-16 15:35:33 +0000
permalink:  moofie_with_reactjs_and_rails
---

For my React Final Project, my wife came with the idea of a movie finder app where you can search for movies, see their overviews, streaming availabilities and add to your watch list. After an extensive search, I decided to to use '[The Movie Database API](http://https://www.themoviedb.org/)'. Unfortunately, this API did not provide the streaming site availabilities for the movies so I had to fake it. 

The Rails API backend has 2 models: users and movies with a has_many/belongs_to relationship and provides API endpoints for basic CRUD actions. I used 'bcrypt' for user authorization & 'Knock' and 'jwt' to handle user authentication. Basically, all API calls are expected to have a header of 'Authorization: Bearer $jwt_token' from the client. After the user logs in, the user's information (id, username & email) is tokenized, saved in browser's localStorage and sent with the request header. 

The frontend was powered by React. The state was managed by a mix of local states and Redux store. States that do not need to persist in database were handled locally, for eg: query terms, movie lists from search results, sign-in and login form inputs. Redux store handled data that is persisted in the database and consisted of current user information and movies in the list. 

```
#Redux Store state
{
moviesById:
   1: {
       title: '......',
       ...............
       }, 
   2: {
       title: '.......',
        ....................
       }
},
auth: {
   user: {
         id: 1,
         username: 'sally',
         email: 'sally@whatever.com'
         }
     }
}
```
Redux thunk actions were dispatched to get/post movies from/to the backend and also to fetch movie lists and movie details from the Movie DB API. The values returned by the promises were used to dispatch regular redux actions to update the state and error handling.

```
#Redux thunk action to post a movie

export function addMovie (movie) {
  return ({
    type: ADD_MOVIE,
    movie: movie
  })
}

export function postMovie (movie) {
  setAuthorizationToken(window.localStorage.jwt)
  return (dispatch) => {
    return axios({
      method: 'post',
      url: '/api/movies',
      data: {
        movie: movie
      }
    }).then(response => {
      dispatch(addMovie(formatMovieinDB(response.data)))
      return response.data
    }
    )
  }
}
```
React Router 4 components were used to configure the routing of the app.

```
#app.js
<Switch>
      <Route path='/signup' component={SignupForm} />
      <Route path='/login' component={LoginForm} />
      <Route path='/logout' component={LogoutForm} />
      <Route path='/movies' component={Home} />
      <Route exact path='/' render={() => (
        <Redirect to='/movies' />
        )} 
	  />
      <Route render={({location}) => (
        <Header as='h2' style={{margin: '0 0 0 20px'}} textAlign='center' color='red'>
          Oooops!!! Nothing here. Gone Fishing!!!
        </Header>
        )} 
	  />
</Switch>
```
A higher order component was created to prevent users from accessing protected routes, instead redirecting them to the login page.

```
#PrivateRoute higher order function
export default const PrivateRoute = ({component: Component, isAuthenticated, ...rest}) => {
  return (
    <Route {...rest}
      render={(props) => isAuthenticated === true
        ? <Component {...props} />
        : <Redirect to={{pathname: '/login', state: {from: props.location.pathname}}} />
  } />
  )
}
```
Personally, this has been the most involving but also the most fun project in the curriculum. I really came to admire the design pattern of React and how lightweight and responsive it feels. I look forward to learning more advanced concepts in React JS in future.

**GitHub**: https://github.com/sarjumulmi/moofie-app
