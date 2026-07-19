# React  
JavaScript Library for building User Interfaces. Such user interfaces that feel like apps (SPA). JS could do it itself, but React makes it easier. JS is browser language. React is a library that makes it easier. It has an easier mental model.  
In React, one defines the target UI state. We define the update cases and the UI state, React itself knows how to get there (the steps), "Declarative UI Programming". In JS, its "Imperitive coding", one has to define how to get the UI state.  

```
npm create vite@latest react-project  
npm install
npm run dev
```

## Why we need node? 
Tools like Vite, which are used to create React projects use node. We dont need node for react.  

## Why can't we create react app by files creation like we do for JavaScript?  
The code like JSX we write is not readabale to browser. So for transformation and optimization we need a complex project setup, i.e a build tool included project. Vite is a build tool. It makes code executable and executable with performance.  

