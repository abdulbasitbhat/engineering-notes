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

In JSX instead of class, className is used.  

## In SPA, States are a concept. React allows us to manage them and render updates based on them. 

## Event Listeners in React using on props and function passing
```javascript
import classes from './NewPost.module.css'

function NewPost() {
    //Javascript way (Imperitive way)
    // document.querySelector('textarea')?.addEventListener('changed', function(e){
    //     console.log(e)
    // }

    //In React we do this with on props and pass a function to it. This is called declarative way.
    //onChange + {functionName}

    function changeBodyHandler(event: React.ChangeEvent<HTMLTextAreaElement>) {
        console.log(event.target.value);
    }
    return (
        <form className={classes.form}>
            <label htmlFor="body">Text</label>
            <textarea name="body" required rows={3} onChange={changeBodyHandler} /> {/* Passing function */}

            <label htmlFor="name">Name </label>
            <input type="text" id="name" required />
        </form>
    )
}

export default NewPost;
```

## State

```javascript
import classes from './NewPost.module.css'

function NewPost() {

    let enteredBody = '';

    function changeBodyHandler(event: React.ChangeEvent<HTMLTextAreaElement>) {
        enteredBody = event.target.value;
    }
    return (
        <form className={classes.form}>
            <label htmlFor="body">Text</label>
            <textarea name="body" required rows={3} onChange={changeBodyHandler} /> {/* Passing function */}
            <p>{enteredBody}</p>
            <label htmlFor="name">Name </label>
            <input type="text" id="name" required />
        </form>
    )
}

export default NewPost;
```
The above code does nothing.  
JSX code is taken once as a snapshot. Any normal variable change is not taken care of if it changes. This is why we need states. useState is the hook we use to initialize this. Hooks cant be executed in regular java functions, they can only be executed inside a component function.  
State value can be anything.  

```javascript
import { useState } from 'react';
import classes from './NewPost.module.css'

function NewPost() {
    const [body, setBody] = useState('');
    
    // const stateData = useState('')
    // stateData[0] //current value
    // stateData[1] //state updating function

    //Change of state via state function calls a rerender of the component it resides in. Rerender only updates the dom related to that state change, not everything

    function changeBodyHandler(event: React.ChangeEvent<HTMLTextAreaElement>) {
        setBody(event.target.value);  //Call rerender of componet and set latest update.
    }
    return (
        <form className={classes.form}>
            <label htmlFor="body">Text</label>
            <textarea name="body" required rows={3} onChange={changeBodyHandler} /> {/* Passing function */}
            <p>{body}</p> {/*Attached with state */}
            <label htmlFor="name">Name </label>
            <input type="text" id="name" required />
        </form>
    )
}

export default NewPost;
```
The state variable is a constant because we cant set it directly. And if we do using setState, it will cause rerender and be a completely new state altogeteher. So state is never assigned new value, its initialized again on rerender that is triggered by setState  

## State vs normal vars
