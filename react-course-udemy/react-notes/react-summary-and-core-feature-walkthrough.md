## Vite 
Build tool to make JSX etc conversion to optimized and transformer javascript/html. Also provide easy tools.  
## Node
Used for build tools and not react  

## React
```
npm create vite@latest app-name
npm run dev
```

main.tsx is the entry file and runs first. package.json defines external dependencies/libraries used. Inisde package.json the main libraries for react is react and react-dom. main.jsx renders App inside the root tagged element in html. App is simply a react componet that is simple a function that returns a JSX and is exported to be used whereever we use. Name of the function inside app must have uppercase first letter. Any component that starts lowercase, reacts looks for it in default html tags and not componets.
```
<App/>
```

React is simply creating these components, nesting them and wiring them to create complex User Interfaces.  

Inside JSX we can use {} and inside it we can put any JavaScript expression. Its executed and outputted there. Not only caluclations but we can reference things/variables etc outside JSX.  

Once we create a component we can use it in 2 ways
```
<Post/>
<Post></Post>
```

main.tsx (Entry point. Linked with html)
```javascript
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'
import App from './App.tsx'

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
  </StrictMode>,
)
```

App.tsx
```javascript
import './App.css'
import Post from './components/Post'

function App() {

  return (
    <>
     <Post author="Abdul Basit" body="This is a sample post."/>
     <Post author="Abdul Basit Bhat" body="This is another sample post."/>
     {/* <Post></Post> */}
    </>
  )
}

export default App
```

Post.tsx
```javascript
// const names = ["Abdul Basit", "Abdul Basit Bhat"];

function Post(props : { author? : string, body? : string }) {
    // const chosenName = Math.random() > 0.5 ? names[0] : names[1];
    return <div>
        {/* <p>{chosenName}</p>
        <p>React Developer</p> */}
        <p>{props.author}</p>
        <p>{props.body}</p>
    </div>
}

export default Post;
```
