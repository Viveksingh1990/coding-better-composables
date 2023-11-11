Flexible Arguments
==================

_This course is based on our [Coding Better Composables](https://www.vuemastery.com/blog/coding-better-composables-1-of-5) blog series authored by Michael Thiessen._

* * *

When using composables in Vue.js, sometimes you already have a ref that you want to use. Other times, you don’t. This lesson will go through a pattern that lets you use your composables either way, giving you more flexibility when writing your applications.

* * *

What are flexible arguments?
----------------------------

Almost all composables require some type of argument as an input. Often, this is a reactive `ref`. It can also be a primitive Javascript type, like a string, number, or object. But we want to write our composables to be even more flexible and reusable, right?

Instead of requiring either a ref _or_ a primitive, we can accept either. The `useTitle` composable that we saw in the previous article applies the flexible argument pattern.

When you pass in a ref, it’s linked to the document title. Then the title will be set to the value of that `ref`:

    const title = ref('This is the title');
    useTitle(title);
    title.value = 'New title please';
    
    

If you pass in just a string, it will create a new `ref` for you and then proceed to link it up to the document title:

    const title = useTitle('This is the title');
    title.value = 'New title please';
    

In these contrived examples, it doesn’t look like much of a difference. However, when you’re using other methods and composables, you might already have a `ref` from somewhere else. Or you might not. Either way, this composable can adapt to what you need.

Now let’s see how to make this work in our composables.

* * *

**Implementing flexible arguments in a composable**
---------------------------------------------------

To make the flexible arguments pattern work, we need to use either the `ref` function or the `unref` function on the argument we get:

    // When we need to use a ref in the composable
    export default useMyComposable(input) {
      const ref = ref(input);
    }
    
    // When we need to use a raw value in the composable
    export default useMyComposable(input) {
      const rawValue = unref(input);
    }
    

The `ref` function will create a new `ref` for us. But if we pass it a `ref`, it just returns that `ref` to us:

    // Create a new refconst myRef = ref(0);
    
    // Get the same ref back
    assert(myRef === ref(myRef));
    
    

The `unref` function works the same, but instead it either unwraps a `ref` or gives us our primitive value back:

    // Unwrap to get the inner valueconst value = unref(myRef);
    
    // Returns the same primitive value
    assert(value === unref(value));
    
    

Let’s see how some composables from VueUse implement this pattern. [VueUse](https://vueuse.org/) is an open-source collection of composables for Vue 3 and is very well written. It’s a great resource to learn how to write great composables!

* * *

**useTitle**
------------

We’ll come back to the `useTitle` composable since we’re already familiar with it.

This composable lets us pass in either a string or a `ref` of a string. It doesn’t care which we provide:

    // Pass in a string
    const titleRef = useTitle('Initial title');
    
    // Pass in a ref of a string
    const titleRef = ref('Initial title');
    useTitle(titleRef);
    
    

In the [source code](https://github.com/vueuse/vueuse/blob/e484c4f8e4320ff58da95c2d18945beb83772b72/packages/core/useTitle/index.ts#L39), you can see that right after we destructure our options object, we create the `title` ref. We use the `ref` function here, which allows us to use either a `ref` or a string to make the `title` ref:

    // ...
    const title = ref(newTitle ?? document?.title ?? null)
    // ...
    

The `??` syntax is the [nullish coalescing operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing_operator) — a fancy-sounding name for “if the value on the left is null or undefined, use the value on the right.” So this line first tries to use `newTitle`, but if that isn’t defined, it will use `document.title`, and if _that_ isn’t defined, it will give up and use `null`.

Something interesting to note for you TypeScript connoisseurs:

The `newTitle` variable used here has the type `MaybeRef<string>`. Here is what [the type is defined as](https://github.com/vueuse/vueuse/blob/f65707876e1d93211c44414c2a30dc90b1178d68/packages/shared/utils/types.ts#L15):

    type MaybeRef<T> = T | Ref<T>
    
    

This type definition means that the type `MaybeRef<string>` can either be a `string` or a `Ref<string>`, which is a ref with a string value inside.

The next composable we’ll look at also uses this type to implement this pattern.

* * *

**useCssVar**
-------------

The [useCssVar](https://vueuse.org/core/usecssvar) composable allows us to grab the value of a CSS variable and use it in our app:

    const backgroundColor = useCssVar('--background-color');
    

Unlike `useTitle` though, here we need the string value so that we can look up the CSS variable in the DOM. Using the `unref` function, this composable can handle both refs and strings being passed in:

    // Using a string
    const backgroundColor = useCssVar('--background-color');
    
    // Using a ref
    const cssVarRef = ref('--background-color');
    const backgroundColor = useCssVar(cssVarRef);
    
    

Looking at the [source code](https://github.com/vueuse/vueuse/blob/e484c4f8e4320ff58da95c2d18945beb83772b72/packages/core/useCssVar/index.ts#L22), we can see that it uses the `unref` function to accomplish this. Actually, it uses a helper function, called `unrefElement`, to ensure we’re getting a DOM element and not just a Vue instance.

Most composables in VueUse implement this pattern, if you want to explore it further. So pick one that looks interesting and dive into the code!

* * *

**Wrapping things up**
----------------------

We just spent some time learning the second pattern in the series, where we can use arguments more flexibly by using `ref` and `unref` intelligently in our composables. The composable will still work whether you happen to have a `ref` or just the raw Javascript value. It adapts to how you use it!

We also looked at how the VueUse library implements this pattern in the `useTitle` and `useCssVar` composables. The `useTitle` composable uses the `ref` function, and the `useCssVar` uses the `unref` function so that we could see both variations in action.

In the next lesson, we’ll look at a pattern to improve return values by making them dynamic. We’ll learn how we can return either a single value or an object, depending on what is needed:

    // Returns a single value
    const isDark = useDark();
    
    // Returns an object of values
    const {
      counter,
      pause,
      resume,
    } = useInterval(1000, { controls: true });
    
    

This pattern can make your composable a lot simpler to use, especially if you only need a single value most of the time.
