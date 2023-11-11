What is a Composable?
=====================

_This course is based on our [Coding Better Composables](https://www.vuemastery.com/blog/coding-better-composables-1-of-5) blog series authored by Michael Thiessen._

* * *

Composables are, by far, the best way to organize business logic in your Vue 3 app.

They let you extract small pieces of logic into functions that you can easily reuse repeatedly. This makes your code easier to write and read.

Since this way of writing Vue code is relatively new, you might be wondering what the best practices are when writing composables. This course will serve as your guide on how to craft solid composables that you and your team can rely on.

First, though, we need to make sure we’re on the same page. So let me take a bit to explain what, exactly, a composable is.

* * *

**What is a Composable?**
-------------------------

According to the [Vue documentation](https://vuejs.org/guide/reusability/composables.html#composables), a composable is “a function that leverages Vue Composition API to encapsulate and reuse **stateful logic**”. If you’re not yet familiar with the composition API, or want to learn more about the advantages of using composables, you’ll want to watch through my [Vue 3 Composition API](https://www.vuemastery.com/courses/vue-3-essentials/why-the-composition-api/) course.

Any code that uses reactivity can be turned into a composable.

Here’s a simple example of a `useMouse` composable from the Vue.js docs:

    import { ref, onMounted, onUnmounted } from 'vue'
    
    export function useMouse() {
      const x = ref(0)
      const y = ref(0)
    
      function update(event) {
        x.value = event.pageX
        y.value = event.pageY
      }
    
      onMounted(() => window.addEventListener('mousemove', update))
      onUnmounted(() => window.removeEventListener('mousemove', update))
    
      return { x, y }
    }
    
    

We define our state as `refs`, then update that state whenever the mouse moves. By returning the `x` and `y` refs, we can use them inside of any component (or even another composable).

Here’s how we’d use this composable inside of a component:

    <script setup>
      import { useMouse } from './composables/mouse';
      const { x, y } = useMouse();
    </script>
    
    <template>
      Mouse position is at: {{ x }}, {{ y }}
    </template>
    

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.1654201387077.gif?alt=media&token=caf97b9a-3d00-4a88-b9bd-2c576996f641)

As you can see, using the `useMouse` composable allows us to easily reuse all of this logic. With very little extra code, we’re able to grab the mouse coordinates in our component.

Now that we’re on the same page, let’s start to think about how we should create composable inputs.

* * *

How to create composable inputs?
--------------------------------

There are a few ways we could do it. We could use an argument for each property:

    const title = useTitle('Product Page', true, '%s | My Socks Store')
    

We could also use an options argument:

    const title = useTitle({ title: 'Product Page', 
                             observe: true, 
                             titleTemplate: '%s | Socks Store' })
    

The benefit here is we don’t have to remember the correct ordering, we know what each option does, and easier to add new options.

Or we could use a combination of both:

    const title = useTitle('Product Page', { observe: true, 
                                             titleTemplate: '%s | Socks Store' })
    

Here we are putting the required arguments first and the other arguments are optional.

It’ll also be _much_ easier to add new options later on. This applies both to adding new options to the composable itself, and to adding options when using the composable.

So, using an options object is great. But how do we implement that, and parse them out.

* * *

How to parse the options argument?
----------------------------------

Here’s how you would implement the options object pattern in a composable:

    export function useTitle(newTitle, options) {
        const {
          observe = false,
          titleTemplate = '%s',
        } = options;
        
        // ...
      }
    

Here, we can accept newTitle (our required argument), and then the last argument is the options object. The next step is to destructure the options object. By destructuring, we can access all the values, and clearly provide defaults for each possible option.

* * *

Introducing VueUse
------------------

Now we’ll look at how two different composables from VueUse apply this pattern. [VueUse](https://vueuse.org/) is an open source collection of composables for Vue 3, and is very well written. It’s a great resource to learn how to write great composables! To use VueUse composables in our application, we can install it in an existing Vue 3 project by running:

    npm i @vueuse/core
    

Now let’s try using a composable from VueUse. First, we’ll look at `useTitle`, and then we’ll see how `useRefHistory` works.

* * *

Using **useTitle**
------------------

The [useTitle](https://vueuse.org/core/useTitle/) composable is fairly straightforward. It let’s you update the page’s title:

    <script setup>
    import { useTitle } from '@vueuse/core'
    
    const title = useTitle('Green Socks', { titleTemplate: '%s | My Store' })
    </script>
    
    <template>
        <h1>Title Composable</h1>
        <input v-model="title" type="text">
    </template>
    

Here is what this produces:

![](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.1654201387078.gif?alt=media&token=53a830b0-5c43-42d0-b093-6853ed1968be)

Now let’s look at a slightly more complicated composable that also uses this options object pattern.

* * *

**useRefHistory**
-----------------

The [useRefHistory](https://vueuse.org/core/userefhistory) composable is a bit more interesting. It let’s you track all of the changes made to a `ref`, allowing you to perform undo and redo operations fairly easily:

    import { useRefHistory } from '@vueuse/core'
    
    // Set up the count ref and track it
    const count = ref(0);
    const { undo } = useRefHistory(count);
    
    // Increment the count
    count.value++;
    
    // Log out the count, undo, and log again
    console.log(counter.value);
    // 1
    undo();
    console.log(counter.value);
    // 0
    

This composable can take a bunch of different options, we could call it like so:

    import { useRefHistory } from '@vueuse/core'
    
    const state = ref({});
    const { undo } = useRefHistory(state, {
      deep: true,  // Track changes inside of arrays and objects
      capacity: 15 // Limit how many steps we track
    });
    

If we look at the [source code](https://github.com/vueuse/vueuse/blob/e484c4f8e4320ff58da95c2d18945beb83772b72/packages/core/useRefHistory/index.ts#L151-L155) for this composable, we see it uses the exact same object destructuring pattern that `useTitle` does:

    export function useRefHistory(source, options) {
      const {
        deep = false,
        flush = 'pre',
        eventFilter,
      } = options;
    
    // ...
    }
    
    

However, in this example we only pull out a few values from the options object here at the start.

This is because `useRefHistory` relies on the `useManualRefHistory` composable internally. The rest of the options are passed as _that_ composable’s options object [later on in the composable](https://github.com/vueuse/vueuse/blob/e484c4f8e4320ff58da95c2d18945beb83772b72/packages/core/useRefHistory/index.ts#L190):

    // ...const manualHistory = useManualRefHistory(
      source,
      {
    // Pass along the options object to another composable
        ...options,
        clone: options.clone || deep,
        setSource,
      },
    );
    
    // ...
    

This also shows something I mentioned earlier: composables can use other composables!

* * *

**Bringing it all together**
----------------------------

This lesson was the first in our course, “Writing Better Composables”.

We looked at how adding an options object as a parameter can make for much more configurable components. For example, you don’t need to worry about argument ordering or to remember what each argument does, and adding more options to an object is far easier than updating the arguments passed in.

But we didn’t just look at the pattern itself. We also saw how the VueUse composables `useTitle` and `useRefHistory` implement this pattern. They do it in slightly different ways, but since this is a simple pattern, there’s not much variation that you can do.

The next lesson in this course looks at how we can accept both refs and regular Javascript values as arguments:

    // Works if we give it a ref we already have
    const countRef = ref(2);
    useCount(countRef);
    
    // Also works if we give it just a number
    const countRef = useCount(2);
    

This adds flexibility, allowing us to use our composables in more situations in our application.
