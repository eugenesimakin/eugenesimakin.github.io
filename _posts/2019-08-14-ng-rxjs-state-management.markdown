---
layout: post
title:  "Simple RxJS-based state management for an Angular application"
date:   2019-08-14 00:00:00 +0000
categories: [angular, state-management]
---

In this article I'll share my experience about an approach to manage state of an app described [here](https://dev.to/avatsaev/simple-state-management-in-angular-with-only-services-and-rxjs-41p8) (_Simple State Management in Angular with only Service and RxJS_).

In case the linked article is not available anymore or you didn't read it, here is the idea of the approach in a nutshell...

You'll need a store service like this:
{% gist 51283d3a43f5a510c02c294beb14dc2b %}

This is an usage example:
{% gist 18b61974f615e9fa2e08c0ec5851b9da %}

Full source code is available at [github.com](https://github.com/esimakin/angular7-todo-app).

As you can see it's, indeed, pretty simple and there's no need for additional libraries. But because it's simple this way has a few drawbacks (or lack of useful features) such as:
* There's no history of the state. The history might be helpful in some cases (not only the undo/redo but the others as well).
* Assuming you don't want to put the http client into the store service (to make testing easier) you'll have to find another place (I put it into the module's constructor).

On the bright side, this solution is simple and efficient. It might be perfect for small apps. For better performance, use the _OnPush_ change detection strategy together with _async_ pipes and _distinctUntilChanged_ rxjs operator.

In conclusion, I'd prefer to use well known and considered solution like NGRX or Redux but for very small apps it'd okay to use this simple one. 

