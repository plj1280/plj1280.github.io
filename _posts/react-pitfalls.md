## Misunderstandings and Pitfalls with React

### React Basics
React is a popular "declarative" UI framework. Basically, you just need to define how the interface should look at a given moment, without worrying about any details about how it got there or where it will go next. React tries to efficiently update the UI by keeping track of what has changed and only telling the browser to update those elements.

React uses a hierarchical data model to match the hierarchical structure of HTML. "Props" can only flow down the hierarchy and cannot be altered. "State" belongs to a particular node and can be altered by descendant nodes that are given permission. The content of the state can flow down the hierarchy as a prop.

So, the general pattern in React is that a component responds to a user interaction by updating the state of itself or an ancestor node. That new state will propagate to descendants as a prop, which can cause the graphical output of that component to change. React uses these restrictions on data flow and mutation to efficiently determine when and where the UI needs to be updated.

#### A note on "Rendering"
Rendering refers to the process of creating a graphical output, all the way from some initial representation of data to pixels on a screen. For example, a browser takes in data in the form of HTML and CSS and renders a graphical webpage.

Unfortunately, "rendering" is also sometimes used to refer intermediate steps in a pipeline. For example, in React, "rendering" usually refers to the creation of the current desired HTML tree, known as the virtual DOM. This is the "rendering" defined in user-created React components.

There is also the "rendering" of the whole virtual DOM tree that takes place in the ReactDOM.render function. This process, also known as "reconciliation," is where React determines how to efficiently alter the browser DOM to match the virtual DOM.

So, there's really three separate stages in a React app that we can call "rendering"
-Components taking state and props and outputting virtual DOM nodes
-React taking a virtual DOM tree and changing elements on the browser DOM to match
-The browser taking the browser DOM and displaying the webpage on the screen

React developers shouldn't really have to worry about the last one, but mixing up the first two concept caused me a lot of confusion.

#### Misunderstanding: "React only re-renders a component if its state or props change"
Let's start with the first sense of rendering, creating new virtual DOM elements. When React detects a change in state, it *does* trigger a re-rendering of the component that owns that state. That means the entire function that makes up the component will re-execute, and the new output will replace the old node on the virtual DOM. Here's the crucial point that I didn't understand at first: rendering of a component means recursive rendering of all child components. So, *every descendant* of the node with a state change will be re-rendered, whether that state change affects their props or output or not. This leads to a lot of seemingly useless code re-execution, but this stage of rendering is usually quite cheap, and it's why the render function must not cause side effects.

So, rule is simply: "React will re-render a component if it *or any ancestor* has a state change." Note that there is no dependence on props, and state changes do not affect siblings or ancestors. And remember, this is all about the creation of virtual DOM nodes, *not* the updating of the browser DOM.

Once a new virtual DOM is created, it is compared to the browser DOM in the "reconciliation" process. In reconciliation, React basically goes through the DOMs node by node to see if they correspond. If the types are the same, React keeps the element in the browser DOM and changes any attributes that are different in the new virtual DOM. If the types don't match, React throws out the entire browser DOM node and replaces it with the virtual DOM node. Note that React needs to remember the types of the components in the browser DOM, even though the browser doesn't need that to render the DOM. The type corresponds to either an HTML tag *or* a reference to the function that renders the component. Importantly, even if two functions render exactly the same HTML tree, they will not match types if the function references are different, so React will completely remove and recreate that node in the browser.

##### Pitfall: Performing side effects or expensive operations in the render function
```JavaScript
function Counter(props) {
    console.log("Rendering Counter");
    return (
        <div>{props.count}</div>
    )
}

function App() {
    console.log("Rendering App");
    const [count, setCount] = useState(0);
    useEffect(()=>{
        setInterval(()=>{
            setCount(c=>c+1);
        }, 1000);
    });
}
```

#### Misunderstanding: "Calling a state setter function always triggers a re-render"
React will perform an equality check of the new and old states to determine if the function call will trigger a re-render. React seems to use a standard JavaScript 