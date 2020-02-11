---
tags:
- devdaily
- react
title: React Render Props
description: How the "render prop" pattern can be used to share code across React
  components.
date: 2020-02-09 07:00:00 +0000

---
One way to share code between React components is by implementing the "render prop" pattern. In the render prop pattern, a function that returns a React element is passed as a property into a component. That component then calls the function instead of defining its own React element to render.

Let's look at an example:

    function () {}