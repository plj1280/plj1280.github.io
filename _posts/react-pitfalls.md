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

    return (
        <div>
            <Counter count={0}></Counter>
        </div>
    )
}
```
The App updates its "count" state every second, triggering a re-render of itself and its child Counter component. As shown by the console printout, the Counter render function executes every second, even though it does not depend on the state variable and its props have not changed. Note that React correctly determines the browser DOM does not need to be updated, so this excessive re-rendering is only problematic if the render functions are expensive or cause side effects.

##### Pitfall: Defining components within a component
```JavaScript
function App() {
    console.log("Rendering App");
    const [count, setCount] = useState(0);
    useEffect(()=>{
        setInterval(()=>{
            setCount(c=>c+1);
        }, 1000);
    });

    function Counter(props) {
        console.log("Rendering Counter");
        return (
            <div>{props.count}</div>
        )
    }

    return (
        <div>
            <Counter count={0}></Counter>
        </div>
    )

```
Due to the restrictions on data flow in React, if state information is used by multiple components it must live in a common ancestor. In a multi-layer hierarchy, this can lead to multiple layers of passing the same prop. I thought I would be clever and define child components *within* the render function of the parent so they could access the state information without it having to be passed as props. This will cause a full browser update of the child components every time the parent re-renders! This is because React compares component nodes in its reconciliation process by function references, and not just the raw HTML tree. If a function is redefined, it will be treated as a completely different component, even if the function definition and output are exactly the same.

#### Misunderstanding: "Calling a state setter function always triggers a re-render"
React will perform an equality check of the new and old states to determine if the function call will trigger a re-render. React seems to use a standard JavaScript strict equality test. This means if the state variable is a number or a string, React will re-render only if the value of the state has changed. If the state variable is an object, React will check if the object *reference* is the same. This means React will not trigger a re-render if you alter the properties of an object but keep using the same object.

##### Pitfall: Setting the state with the same object as the previous state
```JavaScript
function Counter(props) {
    console.log("Rendering Counter");
    return (
        <div>{props.count}</div>
    )
}

function App() {
    console.log("Rendering App");
    const [count, setCount] = useState({count : 0});
    useEffect(()=>{
        setInterval(()=>{
            setCount(old=>{
                old.count +=1;
                return old
            });
        }, 1000);
    });

    return (
        <div>
            <Counter count={count.count}></Counter>
        </div>
    )
} 
```
It's tempting to use a big monolithic object as a state variable so you don't need to deal with multiple calls to useState and multiple setter functions, and instead just read and write to on object's properties. However, as mentioned, React will not detect the state change if the object reference stays the same. In order to use objects as state variables in React, you need to create a new copy of the object every time the state is updated. Object destructuring syntax is great for this use case. If you don't want to have to copy all state data with every change, you'll need to separate your data and use multiple useState calls.