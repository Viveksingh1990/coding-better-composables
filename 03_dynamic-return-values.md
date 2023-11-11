Dynamic Return Values
=====================

_This course is based on our [Coding Better Composables](https://www.vuemastery.com/blog/coding-better-composables-1-of-5) blog series authored by Michael Thiessen._

What if your composable could change what is returned based on its use? If you only needed a single value, it could do that. And if you needed a whole object returned, it could do that, too.

This article will look at a pattern for adding dynamic returns to composables. We’ll understand when to use this pattern, how to implement it, and look at some examples of the pattern in use.

**The Pattern of Dynamic Return Values**
----------------------------------------

If we take a look at the documentation for the useInterval composable, we can see that it can have two different return values:

    // Returns a single value
    const counter = useInterval(200)
    
    // Returns an object of values
    const {
      counter,
      pause,
      resume,
    } = useInterval(1000, { controls: true });
    

But how is this possible? JavaScript by default doesn’t allow for this type of behavior, we’ll have to code it up.

**Implementing in a composable**
--------------------------------

To implement this pattern, we need to do two things:

1.  Add a `controls` option to let the composable know we want alternate return values.
2.  Use `controls` to change the return statement.

Here’s a simple sketch of what this looks like:

    export default useComposable(input, options) {
    // 1. Add in the `controls` option
    const { controls = false } = options;
    
    ...
    
    // 2. Either return a single value or an object
    if (controls) {
        return { singleValue, anotherValue, andAnother };
      } else {
        return singleValue;
      }
    }
    
    

How you decide to do this switching is ultimately up to you. Do what makes the most sense for you and your composable.

Perhaps you’ll want to switch on an existing option instead of using a `controls` option only for this purpose. Maybe it will be cleaner to use a ternary in the return instead of an `if` statement. Or maybe there’s an entirely different way that works best for you. The important thing with this pattern is the switching, not how the switching works.

Next, let’s see how some composables from VueUse implement this pattern.

* * *

### **useInterval**

First, let’s take a deeper look at how [useInterval](https://vueuse.org/shared/useinterval/#useinterval) works.

This composable will update a counter on every interval:

    // Updates `counter` every 500 milliseconds
    const counter = useInterval(500);
    

At [the very top](https://github.com/vueuse/vueuse/blob/e484c4f8e4320ff58da95c2d18945beb83772b72/packages/shared/useInterval/index.ts#L26) of the composable we destructure our options object, pulling out the `controls` option and renaming it to `exposeControls`. By default we won’t show the controls:

    const {
      controls: exposeControls = false,
      immediate = true,
    } = options;
    
    

Then, [at the end](https://github.com/vueuse/vueuse/blob/e484c4f8e4320ff58da95c2d18945beb83772b72/packages/shared/useInterval/index.ts#L33-L41), we have an `if` statement that switches on `exposeControls`. Either we return an object that includes the `counter` ref and all of the controls, or just the `counter` ref:

    if (exposeControls) {
      return {
        counter,
        ...controls,
      };
    else {
      return counter;
    }
    
    

The extra controls come from a helper composable used by the `useInterval` composable. We’ll see this being used again in the next composable.

With these two code snippets, we can make any composable have a more flexible return statement.

Now let’s take a look at the `useNow` composable.

* * *

### **useNow**

The [useNow](https://vueuse.org/core/usenow) composable lets us grab a `Date` object that represents now and updates reactively:

    const now = useNow();
    

By default, it will refresh itself every frame — typically 60 times per second. We can change how often it updates, but we can also pause and resume the update process:

    const { now, pause, resume } = useNow({ controls: true });
    
    

This composable works in a very similar way to the `useInterval` composable. Internally they both use the `useIntervalFn` helper that [VueUse exposes](https://vueuse.org/shared/useIntervalFn/).

First, we [destructure the options object](https://github.com/vueuse/vueuse/blob/f65707876e1d93211c44414c2a30dc90b1178d68/packages/core/useNow/index.ts#L32-L35) to get the `controls` option, again renaming it to `exposeControls` to avoid a naming collision:

    const {
      controls: exposeControls = false,
      interval = 'requestAnimationFrame',
    } = options;
    
    

Then we `return` at the end of the composable. Here we use an `if` statement to switch between the two cases:

    if (exposeControls) {
      return {
        now,
        ...controls,
      };
    else {
      return now;
    }
    
    

As you can see, this pattern is implemented nearly identically in both the `useInterval` and `useNow` composables. All of the composables in VueUse that implement this pattern do it in this particular way.

Here’s a list of other composables — that I could find — that implement this pattern in VueUse, for you to explore more on your own:

*   useTimeout
*   useTimestamp
*   useTimeAgo

* * *

**Wrapping Things Up**
----------------------

We saw that dynamic return values let us choose how we want to use the composable more flexibly. We can get a single value back if that’s all we need. And we can also get an entire object with values, methods, and whatever else we might want.

But we didn’t just look at the pattern itself. We saw how the `useInterval` and `useNow` composables from VueUse implement this pattern. Either they return a single value, a counter or the current timer, or they can also provide controls for pausing and resuming updates.

This pattern is great for simplifying our code in most cases while still allowing for greater complexity when it’s needed. It’s sort of like a desk with a drawer. You can have lots of stuff laid out on the desk when you need it. But you can also put them away in the drawer to keep things tidy.

In the next lesson in this series, we’ll learn how to write composables from scratch. We’ll look at starting with the interface in mind, imagining how it will be used, which makes it easier to write a composable that will work well for us far into the future.
