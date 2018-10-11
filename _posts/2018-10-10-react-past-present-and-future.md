---
title: 'React: Past, Present, and Future'
date: 2018-10-10 00:00:00 +0000
tags: []
description: What does the future of React look like? tldr; It's very bright, my friends.

---
What does the future of React look like? tldr; It's very bright, my friends.

## How React Works

In order to understand the state of React, it's important to understand how React works, at least at a high level. The secret sauce in React is its ability to translate what is returned from your Component's render function into UI updates. It pulls off this tom foolery by using its virtual representation of the UI and reconciliation algorithm to determine what needs to happen in order to make the UI look like the render function says it should.

### The Virtual DOM

Here is some basic React with JSX.

    const Button = (props) => ({
        <button style={{color: props.color}} />
    });
    
    const Form = () => ({
        <form>
          <Button color="blue" />
        </form>
    });

Babel will translate this JSX to `React.createElement`'s:

    const Button = (props) => (
        React.createElement({
          'button',
          {
            style: {
                color: props.color
            }
          },
          null
        });
    );
    
    const Form = () => ({
        <form>
          <Button color="blue" />
        </form>
        React.createElement({
          'form',
          {},
          React.createElement({
            Button,
            { color: 'blue'},
            null
          })
        });
    });

React assembles these elements into a tree structure, also known as the virtual DOM:
![](https://llimllib.github.io/pymag-trees/images/figure4.png)

### Reconciliation

When something changes (e.g. state is updated), React updates the virtual DOM in a process known as reconciliation. In reconciliation, React calls render on all of the component instances that were affected by the state change. It will then update the virtual DOM with the new render output, computing the diff that was necessary to perform the update along the way. This diff is then passed along to the renderer (most likely react-dom or ReactNative), which updates the UI.  
![](https://sg.fiverrcdn.com/photo2s/113265529/original/eb477fc04ea08437021fe754ece30bdbdb6bfc3b.png?1529521868)

## Stack Reconciliation

The original (React <16) implementation of the reconciliation algorithm is referred to as Stack reconciliation. It's called stack reconciliation because computing the diff for the entire virtual DOM happens during a single recursive call.

![](https://marmelab.com/images/blog/react-slider-poll/profiling.png)

## The problem with Stack Reconciliation

![](https://developers.google.com/web/updates/images/inside-browser/part3/pagejank2.png)

Most screens refresh at 60 fps. If reconciliation and updating the browser window don't complete in between frames, it results in page jank. The nice people over at React put together [this demo](https://claudiopro.github.io/react-fiber-vs-stack-demo/stack.html) to see the jank in action:
![](/uploads/jank.gif)

## Fiber to the Rescue!

In React 16, the React team introduced a new reconciliation algorithm called Fiber. With Fiber, reconciliation can be broken up into smaller units of work known as "fibers." These fibers are scheduled in a way to allow other updates, like animations, to occur in the middle of the reconciliation process, thus mitigating page jank. Check out the difference in the call stacks between Fiber reconciliation (top) and Stack reconciliation (bottom).

![](https://s3.eu-west-2.amazonaws.com/websitegiamir/triangles-demo-with-time-slicing.png)

Once again, the React team was nice enough to put together the same demo, but [implemented with Fiber](https://claudiopro.github.io/react-fiber-vs-stack-demo/fiber.html):

![](/uploads/fiber.gif)

## What's Next?

Reducing page jank is just one of the applications of React Fiber. The rewrite opened up a whole new world of possibilities, such as...

### Fragments for Multiple Children

The way Fiber was implemented makes it so components can now return multiple children from their `render` functions, without a wrapper element like `<span>` or `<div>`:

    render() {
      return (
        <>
          Some text.
          <h2>A heading</h2>
          More text.
          <h2>Another heading</h2>
          Even more text.
        </>
      );
    }

### Time Slicing

Time Slicing is the term for breaking up the reconciliation process into prioritized bits of work to reduce jank. This feature is still being developed and has a volatile API, but when it's released, it will look something like this:

    class ExampleApplication extends Component {
      componentDidMount() {
          this.intervalID = setInterval(this.tick, 1000);
      }
    
      tick() {
          ReactDOMFiber.unstable_deferredUpdates(() =>
              this.setState(state => ({ seconds: (state.seconds % 10) + 1 }))
          );
      }
    
      render() {
          return (
              <div>
                  {this.state.seconds}
              </div>
          );
      }
    }

### Error Boundaries

With React Fiber, you can create React components to handle JavaScript errors. This consolidates error handling in a few select components, instead of scattering error handling throughout the application. Components that handle errors are called Error Boundaries. The ability to create Error Boundary components is already available, and looks like this:

    class ErrorBoundary extends Component {
      componentDidCatch(error, info) {
        this.setState({ hasError: true });
      }
    
      render() {
        if (this.state.hasError) {
          return <h1>Something went wrong.</h1>;
        }
        return this.props.children;
      }
    }

The `ErrorBoundary` component can then be used just like any other component:

    <ErrorBoundary>
      <MyWidget />
    </ErrorBoundary>

### Suspense

Because reconciling can happen asynchronously, you can tell React to pause rendering a part of the component tree that is waiting for data to come in (e.g. from a network call), while continuing to process the rest of the tree and other high-priority updates. Suspense itself merits its own article, and it is still a highly volatile API. However, suspense will work like this:

1. A component checks a cache for the data it needs.
2. If the data does not exist, it throws a promise that will resolve when the data is fetched.
3. An Error Boundary component higher up in the chain will catch the promise and display a loading indicator until the promise resolves.
4. Once the promise resolves, React will continue to reconcile the paused section of the component tree.

## Conclusion

Rewriting the reconciliation algorithm was a huge undertaking by the React team; however, there are now so many cool things you can do with React now to improve the user experience. It will be exciting to see what the community comes up with once features like time slicing and suspense are officially released!