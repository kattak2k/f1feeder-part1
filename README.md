# F1FeederApp-Part1

## Getting Started

All you need to do is to clone this repository,


```
git clone https://github.com/raonibr/f1feeder-part1
cd f1feeder-part1
```

Then, install the dependencies:

```
npm install
```

Then, run the Application:

```
npm start
```

You can access your app at 

```
http://localhost:8000/app/
```
# Angular App-Part1

install the dependencies:
```
npm install
```
Then, run the Application:
```
npm start
```
You can access your app at 
```
http://localhost:8000/app/
```

our HTML might look like:
```
<body ng-app="F1FeederApp" ng-controller="driversController">
  <table>
    <thead>
      <tr><th colspan="4">Drivers Championship Standings</th></tr>
    </thead>
    <tbody>
      <tr ng-repeat="driver in driversList">
        <td>{{$index + 1}}</td>
        <td>
          <img src="img/flags/{{driver.Driver.nationality}}.png" />
          {{driver.Driver.givenName}}&nbsp;{{driver.Driver.familyName}}
        </td>
        <td>{{driver.Constructors[0].name}}</td>
        <td>{{driver.points}}</td>
      </tr>
    </tbody>
  </table>
</body>
```
The first thing you’ll notice in this template is the use of expressions (“{{“ and “}}”) to return variable values. In AngularJS, expressions allow you to execute some computation in order to return a desired value. Some valid expressions would be:
```
{{ 1 + 1 }}
{{ 946757880 | date }}
{{ user.name }}
```
Effectively, expressions are JavaScript-like snippets. But despite being very powerful, you shouldn’t use expressions to implement any higher-level logic. For that, we use directives.

##Understanding Basic Directives

The second thing you’ll notice is the presence of ng-attributes, which you wouldn’t see in typical markup. Those are directives.

At a high level, directives are markers (such as attributes, tags, and class names) that tell AngularJS to attach a given behaviour to a DOM element (or transform it, replace it, etc.). Let’s take a look at the ones we’ve seen already:

The ng-app directive is responsible for bootstrapping your app defining its scope. In AngularJS, you can have multiple apps within the same page, so this directive defines where each distinct app starts and ends.
The ng-controller directive defines which controller will be in charge of your view. In this case, we denote the driversController, which will provide our list of drivers (driversList).
The ng-repeat directive is one of the most commonly used and serves to define your template scope when looping through collections. In the example above, it replicates a line in the table for each driver in driversList.
Adding Controllers

Of course, there’s no use for our view without a controller. Let’s add driversController to our controllers.js:
```
angular.module('F1FeederApp.controllers', []).
controller('driversController', function($scope) {
    $scope.driversList = [
      {
          Driver: {
              givenName: 'Sebastian',
              familyName: 'Vettel'
          },
          points: 322,
          nationality: "German",
          Constructors: [
              {name: "Red Bull"}
          ]
      },
      {
          Driver: {
          givenName: 'Fernando',
              familyName: 'Alonso'
          },
          points: 207,
          nationality: "Spanish",
          Constructors: [
              {name: "Ferrari"}
          ]
      }
    ];
});
```
You may have noticed the $scope variable we’re passing as a parameter to the controller. The $scope variable is supposed to link your controller and views. In particular, it holds all the data that will be used within your template. Anything you add to it (like the driversList in the above example) will be directly accessible in your views. For now, let’s just work with a dummy (static) data array, which we will replace later with our API service.

Now, add this to app.js:
```
angular.module('F1FeederApp', [
  'F1FeederApp.controllers'
]);
```
With this line of code, we actually initialize our app and register the modules on which it depends. We’ll come back to that file (app.js) later on.

Now, let’s put everything together in index.html:
```
<!DOCTYPE html>
<html>
<head>
  <title>F-1 Feeder</title>
</head>

<body ng-app="F1FeederApp" ng-controller="driversController">
  <table>
    <thead>
      <tr><th colspan="4">Drivers Championship Standings</th></tr>
    </thead>
    <tbody>
      <tr ng-repeat="driver in driversList">
        <td>{{$index + 1}}</td>
        <td>
          <img src="img/flags/{{driver.Driver.nationality}}.png" />
          {{driver.Driver.givenName}}&nbsp;{{driver.Driver.familyName}}
        </td>
        <td>{{driver.Constructors[0].name}}</td>
        <td>{{driver.points}}</td>
      </tr>
    </tbody>
  </table>
  <script src="bower_components/angular/angular.js"></script>
  <script src="bower_components/angular-route/angular-route.js"></script>
  <script src="js/app.js"></script>
  <script src="js/services.js"></script>
  <script src="js/controllers.js"></script>
</body>
</html>
```
Modulo minor mistakes, you can now boot up your app and check your (static) list of drivers.

Note: If you need help debugging your app and visualizing your models and scope within the browser, I recommend taking a look at the awesome Batarang plugin for Chrome.

Like what you're reading?Get the latest updates first.

Enter your email address...
Get Exclusive Updates
No spam. Just great engineering posts.
Loading Data From the Server

Since we already know how to display our controller’s data in our view, it’s time to actually fetch live data from a RESTful server.

To facilitate communication with HTTP servers, AngularJS provides the $http and $resource services. The former is but a layer on top of XMLHttpRequest or JSONP, while the latter provides a higher level of abstraction. We’ll use $http.

To abstract our server API calls from the controller, let’s create our own custom service which will fetch our data and act as a wrapper around $http by adding this to our services.js:
```
angular.module('F1FeederApp.services', []).
  factory('ergastAPIservice', function($http) {

    var ergastAPI = {};

    ergastAPI.getDrivers = function() {
      return $http({
        method: 'JSONP', 
        url: 'http://ergast.com/api/f1/2013/driverStandings.json?callback=JSON_CALLBACK'
      });
    }

    return ergastAPI;
  });
  ```
With the first two lines, we create a new module (F1FeederApp.services) and register a service within that module (ergastAPIservice). Notice that we pass $http as a parameter to that service. This tells Angular’s dependency injection engine that our new service requires (or depends on) the $http service.

In a similar fashion, we need to tell Angular to include our new module into our app. Let’s register it with app.js, replacing our existing code with:
```
angular.module('F1FeederApp', [
  'F1FeederApp.controllers',
  'F1FeederApp.services'
]);
```
Now, all we need to do is tweak our controller.js a bit, include ergastAPIservice as a dependency, and we’ll be good to go:
```
angular.module('F1FeederApp.controllers', []).
  controller('driversController', function($scope, ergastAPIservice) {
    $scope.nameFilter = null;
    $scope.driversList = [];

    ergastAPIservice.getDrivers().success(function (response) {
        //Dig into the responde to get the relevant data
        $scope.driversList = response.MRData.StandingsTable.StandingsLists[0].DriverStandings;
    });
  });
```
Now reload the app and check out the result. Notice that we didn’t make any changes to our template, but we added a nameFilter variable to our scope. Let’s put that variable to use.

Filters

Great! We have a functional controller. But it only shows a list of drivers. Let’s add some functionality by implementing a simple text search input which will filter our list. Let’s add the following line to our index.html, right below the <body> tag:
```
<input type="text" ng-model="nameFilter" placeholder="Search..."/>
```
We are now making use of the ng-model directive. This directive binds our text field to the $scope.nameFilter variable and makes sure that its value is always up-to-date with the input value. Now, let’s visit index.html one more time and make a small adjustment to the line that contains the ng-repeat directive:
```
<tr ng-repeat="driver in driversList | filter: nameFilter">
```
This line tells ng-repeat that, before outputting the data, the driversList array must be filtered by the value stored in nameFilter.

At this point, two-way data binding kicks in: every time a value is input in the search field, Angular immediately ensures that the $scope.nameFilter that we associated with it is updated with the new value. Since the binding works both ways, the moment the nameFilter value is updated, the second directive associated to it (i.e., the ng-repeat) also gets the new value and the view is updated immediately.

Reload the app and check out the search bar.

##app search bar

Notice that this filter will look for the keyword on all attributes of the model, including the ones we´re not using. Let’s say we only want to filter by Driver.givenName and Driver.familyName: First, we add to driversController, right below the $scope.driversList = []; line:
```
$scope.searchFilter = function (driver) {
    var keyword = new RegExp($scope.nameFilter, 'i');
    return !$scope.nameFilter || keyword.test(driver.Driver.givenName) || keyword.test(driver.Driver.familyName);
};
```
Now, back to index.html, we update the line that contains the ng-repeat directive:
```
<tr ng-repeat="driver in driversList | filter: searchFilter">
```
Reload the app one more time and now we have a search by name.

##Routes

Our next goal is to create a driver details page which will let us click on each driver and see his/her career details.

First, let’s include the $routeProvider service (in app.js) which will help us deal with these varied application routes. Then, we’ll add two such routes: one for the championship table and another for the driver details. Here’s our new app.js:
```
angular.module('F1FeederApp', [
  'F1FeederApp.services',
  'F1FeederApp.controllers',
  'ngRoute'
]).
config(['$routeProvider', function($routeProvider) {
  $routeProvider.
	when("/drivers", {templateUrl: "partials/drivers.html", controller: "driversController"}).
	when("/drivers/:id", {templateUrl: "partials/driver.html", controller: "driverController"}).
	otherwise({redirectTo: '/drivers'});
}]);
```
With that change, navigating to http://domain/#/drivers will load the driversController and look for the partial view to render in partials/drivers.html. But wait! We don’t have any partial views yet, right? We’ll need to create those too.

##Partial Views

AngularJS will allow you to bind your routes to specific controllers and views.

But first, we need to tell Angular where to render these partial views. For that, we’ll use the ng-view directive, modifying our index.html to mirror the following:
```
<!DOCTYPE html>
<html>
<head>
  <title>F-1 Feeder</title>
</head>

<body ng-app="F1FeederApp">
  <ng-view></ng-view>
  <script src="bower_components/angular/angular.js"></script>
  <script src="bower_components/angular-route/angular-route.js"></script>
  <script src="js/app.js"></script>
  <script src="js/services.js"></script>
  <script src="js/controllers.js"></script>
</body>
</html>
```
Now, whenever we navigate through our app routes, Angular will load the associated view and render it in place of the <ng-view> tag. All we need to do is create a file named partials/drivers.html and put our championship table HTML there. We’ll also use this chance to link the driver name to our driver details route:
```
<input type="text" ng-model="nameFilter" placeholder="Search..."/>
<table>
<thead>
  <tr><th colspan="4">Drivers Championship Standings</th></tr>
</thead>
<tbody>
  <tr ng-repeat="driver in driversList | filter: searchFilter">
    <td>{{$index + 1}}</td>
    <td>
      <img src="img/flags/{{driver.Driver.nationality}}.png" />
      <a href="#/drivers/{{driver.Driver.driverId}}">
	  	{{driver.Driver.givenName}}&nbsp;{{driver.Driver.familyName}}
	  </a>
	</td>
    <td>{{driver.Constructors[0].name}}</td>
    <td>{{driver.points}}</td>
  </tr>
</tbody>
</table>
```
Finally, let’s decide what we want to show in the details page. How about a summary of all the relevant facts about the driver (e.g., birth, nationality) along with a table containing his/her recent results? To do that, we add to services.js:
```
angular.module('F1FeederApp.services', [])
  .factory('ergastAPIservice', function($http) {

    var ergastAPI = {};

    ergastAPI.getDrivers = function() {
      return $http({
        method: 'JSONP', 
        url: 'http://ergast.com/api/f1/2013/driverStandings.json?callback=JSON_CALLBACK'
      });
    }

    ergastAPI.getDriverDetails = function(id) {
      return $http({
        method: 'JSONP', 
        url: 'http://ergast.com/api/f1/2013/drivers/'+ id +'/driverStandings.json?callback=JSON_CALLBACK'
      });
    }

    ergastAPI.getDriverRaces = function(id) {
      return $http({
        method: 'JSONP', 
        url: 'http://ergast.com/api/f1/2013/drivers/'+ id +'/results.json?callback=JSON_CALLBACK'
      });
    }

    return ergastAPI;
  });
```
This time, we provide the driver’s ID to the service so that we retrieve the information relevant solely to a specific driver. Now, we modify controllers.js:
```
angular.module('F1FeederApp.controllers', []).

  /* Drivers controller */
  controller('driversController', function($scope, ergastAPIservice) {
    $scope.nameFilter = null;
    $scope.driversList = [];
    $scope.searchFilter = function (driver) {
        var re = new RegExp($scope.nameFilter, 'i');
        return !$scope.nameFilter || re.test(driver.Driver.givenName) || re.test(driver.Driver.familyName);
    };

    ergastAPIservice.getDrivers().success(function (response) {
        //Digging into the response to get the relevant data
        $scope.driversList = response.MRData.StandingsTable.StandingsLists[0].DriverStandings;
    });
  }).

  /* Driver controller */
  controller('driverController', function($scope, $routeParams, ergastAPIservice) {
    $scope.id = $routeParams.id;
    $scope.races = [];
    $scope.driver = null;

    ergastAPIservice.getDriverDetails($scope.id).success(function (response) {
        $scope.driver = response.MRData.StandingsTable.StandingsLists[0].DriverStandings[0]; 
    });

    ergastAPIservice.getDriverRaces($scope.id).success(function (response) {
        $scope.races = response.MRData.RaceTable.Races; 
    }); 
  });
```
The important thing to notice here is that we just injected the $routeParams service into the driver controller. This service will allow us to access our URL parameters (for the :id, in this case) using $routeParams.id.

Now that we have our data in the scope, we only need the remaining partial view. Let’s create a file named partials/driver.html and add:
```
<section id="main">
  <a href="./#/drivers"><- Back to drivers list</a>
  <nav id="secondary" class="main-nav">
    <div class="driver-picture">
      <div class="avatar">
        <img ng-show="driver" src="img/drivers/{{driver.Driver.driverId}}.png" />
        <img ng-show="driver" src="img/flags/{{driver.Driver.nationality}}.png" /><br/>
        {{driver.Driver.givenName}} {{driver.Driver.familyName}}
      </div>
    </div>
    <div class="driver-status">
      Country: {{driver.Driver.nationality}}   <br/>
      Team: {{driver.Constructors[0].name}}<br/>
      Birth: {{driver.Driver.dateOfBirth}}<br/>
      <a href="{{driver.Driver.url}}" target="_blank">Biography</a>
    </div>
  </nav>

  <div class="main-content">
    <table class="result-table">
      <thead>
        <tr><th colspan="5">Formula 1 2013 Results</th></tr>
      </thead>
      <tbody>
        <tr>
          <td>Round</td> <td>Grand Prix</td> <td>Team</td> <td>Grid</td> <td>Race</td>
        </tr>
        <tr ng-repeat="race in races">
          <td>{{race.round}}</td>
          <td><img  src="img/flags/{{race.Circuit.Location.country}}.png" />{{race.raceName}}</td>
          <td>{{race.Results[0].Constructor.name}}</td>
          <td>{{race.Results[0].grid}}</td>
          <td>{{race.Results[0].position}}</td>
        </tr>
      </tbody>
    </table>
  </div>

</section>
```
Notice that we’re now putting the ng-show directive to good use. This directive will only show the HTML element if the expression provided is true (i.e., neither false, nor null). In this case, the avatar will only show up once the driver object has been loaded into the scope by the controller.
