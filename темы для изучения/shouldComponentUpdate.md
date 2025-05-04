In a comment above _Gabriele Petrioli_ links to the React.memo documentation that explains how to implement shouldComponentUpdate. I was googling combinations of shouldComponentUpdate + useEffect + "react hooks", and this came up as the result. So after solving my problem with the linked documentation I thought I would bring the information here as well.

This is the old way of implementing shouldComponentUpdate:

```javascript
class MyComponent extends React.Component{
  shouldComponentUpdate(nextProps){
    return nextProps.value !== this.props.value;
  }
  render(){
    return (
     <div>{"My Component " + this.props.value}</div>
    );  
 }
}
```

The New React Hooks way:

```javascript
React.memo(function MyComponent (props) {

  return <div>{ "My Component " + props.value }</div>;

}) 
```

I know you were probably asking for more in your question, but for anyone coming from Google looking for how to implement shouldComponentUpdate using React Hooks, there you go.

The documentation is here: [how-do-i-implement-shouldcomponentupdate](https://reactjs.org/docs/hooks-faq.html#how-do-i-implement-shouldcomponentupdate)
