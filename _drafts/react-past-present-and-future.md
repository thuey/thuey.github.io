---
title: 'React: Past, Present, and Future'
date: 2018-10-10 00:00:00 +0000
tags: []
description: ''

---
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