# Connecting the dots - step 4

It's time to connect the dots and get the two applications to talk to each other.

First we need to deploy the Rails application to a production server. As usual, we'll be using the simplest solution available.

Let's create a Heroku application

```
$ heroku create ca-cooper-api
  # where 'ca' are your initials
```

Now let's create a database and push the app to Heroku.

```
$ heroku addons:create heroku-postgresql
$ git push heroku master
$ heroku run rake db:migrate
```

You can manually test the API using [Postman](https://www.getpostman.com/) by doing a POST request to the registration endpoint.

**As a matter of fact, you need to create at least one user this way in order to move forward.**

![Register a user](/images/cooper_api_postman_sucess.png)  
And if you try to send the request again, that should fail.  
![Registration failure](/images/cooper_api_postman_failure.png)

Alright, if that works we should shift our focus to the Ionic application.

We will be using [ng\_token\_auth](https://github.com/lynndylanhurley/ng-token-auth) - a token based authentication module for AngularJS that works really well with `devise_token_auth`.

We'll start by installing the library using Bower. Run the install command from your Terminal.

```
$ bower install ng-token-auth --save
```

Make sure that `angular-cookie`, and `ng-token-auth` are included in your `index.html`.

!FILENAME www/index.html

```html
<!-- ionic/angularjs js -->
<script src="lib/ionic/js/ionic.bundle.js"></script>
<script src="lib/angular-cookie/angular-cookie.js"></script>
<script src="lib/ng-token-auth/dist/ng-token-auth.js"></script>
```

Include `ng-token-auth` in your module's dependencies and add a basic configuration.

!FILENAME www/js/app.js

```javascript
angular.module('starter', ['ionic', 'starter.controllers', 'ng-token-auth'])
    .constant('API_URL', 'https://ca-cooper-api.herokuapp.com/api/v1')

  .config(function ($authProvider, API_URL) {
    $authProvider.configure({
      apiUrl: API_URL
    });
  })
```

In `controllers.js` `AppCtrl` locate the `doLogin()` method.

!FILENAME www/js/controllers.js

```javascript
//...
// Perform the login action when the user submits the login form
$scope.doLogin = function () {
  $auth.submitLogin($scope.loginData)
    .then(function (resp) {
      // handle success response
      $scope.closeLogin();
    })
    .catch(function (error) {
      // handle error response
      $scope.errorMessage = error;
    });
};
//...
```

Now, we need to make some small changes and additions to our view templates.

Change the input field `Username` to `Email` and the `ng-model` from `loginData.username` to `loginData.email`.

!FILENAME www/templates/login.html

```html
//...
<label class="item item-input">
  <span class="input-label">Email</span>
  <input type="text" ng-model="loginData.email">
</label>
//...
```

And add a placeholder for display of error messages.

!FILENAME www/templates/login.html

```html
//...
<ion-content>
  <div class="row" ng-if="errorMessage">
    <p ng-repeat="error in errorMessage.errors">
      {{error}}
    </p>
  </div>
  //...
```

We also want to create a `currentUser` object. In the`AppCtrl` add this method to create the `currentUser` on successful authentication.

!FILENAME www/js/controllers.js

```javascript
//...
$rootScope.$on('auth:login-success', function(ev, user) {
  $scope.currentUser = user;
});
//...
```

And make use of this object on the `about.html` template.

!FILENAME www/templates/about/about.html

```html
//...
 <div class="col">
    Craft Academy Cooper Test Challenge - Mobile Client.
    <p ng-if="currentUser">Logged in as {{currentUser.email}}</p>
 </div>
//...
```

One final touch to enhance the user experience. Sometimes the API endpoint will take some time to respond and the user might be left wondering if his request is actually being processed or not. There is a very simple way to show the user that we are really processing his request by displaying an overlay with some sort of a message. In our case "Logging is..." could do, right?

Ionic provides us with [`$ionicLoading`](http://ionicframework.com/docs/api/service/$ionicLoading/) as a way to display such overlays. Let's implement it.

Add `$ionicLoading` to the `AppCtrl`.

!FILENAME www/js/controllers.js

```javascript
//...
.controller('AppCtrl', function ($rootScope,
                                 $scope,
                                 $ionicModal,
                                 $timeout,
                                 $auth,
                                 $ionicLoading) {
//...
```

And update the `doLogin()` function with the following code.

!FILENAME www/js/controllers.js

```javascript
//...
// Perform the login action when the user submits the login form
  $scope.doLogin = function () {
    $ionicLoading.show({
      template: 'Logging in...'
    });
    $auth.submitLogin($scope.loginData)
      .then(function (resp) {
        // handle success response
        $ionicLoading.hide();
        $scope.closeLogin();
      })
      .catch(function (error) {
        $ionicLoading.hide();
        $scope.errorMessage = error;
      });
  };
//...
```

With this in place an overlay will be displayed while the app is making the request and hidden when the promise is resolved.

At this stage we have a method to login the user. The next step will be to add an interface to create and update users. That is, however, something that we leave up to you. Just a friendly reminder, make sure that you read the [ng\_token\_auth](https://github.com/lynndylanhurley/ng-token-auth) documentation. Everything you need to know is well documented in the README file of the project.

