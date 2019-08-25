---
layout: post
title:  "RxJS-based State Management in Angular"
date:   2019-08-14 00:00:00 +0000
categories: [angular, state-management]
---

The idea that an app need state management came with the growing popularity of [React + Redux](https://redux.js.org/faq/general#when-should-i-use-redux) solution. Angular has its own _Redux_ which is called [NGRX](https://ngrx.io/docs). In this article I'm going to take a look at the approach described [here](https://dev.to/avatsaev/simple-state-management-in-angular-with-only-services-and-rxjs-41p8) and share my experience using in a small angular application. In addition to that, I'll name benefits and drawbacks of the approach.

Let's review the approach first. The idea behind it is very simple. The [BehaviorSubject](https://github.com/ReactiveX/rxjs/blob/master/doc/subject.md#behaviorsubject) is used as store for the app state. The [Observales](https://github.com/ReactiveX/rxjs/blob/master/doc/observable.md#observable), derived from the _BehaviorSubject_ are used to bind the state to different parts of an app (using the _subscribe_ method or _async_ pipes).

Going into details... You'll need a store service like this:
{% gist 51283d3a43f5a510c02c294beb14dc2b %}

Here what the author of the [linked article](https://dev.to/avatsaev/simple-state-management-in-angular-with-only-services-and-rxjs-41p8) suggests:
- We set the initial state in BehaviorSubject's constructor
- Nobody outside the Store should have access to the BehaviorSubject because it has the write rights
- Writing to state should be handled by specialized Store methods (ex: addTodo, removeTodo, etc)
- Create one BehaviorSubject per store entity, for example if you have TodoGroups create a new BehaviorSubject for it, as well as the observable$, and getters/setters

I agree with the first three points but for the last one I chose the different way to go. Instead of creating one BehaviorSubject per store entity, I'm using `BehaviorSubject<AppState>`, where _AppState_ is a class with immutable fields.

This is an usage example:
{% gist 18b61974f615e9fa2e08c0ec5851b9da %}

Full source code is available at [github.com/me/ng-todo-app](https://github.com/esimakin/angular7-todo-app).

As you can see it's, indeed, pretty simple and there's no need for additional libraries. But because it's simple this way has a few drawbacks (or lack of useful features) such as:
* There's no history of the state. The history might be helpful in some cases (not only the undo/redo but the others as well).
* Assuming you don't want to put the http client into the store service (to make testing easier) you'll have to find another place (I put it into the module's constructor).

On the bright side, this solution is simple and efficient. It might be perfect for small apps. For better performance, use the _OnPush_ change detection strategy together with _async_ pipes and _distinctUntilChanged_ rxjs operator.

In conclusion, I'd prefer to use well known and considered solution like NGRX or Redux but for very small apps it'd okay to use this simple one. 

