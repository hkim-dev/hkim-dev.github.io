---
title: "Click Detection Outside a React Component: Simplified Integration of Multiple Refs"
date: 2024-02-08 15:16:28 -0400
categories: "frontend"
tags:
  - React
  - Typescript
classes: wide
toc: true
---

# Background
> This article extends upon the one I wrote on [React DnD](https://hkim-dev.github.io/frontend/Implement-Drag-and-Drop-for-React-App-with-React-DnD/). Check this one out too if interested! (or for better understanding :smiley:)


| ![layout](/assets/images/dnd_app_layout.jpeg) |
|:--:|
| *Figure 1. Simplified Layout of React App* |

While enabling the click-to-fill behavior on top of Drag and Drop as per PO's request, I realized that it is almost counterintuitive not to remove UI effects appearing on a selected item **when the user has clicked outside of a desired/clickable area.** Just like how [input](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input) is, I thought the user would expect the component to have focus when clicked, and lose it when the user has clicked something else.
I hope this curation of the discoveries I made while working on the feature can be a thorough guide of the subject matter. Now let's start with the concept of React Ref.


# What is a React Ref?
In a nutshell, it is a mutable reference that persists across component renders. As opposed to a state, it does not trigger new renders.

```tsx
const ref = useRef(0);
```

useRef returns an object like this so that you can access the current value of the ref through the `ref.current` property. 

```tsx
{ 
  current: 0 // The value you passed to useRef
}
```

Why do we need refs in the context of focus and blur? There is no built-in way to do manipulate those things in React, so we need a ref to access the DOM elements managed by React. Pass your ref as the ref attribute to the JSX tag for which you want to get the DOM node.

```tsx
<div ref={ref}>
```

Using the property `current`, you can access this DOM node.


# How to Detect Clicks Outside Clickable Area

| ![blue_layout](/assets/images/blue_highlighted_app_layout.jpeg) |
|:--:|
| *Figure 2. Clickable Area* |

Let's begin with defining what a *clickable area* is here. It really varies depending on individual use cases, but in my case, it is the blue colored area except for Picks and Tiles. The goal here is to reset a click on one of the tiles in the main grid when the consecutive click is made within the blue color background.

Another important question - what does a click does to the UI?

```tsx
const handleTileClick = () => {
  setClickedTile(tile);
  setIsClicked(true);
};
```

A click on an item makes the selected item highlighted (or other visual effects you have in place for when isClicked is `true`) and sets a new value to a state variable `clickedTile`.


## The `handleClickOutside` function

Now let's create a ref and put it to use. Refer to the following code for the implementation of the click event listener `handleClickOutside` and how a ref is used here.


```tsx
const clickOutsideRef = useRef<HTMLDivElement>(null);

const handleClickOutside = (event: MouseEvent) => {
  if (clickOutsideRef.current && !clickOutsideRef.current.contains(event.target as HTMLElement)) {
    const isClickOnPick = !!event.target && (event.target as HTMLElement).closest('.pick'); // Check if the click occurred within a Pick component
    if (!isClickOnPick) { // click occurred outside of Pick, completely reset the behaviors related to tile clicks
      setClickedChampion(null);
    };
    setIsClicked(false);
  };
};

useEffect(() => {
  document.addEventListener('click', handleClickOutside, true); // bind the event listener
  return () => {
    document.removeEventListener('click', handleClickOutside, true); // unbind the listener on clean up
  };
}, []);
```

`handleClickOutside` checks if clickOutsideRef.current exists (i.e., if the referenced element exists in the DOM) and if the click event's target is not within the referenced element. If there is a click detected outside the referenced element, it further checks if the click has taken place within an element with the class '.pick', which is another acceptable clickable area in my case.
> [`document`](https://developer.mozilla.org/en-US/docs/Web/API/Window/document) refers to the global `Document` object provided by the browser's DOM.

Make sure to pass `clickOutsideRef` to the component you wish to detect clicks outside. In my case, it's the Tile component whose elements are defined as follows:
```tsx
<div
  ref={mergedRef}
  id={`tile-${index}`}
  key={index}
  className={`transform hover:scale-110 transition-transform duration-300 // increase the size on hover
    ${isDragging ? 'opacity-50 shadow-lg cursor-grabbing ease-in-out' : 'opacity-100 cursor-pointer ease-in-out'}
    ${isDropped ? 'grayscale pointer-events-none' : 'pointer-events-auto'}
    ${isClicked ? 'border-dotted border-2 border-indigo-600' : ''}
    rounded-lg`}
  onMouseDown={onDragStart}
  onClick={handleTileClick}
>
  <img
    className="mx-auto rounded-lg cursor-pointer filter"
    src={`${CONFIG.IMAGE_URL}/${tile.image}`}
    alt={`${tile.image}`}
  />
</div>
```

You have probably notice that the ref as the `ref` attribute is something else, other than `clickOutsideRef`. Where did it come from? Hopefully the name `mergedRef` has tipped you off a little bit. I'll explain about merging multiple refs assuming any of you already have a ref attached to the element that you need to add another ref to, like I did.


## Merging Multiple Refs
It was initially puzzling as to how I could make `Tile` draggable and able to detect outside clicks at the same time when [it already uses a `drag` ref.] (https://hkim-dev.github.io/frontend/Implement-Drag-and-Drop-for-React-App-with-React-DnD/#set-up-draggable-component) A few articles I stumbled upon helped me greatly in integrating those two refs, possibly more than two if needed in the future:
- [How to merge refs in React component](https://mayursinhsarvaiya.medium.com/how-to-merge-refs-in-react-component-d5e4623b6924)
- [Using multiple refs on a single React element](https://stackoverflow.com/questions/60270678/using-multiple-refs-on-a-single-react-element)

This is the Typescript version of the code that was perfect for my project:

```tsx
import { type MutableRefObject, type RefCallback } from 'react';

type MutableRefList<T> = Array<RefCallback<T> | MutableRefObject<T> | undefined | null>;

export function mergeRefs<T>(...refs: MutableRefList<T>): RefCallback<T> {
  return (val: T) => {
    setRef(val, ...refs);
  };
};

export function setRef<T>(val: T, ...refs: MutableRefList<T>): void {
  refs.forEach((ref) => {
    if (typeof ref === 'function') {
      ref(val);
    } else if (ref != null) {
      ref.current = val;
    }
  });
};
```
This code is designed to work with multiple refs and handle their different types such as callback functions and ref objects. Use this function like the following and utilize the merged `ref` according to your project's needs! 

```tsx
const mergedRef = mergeRefs(drag, clickOutsideRef);
```

# Wrapping Up
In this article, we've explored how to detect clicks outside a React component using React refs and event listeners. By understanding these concepts and implementing them in our applications, we can create interfaces that are more interactive and user-friendly.
Personally, working on this feature led me to delve into a variety of underlying concepts, such as the DOM. It was so much fun to tap into the power of React refs for tracking user interactions, especially clicks on specific areas. Hoping it was informative for the readers, I'm signing off. Happy coding! :v:

<br>


#### References
- https://react.dev/learn/referencing-values-with-refs
- https://stackoverflow.com/questions/32553158/detect-click-outside-react-component