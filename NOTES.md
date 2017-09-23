
```html
<html ng-app>
<head>
</head>
<body>
<div ng-init="hello='world'">
<h1>{{hello}}</h1>
<p><input type="text" ng-model="hello"></p>
</div>
</body>
</html>
```

* module
    * controller
        * view

```
<html ng-app="bukmarks">
```

```
angular.module('bukmarks', [
    // array of dependencies.
])
.controller('MainCtrl', function($scope) {
    // once you attach stuff to $scope, then they are
    // available in the view for binding.
    $scope.hello = 'world';

    // $scope is the glue between the controller and the view.
});
```

```
<body ng-controller="MainCtrl">
    <h1>{{hello}}</h1>
</body>
```

```
<ul>
    <li ng-repeat="category in categories" ncClass="{'active':isCurrentCategory(category)}">
        <a href="#">{{category.name}}</a>
    </li>
</ul>
```

-----------------

```
ctrl:
    $scope.currentCategory = null;

    function setCurrentCategory(category) {
        scope.currentCategory = category;
    }

    function isCurrentCategory(category) {
        return $scope.currentCategory !== null && category.name === $cope.currentCategory.name;
    }

    // Now the method is available to the view.
    $scope.setCurrentCategory = setCurrentCategory;
```

```
<a href="#" ng-click="setCurrentCategory(category)">{cateogory.name}</a>

<div ng-repeat="bookmark in bookmarks | filter:{category:currentCategory.name}"></div>
```

```
$scope.creating = false;
$scope.editing = false;
```

```
function shouldShowCreating() {
    return $scope.currentCategory && !$scope.editing;
}

function shouldShowEditing() {
    return $scope.editing && !$scope.creating;
}

function setEditedBookmark(bookmark) {
    $scope.editedBookmark = angular.copy(bookmark);
}

function updateBookmark(bookmark) {
    const index = _.findIndex($scope.bookmarks, function(b) {
        return b.id === bookmark.id;
    });

    $scope.bookmarks[index] =bookmark;

    $scope.editedBookmark = null;
    $scope.isEditing = false;
}

$scope.updateBookmark = updateBookmark;

$scope.shouldShowCreating = shouldShowCreating;
$scope.shouldShowEditing = shouldShowEditing;
```

```
<div ng-if="shouldShowCreating()"></div>
<div ng-if="shouldShowEditing()"></div>

<input ng-model="newBookmark.title" placeholder="Enter title">
```

Controller and the $scope together can be considered as a ViewModel.

Any change in the view is bidirectionally bound to the ViewModel with the databinding glue of the $scope.
Any change in the scope similarly (hence bidirectional) reflected to the view.

```
Model (database|services) <-> ViewModel (Controller|$scope) <-> View
```

----------------------------

Goal: Move the business logic to the models so that controller and views become lightweight and **specific**.

## File Structure

Organize the code by features, not by type (*i.e. don’t move all controllers into a single “controllers” folde*r).

```
app
    categories
        bookmarks
            create
                bookmark-create.js
                bookmark-create.tmpl.html
            edit
                bookmark-edit.js
                bookmark-edit.tmpl.html
            bookmarks.js
            bookmarks.tmpl.html
        categories.js
        categories.tmpl.html
    common
        models
            bookmarks-model.js
            categories-model.js
```

Modules are just containers that separate your code by features.

```
angular.module('app.models.bookmarks', [])
    .service('BookmarksModel', function() {
        const model = this;

        const bookmarks = [/* … bookmarks go here … */];

        model.getBookmarks = function() {
            return bookmarks;
        };
    });
```

```
angular.module('app.models.categories', [])
    .service('CategoriesModel', function($http) {
        const model = this;

        const URLS = {
            FETCH: 'data/categories.json'
        };

        categories = [/*content*/];

        model.getCategories = function() {
            // return categories;

            return $http.get(URLS.FETCH);
        };
    });
```

```
// MODEL is just a simple module with a state (i.e. injected $stateProvider)
angular.module('categories', [
    'app.models.categories'
])
    .config(function($stateProvider) {
         // ROUTES associated with the MODEL
         $stateProvider
            .state('app.categories', {
                url: '/',
                views: {
                    // @ symbol makes it an absolute path.
                    'categories@': {
                        controller: 'CategoryListCtrl as categories',
                        templateUrl: 'app/categories/categories.tmpl.html'
                    }, 
                    'bookmarks@': {
                        controller: 'BookmarksCtrl',
                        templateUrl: 'app/categories/bookmarks/bookmarks.tmpl.html'
                    }
                }
            });
    })
    // CONTROLLERS associated with the model.
    .controller('CategoryListCtrl', function(CategoriesModel) {
        var ctrl = this;

        CategoriesModel.getCategories().then(function (res) {
            ctrl.categories = res.data;
        });
    });
```

```
angular.module('categories.bookmarks', [
    'categories.bookmarks.create',
    'categories'bookmarks.edit',
    'app.models.categories',
    'app.models.bookmarks'
])
    .config(function($stateProvider) {
        $stateProvider
            .state('app.categories.bookmarks', {
                url: 'categories/:category',
                views: {
                    'bookmarks@': {
                        templateUrl: 'app/categories/bookmarks/bookmarks.tmpl.html',
                        controller: 'BookmarksCtrl as bookmarkListCtrl'
                    }
                }
            });


        // state.go
        // sref
    })
    // $stateParams is a service.
    .controller('BookmarkListCtrl', function($stateParams, BookmarksModel, CategoriesModel) {
        var bookmarkListCtrl = this;

        // bookmarkListCtrl.currentCategoryName = $stateParams.category;
        CategoriesModel.setCurrentCategory($stateParams.category);

        // bookmarkListCtrl.bookmarks = BookmarksModel.getBookmarks();

        BookmarksModel.getBookmarks().then(function(bookmarks) {
            bookmarkListCtrl.bookmarks = bookmarks;
        });

        bookmarkListCtrl.getCurrentCategory = CategoriesModel.getCurrentCategory;
        bookmarkListCtrl.getCurrentCategoryName = CategoriesMode.getCurrentCategoryName;
    });
```

```
angular.module('categories.bookmarks.edit', [])
    .config(function($stateProvider) {
        $stateProvider.state('app.categories.bookmarks.edit', {
            url: '/bookmarks/:bookmarkId/edit',
            templateUrl: 'app/categories/bookmarks/create/bookmark-edit.tmpl.html',
            controller: 'EditBookmarkCtrl as editBookmarkCtrl'
        });
    })
    .controller('EditBookmarkCtrl', function() {

    });
```

```
// define a module that defines some routes links views to those routes and binds
// a controller to those routes.
// 
// The .module, .config, .controller etc methods return the original object for chaining.
// So whenever you define a module, anything you chain to it is bound to that module.
// Knowing this, can help you split out the code further if there is a need.
//
// module config controller service
//
angular.module('categories.bookmarks.create', [])
    .config(function($stateProvider) {
        $stateProvider.state('app.categories.bookmarks.create', {
            url: '/bookmarks/create',
            templateUrl: 'app/categories/bookmarks/create/bookmark-create.tmpl.html',
            controller: 'CreateBookmarkCtrl as createBookmarkCtrl'
        });
    })
    .controller('CreateBookmarkCtrl', function($state, $stateParams, BookmarksModel) {
        const ctrl = this;

        function returnToBookmarks() {
            $state.go('app.categories.bookmarks', {category: $stateParams.category});
        }

        function cancelCreating() {
            returnToBookmarks();
        };

        function createBookmark(bookmark) {
            BookmarksMode.createBookmark(bookmark);

            returnToBookmarks();
        }

        function resetForm() {
            ctrl.newBookmark = {
                title: '',
                url: '',
                category: $stateParams.category
            }
        }

        resetForm();

        ctrl.cancelCreating = cancelCreating;
        ctrl.createBookmark = createBookmark;
    });

```

```
angular.module('app', [
    'ui.router',
    'categories',
    'categories.bookmarks'
])
    .config(function($stateProvider, $urlRouterProvider) {

        // state machine.
        // $stateProvider
        //    .state('app', {
        //        url: '/',
        //        templateUrl: 'app/categories/categories.tmpl.html'
        //        controller: 'MainCtrl'
        //    });

        $stateProvider
            // an abstract state is a state that can have child states but
            // cannot be activated itself.
            .state('app', {
                 url: '',
                 abstract: true
            });

        // url router provider redirecting to the root of the application.
        $urlRouterProvider.otherwise('/');
    })
    .controller('MainCtrl', function($scope, $state) {
        function setCurrentCategory(category) {
            // $state.go('app.categories.bookmarks', {category: category.name});
        };
    });
```

```
<h1>{{bookmarks.currentCategoryName}}</h1>
```

```
// ui-sref is a directive. $state is a service.
<a ui-sref="app.categories.bookmarks({category:category.name})" ng-click="setCurrentCategory(category)">{{category.name}}</a>
```


```
<div ui-view></div>
```

```
<div>
    <div ui-view="categories"></div>
    <div ui-view="bookmarks"></div>
</div>
```

## Scope Inheritance

Scope objects prototypally inherit from their parent objects all the way up to the root scope.


