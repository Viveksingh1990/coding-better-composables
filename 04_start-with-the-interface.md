Start with the Interface
========================

_This course is based on our [Coding Better Composables](https://www.vuemastery.com/blog/coding-better-composables-1-of-5) blog series authored by Michael Thiessen._

The composable you wrote last month returns an object, but right now, you _really_ need it to return an array instead. But it’s too late to change now. That would require literally _days_ of refactoring since the composable is already all over the codebase. If only you had taken a bit more time, in the beginning, to really figure out how you’d want to use the composable, you might have been able to save yourself from this frustration.

In this lesson, we will learn about writing interfaces for our composables and how this process can help us craft composables that serve us far into the future.

* * *

**Write the Interface First**
-----------------------------

Instead of starting with the implementation, you should start with how you want the composable to be _used_. That’s what I mean when I say, “begin with the interface.” It’s the API of the composable. It’s the boundary between the composable and other pieces of your app.

It’s tempting to create a new file and start coding away on the implementation, but it’s rarely the best approach. Instead, it’s easier to prototype when you start with the interface, and you’re also likely to create a _better_ interface.

This is important because the interface is way harder to change 6 months from now than the implementation is. So it’s better to spend extra time on this step getting it right.

This is especially true when dealing with highly reusable code. Most reusable pieces of code, whether a component or composable, undergo a lot of volatility when they’re first created. The interface keeps changing because we keep discovering edge cases that require us to refactor. Instead, if we can anticipate most of those edge cases initially, we can save ourselves a lot of pain and frustration.

(But unfortunately, we can never predict the future 😢)

To start with our interface, we write our code pretending that the composable already exists. We also have to answer a few questions through this process:

1.  What arguments do we pass into our composable? A ref, a raw value, or a series of values?
2.  What options should be included in the options object?
3.  What values does our composable return? Is it just a single value, or do we want to use the dynamic return value pattern?

These don’t have to be answered in a specific order, but most should be figured out. Also, I find that I do this iteratively. As I start to answer one question, I better understand how to answer the others. These questions serve as more of a guide than a rigid process you need to follow.

You’ll notice each of these questions is only concerned with how we interact with and use the composable. We’re not yet worried about how the composable actually _works_. Of course, we’ll get to that part, but only after we’ve figured out the interface.

So why don’t we work through an example to see this process in action?

* * *

**Figuring out the useMouse interface**
---------------------------------------

We’ll go through a bit of a sketch to see how this can play out, with a `useMouse` composable. This composable will give us access to the mouse coordinates on the screen. Let’s answer each question and see how we can figure out this interface.

### **Question 1: What arguments do we pass into our composable?**

The answer to this question is straightforward — the `useMouse` composable doesn’t take in any arguments. Instead, we want the mouse coordinates to return to us and update reactively.

### **Question 2: What options should be included in the options object?**

Here’s what we might come up with:

*   **type** - \*\*\*\*When it comes to on-screen coordinates, do we want the mouse position based on **page** (X/Y relative to the whole webpage) or **client** (X/Y relative to the browser viewport). This should default to **page**.
*   **touch** - Should we listen for **touchmove** events? Like if the browser is on a touchscreen (mobile phone or tablet). Default to false.
*   **resetOnTouchEnds** - Should we reset to initial value on **touchend**. Again for touch devices, when the touch stops what should be the registered X/Y? Default to false.
*   **initialValue -** What should the initial value be set to? Defaults to `{ x:0, y:0 }`

### **Question 3: What values does the composable return?**

This one is not so clear from the start. Sometimes we just need to start writing code that uses the composable, and along the way, we discover how we want to use it. That process can give us a lot of clues as to what our interface should be.

Maybe we start by grabbing the coordinates as an object:

    const { x, y } = useMouse();
    

As we continue prototyping, we realize we also need to know if the coordinates are coming from **mouse** or **touch**.

    const { x, y, sourceType } = useMouse();
    

This works, but it’s kind of clunky. As we continue writing our app, we realize that what we really need is to grab the mouse coordinates in a separate object:

    const { position, sourceType } = useMouse();
    
    

Perhaps, as we continue building this out, we discover that sometimes it’s handy to have the coordinates as an array. We can include that in our interface:

    const {
      position,
      positionArr,
      sourceType,
    } = useMouse();
    
    

At this point, we have a pretty good understanding of how we’ll want to use the `useMouse` composable. But, of course, requirements change over time, so this isn’t set in stone. But you can see how much our interface has changed by prototyping a bit. If we had just gone with our first guess, we might have written a composable that would need to be refactored soon.

* * *

**Wrapping Things Up**
----------------------

In this article, we learned that by thinking critically about how we want to use our composable in the beginning, we can write a better implementation. And by writing a better implementation, we save ourselves time and frustration in the future. We won’t need to refactor as much because we’ve been thoughtful about how we want to use the composable.

Of course, we don’t want to spend _too_ much time on this process. There is only so much that you can anticipate, and we all know that requirements can change often. Also, there is no _perfect_ interface, but we can avoid many headaches with a bit of effort at the start.
