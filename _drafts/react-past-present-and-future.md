---
title: 'React: Past, Present, and Future'
date: 2018-10-10 00:00:00 +0000
tags: []
description: ''

---
## The Virtual ~~DOM~~ (Current)

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

## ![](https://llimllib.github.io/pymag-trees/images/figure4.png)  
Stack Reconciliation (Past)

When something changes (e.g. state is updated), React updates the virtual DOM in a process known as reconciliation. In reconciliation, React calls render on all of the component instances that were affected by the state change. It will then update the virtual DOM with the new render output, computing the diff that was necessary to perform the update. This diff is passed along to react-dom or ReactNative, which updates the UI.  
![](https://sg.fiverrcdn.com/photo2s/113265529/original/eb477fc04ea08437021fe754ece30bdbdb6bfc3b.png?1529521868)

The original (React <16) implementation of the reconciliation algorithm is referred to as **stack** reconciliation. It's called stack reconciliation because computing the diff for the entire virtual DOM happens during a single recursive call.

![](https://marmelab.com/images/blog/react-slider-poll/profiling.png)

## The problem with Stack Reconciliation