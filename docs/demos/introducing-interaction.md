<!-- INJECT:"IntroducingInteraction" -->

# Introducing interaction

## 1. Interact with bar charts on hover

React-Vis has many methods to [handle interaction](http://uber.github.io/react-vis/#/documentation/general-principles/interaction). 
We're already using the state of our app to store our data and interaction with the deck.gl components, so let's use it for react-vis interaction as well. 

In app.js, let's add this property to our initial state,

```js
constructor(props) {
    super(props);
    this.state = {
      
// all previous properties

      highlightedHour: null
    };
  }
```

then in the render method:

```js
<Charts {...this.state}
  highlight={(highlightedHour) => this.setState({highlightedHour})}
/>
```

That's pretty classic - we create a way to change the state and an initial value for the property we're interested in.


Now in charts.js, we're going to do the following changes:

we add arguments to Charts:

```js
function Charts({
  highlight,
  highlightedHour,
  pickups
})
```

Then, before the return statement: 

```js
const data = pickups.map(d => ({
  ...d, color: d.hour === selectedHour ? '#19CDD7' : '#125C77'
});
```

And finally, in the VerticalBarSeries component:

```js
<VerticalBarSeries
  colorType="literal"
  data={data}
  onValueMouseOver={(d) => highlight(d.hour)}
/>
```

<!-- INSERT:"HoverInteraction" -->

We are getting which hour is highlighted from the state of the parent component, and a callback method to change it. 

We are now integrating that information to prepare a dataset: we're going to add some color information to it. If a bar is highlighted, we're giving it a special color.

In VerticalBarSeries, the onValueMouseOver is the way to plug our callback to an interaction event. When a user will mouseover a bar of the series, highlight will be called.

When I prepared the dataset for the series, pickups, I provided an x and a y value for each mark on the series, which is required by React-Vis. However, I can provide all the attributes I want to my dataset. I chose to include an "hour" property which corresponds to the integer value of the hour when a pickup happened. 

onValueMouseOver passes the object which corresponds to the mark which is highlighted, with all its properties. I can then pass the hour property to that highlight callback. 

I also changed the colorType to be "literal". There are many ways to pass color to a react-vis series, but if we pass explicit color values in the dataset, we must signal it to the series. 

```js
<XYPlot
    margin={{left: 40, right: 0, top: 0, bottom: 20}}
    height={140}
    width={280}
  >
  <LineSeries data={pickups} color='#0080FF' />
  <LineSeries data={dropoffs} color='#FF0080' />
  <XAxis
    tickValues={[0, 6, 12, 18, 24]}
    tickFormat={(d) => (d % 24) >= 12 ? (d % 12 || 12) + 'PM' : (d % 12 || 12) + 'AM'}
  />
  <YAxis tickFormat={(d) => (d / 100).toFixed(0) + '%'}
  />
</XYPlot>
```

## 2. Fine-tuning: handling mousing out of the chart and clicks. 

For now the last highlighted bar remains highlighted even if the cursor leaves the chart. We can fix that by adding the following property to XYPlot: 

```js
<XYPlot
  ...
  onMouseLeave={() => highlight(null)}
/>
```

But eventually we'd like to leave a bar selected while we mouse over elsewhere on the chart. So, we'd like to handle clicks. 

Let's go back to app.js and make the following changes:

```js
constructor(props) {
    super(props);
    this.state = {
      
// all previous properties

      selectedHour: null
    };
  }
```

and in the render method: 

```js
  <Charts {...this.state}
    highlight={(highlightedHour) => this.setState({highlightedHour})}
    select={(hour) =>
      this.setState({
        selectedHour: hour === this.state.selectedHour ? null : hour
      })
    }
  />
```

Now, back to charts.js: let's change the beginning of Charts

```js
export default function Charts({
  highlight,
  highlightedHour,
  pickups,
  select,
  selectedHour
  }) {
  if (!pickups) {
    return (<div style={charts}/>);
  }
  const data = pickups.map(d => {
    let color = '#125C77';
    if (d.hour === selectedHour) {
      color = '#19CDD7';
    }
    if (d.hour === highlightedHour) {
      color = '#17B8BE';
    }
    return {...d, color};
  });
```

And in the VerticalBarSeries component:

```js
  <VerticalBarSeries
    colorType="literal"
    data={data}
    onValueMouseOver={(d) => highlight(d.hour)}
    onValueClick={(d) => select(d.hour)}
    style={{cursor: 'pointer'}}
  />
```

onValueClick is to onValueMouseOver what click is to mouse over. 
We can change the style of the cursor to pointer by passing a style property, that's a nice way to signal that an element is clickable. 

If the user clicks on a bar twice, it will be unselected.