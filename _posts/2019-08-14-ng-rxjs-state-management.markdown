---
layout: post
title:  "Simple RxJS-based State Management in Angular"
---


I got a new project to work on and needed a state management library for that project. Because my team lead
didn't have experience with modern state management libraries and the project appears to be small (even in perspective),
I wanted a simple solution. A solution, which would be easy to use and would work nicely with Angular. Google led me to
[this article](https://dev.to/avatsaev/simple-state-management-in-angular-with-only-services-and-rxjs-41p8) and I decided 
to use it in the new project.

**Warning!** This blog post is not a tutorial and is my experience only. If you're looking for a guide, please, refer to
"[Simple state management in Angular with only Services and RxJS](https://dev.to/avatsaev/simple-state-management-in-angular-with-only-services-and-rxjs-41p8)"
article at [dev.to](https://dev.to/).

Let's review the approach first. The idea is pretty simple. You have a state (plain js object) wrapped in 
[BehaviorSubject](http://reactivex.io/rxjs/manual/overview.html#behaviorsubject).
Any ng component can be subscribed to the _BehaviorSubject_ in order to get updates of the app's state. 
To achieve that, we need to convert the _BehaviorSubject_ to _Observable_ and apply rxjs operators to get 
slices of the state:

{% highlight typescript %}
class StoreService {
  private _state = new BehaviorSubject<State>(new State());
  public state$ = this._state.asObservable(); // might be private

  public partOfState$ = this.state$.pipe(...);

  public changeStateMethod() {
      this._state.next(...)
  }
}
{% endhighlight %}

Inside an ng component:

{% highlight html %}
<some-component>
  <app-todo *ngFor="let todo of todosStore.todos$ | async"
    [todo]="todo">
  </app-todo>
</some-component>
{% endhighlight %}

The _AsyncPipe_ is quite important here. In addition to that, the _OnPush_ change detection strategy could be
used to improve performance but in my case, when the application is not big or medium size, this is not important.
And that's pretty much all:
- We set the initial state in BehaviorSubject's constructor
- Nobody outside the Store should have access to the BehaviorSubject because it has the write rights
- Writing to state should be handled by specialized Store methods (ex: addTodo, removeTodo, etc)
- Create one BehaviorSubject per store entity, for example if you have TodoGroups create a new BehaviorSubject for it, as well as the observable$, and getters/setters

The solution is, indeed, pretty simple and doesn't require additional libraries in Angular. 
But it has a few drawbacks (or lack of useful features) such as:
* There's no history of the state. The history might be helpful in some cases (not only the undo/redo but the others as well).
* There's no middleware. For example, I don't see how a logger service could be added (as independent entity).
* The solution is not quite flexible.

On the bright side, this solution is simple and efficient. It might be perfect for small apps. 
For better performance, use the _OnPush_ change detection strategy together with _async_ pipes and _distinctUntilChanged_ rxjs operator.


