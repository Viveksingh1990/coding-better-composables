Async Without Await
===================

_This course is based on our [Coding Better Composables](https://www.vuemastery.com/blog/coding-better-composables-1-of-5) blog series authored by Michael Thiessen._

If you can get async code to work correctly, it can significantly simplify your code. But wrangling that added complexity, especially with composables, can be confusing.

This lesson walks through the **Async without Await** pattern. It’s a way to write async code in composables without the usual headaches.

* * *

**Async Without Await**
-----------------------

Writing async behavior with the composition API can be tricky at times. All asynchronous code must be at the end of your `setup` function after any reactive code. If you don’t do this, it can interfere with your reactivity.

The `setup` function will return when it runs into an `await` statement. Once it returns, the component is mounted, and the application continues executing as usual. Any reactivity defined _after_ the `await`, whether it’s a computed prop, a watcher, or something else, won’t have been initialized yet.

This means that a computed property defined after an `await` won’t be available to the template at first. Instead, it will only exist once that async code is finished and the `setup` function completes execution.

However, there is a way to write async components that can be used _anywhere_, without all of this trouble:

    const count = ref(0);
    // This async data fetch won't interfere with our reactivity
    const { state } = useAsyncState(fetchData());
    const doubleCount = computed(() => count * 2);
    
    

This pattern makes working with async code so much safer and more straightforward. Anything that reduces the amount of stuff you have to keep track of in your head is always helpful!

* * *

**Implementing the Async Without Await Pattern**
------------------------------------------------

To implement the pattern, we’ll hook up all of the reactive values synchronously. Then, those values will be updated asynchronously whenever the async code finishes.

First, we’ll need to get our state ready and return it. We’ll initialize with a value of `null` because we don’t know what the value is yet:

    export default useMyAsyncComposable(promise) {
      const state = ref(null);
      return state;
    }
    

Second, we create a method that will wait for our `promise` and then set the result to our `state` ref:

    const execute = async () => {
      state.value = await promise;
    }
    
    

Whenever this promise returns, it will update our state reactively.

Now we just need to add this method into our composable:

    export default useMyAsyncComposable(promise) {
      const state = ref(null);
    
    // Add in the execute method...const execute = async () => {
        state.value = await promise;
      }
    
    // ...and execute it!
      execute();
    
      return state;
    }
    
    

We invoke the `execute` function right before returning from the `useMyAsyncComposable` method. However, we don’t use the `await` keyword.

When we stop and wait for the promise inside the `execute` method, the execution flow returns immediately to the `useMyAsyncComposable` function. It then continues past the `execute()` statement and returns from the composable.

Here’s a more detailed illustration of the flow:

    export default useMyAsyncComposable(promise) {
      const state = ref(null);
    
      const execute = async () => {
    // 2. Waiting for the promise to finish
        state.value = await promise
    
    // 5. Sometime later...// Promise has finished, `state` is updated reactively,// and we finish this method
      }
    
    // 1. Run the `execute` method
      execute();
    // 3. The `await` returns control to this point// 4. Return state and continue with the `setup` functionreturn state;
    }
    
    

The promise is executed “in the background,” and because we aren’t waiting for it, it doesn’t interrupt the flow in the `setup` function. We can place this composable anywhere without interfering with reactivity.

Let’s see how some VueUse composables implement this pattern.

* * *

**useAsyncState**
-----------------

The [useAsyncState](https://vueuse.org/core/useasyncstate/) composable is a much more polished version of what we’ve already experimented with in this article. It lets us execute any `async` method wherever we want, and get the results updated reactively:

    const { state, isLoading } = useAsyncState(fetchData());
    

When [looking at the source code](https://github.com/vueuse/vueuse/blob/main/packages/core/useAsyncState/index.ts), you can see it implements this exact pattern, but with more features and better handling of edge cases.

Here’s a simplified version that shows the outline of what’s going on:

    export function useAsyncState(promise, initialState) {
      const state = ref(initialState);
      const isReady = ref(false);
      const isLoading = ref(false);
      const error = ref(undefined);
    
      async function execute() {
        error.value = undefined;
        isReady.value = false;
        isLoading.value = true;
    
        try {
          const data = await promise;
          state.value = data;
          isReady.value = true;
        }
        catch (e) {
          error.value = e;
        }
    
        isLoading.value = false;
      }
    
      execute();
    
      return {
        state,
        isReady,
        isLoading,
        error,
      };
    }
    
    

This composable also returns `isReady`, which tells us when data has been fetched. We also get the `isLoading` ref and an `error` ref to track our loading and error states from the composable.

Now let’s look at another composable, which I think has a fascinating implementation!

* * *

**useAsyncQueue**
-----------------

This composable is a fun one (there are lots of fun composables in VueUse!).

If you give `useAsyncQueue` an array of functions that return promises, it will execute each in order. But it does this sequentially, waiting for the previous task to finish before starting the next one. To make it _even_ _more_ useful, it passes the result from one task as the input to the next task:

    // This `result` will update as the tasks are executed
    const { result } = useAsyncQueue([getFirstPromise, getSecondPromise]);
    

Here’s an example based on the documentation:

    const getFirstPromise = () => {
    // Create our first promise
    return new Promise((resolve) => {
        setTimeout(() => {
          resolve(1000);
        }, 10);
      });
    };
    
    const getSecondPromise = (result) => {
      return new Promise((resolve) => {
        setTimeout(() => {
          resolve(1000 + result);
        }, 20);
      });
    };
    
    const { activeIndex, result } = useAsyncQueue([
      getFirstPromise,
      getSecondPromise
    ]);
    
    

Even though it’s executing code asynchronously, we don’t need to use `await`. Even internally, the composable doesn’t use `await`. Instead, we’re executing these promises “in the background” and letting the result update reactively.

If you want to learn more about how this function works internally, be sure to check out the [full blog post](https://www.vuemastery.com/blog/coding-better-composables-5-of-5/) by Michael Thiessen.

* * *

**Wrapping It Up**
------------------

We can use `async` composables much more easily if we use the Async Without Await pattern. This pattern lets us place our async code wherever we want without worrying about breaking reactivity.

The key principle to remember is this: if we first hook up our reactive state, we can update it whenever we want, and the values will flow through the app because of reactivity. So there’s no need to `await` around!
