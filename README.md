# Angular Directives

## Objectives

- Understand the value of `ng-repeat`
- use `ng-if`, `ng-switch` and `ng-class`
- find the various event directives available such as `ng-focus`
- use event directives to call methods in a controller

## Overview of Directives

Within Angular.js, directives serve as the building blocks, or components, with which you can assemble your applications. These could be as simple as merely switching classes on and off (as we'll see with `ng-class`) or as complicated as a multi-step wizard that generates a custom data model such as a test or user profile. In all cases, directives attach a specific bit of javascript, and the functionality that comes with that code, to individual nodes, or group of nodes, within your HTML.

For this lesson, we'll focus on some of the built-in directives that come with Angular and how to utilize them. Though they can be attached to HTML nodes in multiple ways, we'll only use the "attribute" method, as it's both the most common and versatile of the methods available, and it also is enabled by default on all built-in and custom directives. To attach a directive in this manner, all you have to do is add the directive name as an attribute on the HTML element, optionally passing along additional data.

For example, to attach a directive named `test-directive`, you simply add it to the HTML element like this:

```html
<div test-directive><!-- element content --></div>
```

And if you want to pass along data to the directive, you can use the attribute value, such as:

```html
<div test-directive="myValue"><!-- element content --></div>
```

In most cases, the value that you pass in to the directive will be a "scoped" property, much like you might utilize within the HTML directly via the double-bracket (`{{myValue}}`) syntax elsewhere. While there are exceptions to this, for this lesson all values will be scoped this way, and we'll learn about the other options later. One point of note, when naming directives, follow the casing conventions of the respective language. That means with javascript, use lowerCamelCase, such as `testDirective`, but when actually attaching it within the HTML, use lower-dasherized casing, ergo `test-directive`.

## `ng-repeat`

As one of the most commonly used directives, `ng-repeat` allows us to iterate over a collection, either an array or an object, and repeat one or more HTML nodes for each item within that collection. For each iteration, the scope is given a local variable that corresponds to the current element of the collection, thus rendering the same template differently depending on the data.

In each case, when we add the attribute `ng-repeat` to an element, the value we pass in follows a particular syntax. For arrays, that's `label in collection` and for objects it's `(key, value) in collection`. This tends to make more sense when you actually see it, so as an example, given a base controller with two scoped collections:

```javascript
angular.module('myApp')
    .controller('myCtrl', function ($scope) {
        $scope.people = [
            {
                id: 1,
                name: "John Smith",
                role: "Manager"
            },
            {
                id: 2,
                name: "Jane Doe",
                role: "CFO"
            }
        ];

        $scope.departments = {
            "hr": "Human Resources",
            "fin": "Finance"
        }
    });
```

We could render them out as follows:

```html
<div class="employees">
    <p ng-repeat="employee in people">
        {{employee.name}}
        (<span class="role">{{employee.role}}</span>)
    </p>
</div>

<div class="departments">
    <p ng-repeat="(key, label) in departments">
        {{label}}
        (<span class="code">{{key}}</span>)
    </p>
</div>
```

In each case, Angular will iterate through either the items of the array or the keys (or properties) of an object and repeat the template for each item. One important note is that while an array will render in the order of the original array, because javascript doesn't guarantee ordering on an object, Angular will always render objects in alphabetical order by the key names, so as to ensure the same output across all browsers and instances of your app.

## `ng-class`

Consider our list above and imagine you want to set a particular class on each employee's paragraph tag depending on the role they have. To do this, you can use the `ng-class` directive which will dynamically add or remove one or more classes from an element based on any condition you wish. For example, to set the class based on the role, all we have to do is modify our paragraph tag thus:

```html
<p ng-repeat="employee in people" ng-class="employee.role">
    {{employee.name}}
    (<span class="role">{{employee.role}}</span>)
</p>
```

Now, in a real app, you might want to ensure the role is lowercased, or certainly doesn't have any spaces, but that's an exercise for another time. Consider now, however, that instead of just outputting the role, you want a couple different classname options based on some other value, such as `bonus-eligible`. Perhaps Jane, as an executive, is excluded from the standard bonus payouts, whereas John, who is a mid-level manager, still qualifies:

```javascript
$scope.people = [
    {
        id: 1,
        name: "John Smith",
        role: "Manager",
        bonusEligible: true
    },
    {
        id: 2,
        name: "Jane Doe",
        role: "CFO",
        bonusEligible: false
    }
];
```

For this, we can pass in a map, or object hash, to `ng-class`, where the key names will be used as classes, and the property values are the conditionals. If the value evaluates to something "truthy", the class is added. If not, the class is removed.

```html
<p ng-repeat="employee in people" ng-class="{eligible: employee.bonusEligible, ineligible: !employee.bonusEligible}">
    {{employee.name}}
    (<span class="role">{{employee.role}}</span>)
</p>
```

## `ng-if`

Suppose, however, instead of just setting a class based on eligibility, we want to actually completely remove any employees from our list that aren't eligible. For this, we can use `ng-if`, which also evaluates a conditional as "truthy" or "falsey", but in this case if the result is falsey, the element is removed the DOM completely:

```html
<p ng-repeat="employee in people" ng-class="employee.role" ng-if="employee.bonusEligible">
    {{employee.name}}
    (<span class="role">{{employee.role}}</span>)
</p>
```

As a general rule, it's best to keep any logic within your HTML as simple as possible, like we have above with a simple boolean check. However, it is worth noting that conditional statements within Angular can be just as complicated as any within standard javascript, though of course all variables are evaluated within the current scope. For example, if instead of eligibility we wanted to look at all employees who had been employed at least a year, we could update our array and HTML as follows:

```javascript
$scope.people = [
    {
        id: 1,
        name: "John Smith",
        role: "Manager",
        monthsEmployed: 10
    },
    {
        id: 2,
        name: "Jane Doe",
        role: "CFO",
        monthsEmployed: 28
    }
];
```

```html
<p ng-repeat="employee in people" ng-class="employee.role" ng-if="employee.monthsEmployed >= 12">
    {{employee.name}}
    (<span class="role">{{employee.role}}</span>)
</p>
```

## `ng-switch`

There are, of course, times when a simple boolean conditional isn't enough. Perhaps, depending on the role of the employee, you want to render a slightly different template for their paragraph. In this case, `ng-switch` comes into play. Like a standard switch statement within javascript, this directive allows you to specify a variable or property to evaluate, and then provide one or more "cases" to compare it to, along with an optional default if nothing else matches. Consider a case where, if the employee is a manager, you want to display the number of direct reports they have; if a base employee, you want to display the name of their manager; and otherwise, just render the template as we've seen already. In this case, our list of employees updates to:

```javascript
$scope.people = [
    {
        id: 1,
        name: "John Smith",
        role: "Manager",
        directReports: 5
    },
    {
        id: 2,
        name: "Jane Doe",
        role: "CFO"
    },
    {
        id: 3,
        name: "Bobby Tables",
        role: "Employee",
        manager: "John Smith"
    }
];
```

And our HTML becomes:

```html
<div ng-repeat="employee in people" ng-switch on="employee.role">
    <p ng-class="employee.role" ng-switch-default>
        {{employee.name}}
        (<span class="role">{{employee.role}}</span>)
    </p>
    <p ng-class="employee.role" ng-switch-when="'Manager'">
        {{employee.name}}
        (<span class="role">{{employee.role}} of {{employee.directReports}} reports</span>)
    </p>
    <p ng-class="employee.role" ng-switch-when="'Employee'">
        {{employee.name}}
        (<span class="role">{{employee.role}}, managed by {{employee.manager}}</span>)
    </p>
</div>
```

There are couple key things to note. First, we added a wrapping `div` tag for each employee. That's often a necessary side effect of using `ng-switch`, as we need one element for the actual switch directive, and then one or more elements inside for each of the possible templates. Secondly, while we put the default first in this case, that's not necessary, and in general order doesn't matter, though like a true switch statement, whichever "when" clause matches first will be what's used, so if it's possible that more than one could match your conditional, order could be important. And finally, note that we passed in "Manager" and "Employee" explicitly as strings. That's because, as we mentioned before, values passed into directives are usually parsed as properties on a scope, so if you want to pass in an actual raw value, you'll need to quote it.

## Event Directives

As our final focus for this lesson, we'll take a look at "event" directives. These are directives that bind some method or action to a particular event, usually one triggered by your user. These could be any of the standard DOM events, such as click, focus, mouseover, etc, and all follow the same naming pattern of `ng-{$EVENT}`, such as `ng-click`, `ng-focus`, etc. You can read about all of the available event directives and any unique aspects to each on the [Angular API documentation][angular-docs], but for our purposes we'll just use `ng-click`, as it's by far the most common.

To utilize an event directive, all you need is a method which you want to call and an element to bind it to. If, for example, you want your user to be able to save their current state, you could simply create a `save` method:

```javascript
angular.module('myApp')
    .controller('myCtrl', function ($scope) {
        $scope.save = function () {
            // handle saving 
        }
    });
```

Then attach it to a button:

```html
<div>
    <!-- Form for user info -->
    <button ng-click="save()">Save</button>
</div>
```

Now your method will be triggered whenever the user clicks that button. There are a couple things to note. First, of course, your method must be attached to the scope for it to be available to an event directive. Secondly, note that we don't just pass in the method name, but actually "call" it by including the parentheses as well. This means that we can also pass in arguments, either other scoped properties (`ng-click="save(user)"`), static values (`ng-click="save('homepage')`), or even the DOM event object itself, which you might be familiar with from jQuery or raw js event listeners (`ng-click="save($event)`). In this final option, you do have to use the exact name `$event`, as that's a locally injectable variable that all event directives look for and bind to the DOM event if it exists.

## Resources

### [Angular API Docs][angular-docs]

---
[angular-docs]: https://docs.angularjs.org/api "Angular API Docs"