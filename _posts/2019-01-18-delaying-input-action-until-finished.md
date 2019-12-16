---
title: "Waiting "
layout: post
excerpt: "I recently encountered a problem in Angular, where an api was called on keyup in a field. Typescript has a handy little debounce option for this. But upon logging the api call, it quickly became clear that, though debounce does delay the action, it doesn't seem to cancel it. That defeated the purpose of the debounce. I would type 3 letters, it would wait until I had finished, and then make 3 api calls. So I quickly cooked up a simple solution."
last_modified_at: 2018-09-04T19:27:01
categories:
  - code snippet
tags:
  - code
  - javascript
---

I recently encountered a problem in Angular, where an api was called on keyup in a field. Typescript has a handy little debounce option for this. But upon logging the api call, it quickly became clear that, though debounce does delay the action, it doesn't seem to cancel it. That defeated the purpose of the debounce. I would type 3 letters, it would wait until I had finished, and then make 3 api calls. So I quickly cooked up a simple solution.

I created a service with only one function. Here's what it came down to:

```typescript
class InputDelayService {
  private timer;

  awaitFinished(callback) {
    clearTimeout(this.timer);

    this.timer = setTimeout(function () {
      callback();
    }, 500);
  }
}

class SomeOtherClass {
  constructor(private inputDelayService: InputDelayService,
              private apiService: ApiService) {}

  onKeyUp(value) {
    this.inputDelayService.awaitFinished(() => {
      this.apiService.get(value);
    });
  }
}
```

The InputDelayService would keep clearing the timeout whenever onKeyUp was called, unless the delay was > 500 ms, at which point it executes the callback*.

I found a few examples of code that did a similar thing, but it was way more specific to string inputs. I realized after implementing those examples, I could delete half the code, and it would still work perfectly. It's even useable to other types of input.

*In my actual code, I used an emitter instead of a callback. That was just a choice I made, either way would work.
