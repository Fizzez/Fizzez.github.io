---
title: "How to use Ployly.js in Jupyter Notebook"
date: 2020-09-23 08:02:55
tags:
- Python
- JavaScript
- Notebook
- Visualization
- English
---

> 本文中文版：
>
> https://fizzez.github.io/2020/09/20/nb-magic-js/

# Description

Plotly is one of the best tools I experienced to visualize data in Jupyter Notebook. While I am making a simple APP with JavaScript and Plotly.js, I found the API names of Plotly.py and Plotly.js are quite similar to each other. This new find inspired me to use Plotly.js in Jupyter Notebook, in which way I think is more flexible (maybe).

# Method

Before start implementing anything, I searched for existing solutions. The method by HylaruCoder is one of the best (see Ref.[1]). According to HylaryCoder, RequireJS is a JS tool supported originally by Jupyter Notebook to dynamically load  a JS module, which enable us to load plotls.js from the [CDN address provided officially by Plotly](https://plotly.com/javascript/getting-started/#plotlyjs-cdn).

Okay the code to load Plotly.js is like the following. By the way, the magic command to tell Jupyter Notebook to execute a JS script is `%%javascript` or `%%js`:

```js
%%js
requirejs.config({
    paths: {
        plotly: 'https://cdn.plot.ly/plotly-1.55.2.min.js?noext',
    }
});
```

Then we'll need a `<div>` element in the HTML page for sure, to contain the plot. The element is appended with command `element.append()`. Given the example data and call Plotly.js API like the following code, we could have the example figure plotted.

```js
%%js
element.append('<div id="plotly_graph" margin: 0 auto"></div>');
(function(element) {
    requirejs(['plotly'], function(Plotly) {
        var trace1 = {
          x: [1, 2, 3, 4],
          y: [10, 15, 13, 17],
          mode: 'markers',
          type: 'scatter'
        };

        var trace2 = {
          x: [2, 3, 4, 5],
          y: [16, 5, 11, 9],
          mode: 'lines',
          type: 'scatter'
        };

        var trace3 = {
          x: [1, 2, 3, 4],
          y: [12, 9, 15, 12],
          mode: 'lines+markers',
          type: 'scatter'
        };

        var data = [trace1, trace2, trace3];

        Plotly.newPlot(document.getElementById('plotly_graph'), data);
    });
})(element);
```

Wait, an error was thrown when I tried to execute the 2 code cells separately. The error message was from RequireJS, saying `Failed to load resource: the server responded with a status of 404 (Not Found)`. Thanks to the answer from Ref.[2], the error disappeared after I merged the 2 code cells and execute. The plotted figure shoule like this:

![Plotly.js in Jupyter Notebook](/images/nb-magic-js/eg_graph.png) 



In the example above, the example data for plotting is given in JS scripts. Then we'll need to consider how to pass the processed data from Python cells to JS cells.

The answer is to use JSON (via the HTML page). Here's the Python script to pass the generated data to HTML page ( `window.plotly_json`):

```python
import json
import numpy as np
from IPython.core.display import Javascript

eval_x = np.linspace(0, 3 * np.pi, 100)

trace_1 = {'x': eval_x.tolist(), 
           'y': np.sin(eval_x).tolist(), 
           'mode': 'lines+markers', 
           'type': 'scatter',
           'line': {'width': 3}, 
           'marker': {'symbol': 'cross'}}
trace_2 = {'x': eval_x.tolist(), 
           'y': np.cos(eval_x).tolist(), 
           'mode': 'lines+markers',
           'type': 'scatter',
           'line': {'width': 1.5}, 
           'marker': {'symbol': 'circle'}}

plotly_json = {'data': [trace_1, trace_2]}

Javascript(f"window.plotly_json = JSON.parse('{json.dumps(plotly_json, ensure_ascii=False)}')")
```

So in this script, 2 plot traces in `dict` type are generated (naming of variables here are very like those in Plotly.js). Then serialize the dictionary to JSON format and pass to `window.plotly_json`. (Notice: ndarray object is not be able to be serialized to JSON, use `tolist()` to conver to list in advance.)

Then the following script shows how to read the JSON data from HTML page and visualize with Plotly.js in the way introduced at the beginning.

```js
%%js
element.append('<div id="plotly_graph_json" margin: 0 auto"></div>');
(function(element) {
    requirejs(['plotly'], function(Plotly) {
        Plotly.newPlot(document.getElementById('plotly_graph_json'), plotly_json.data);
    });
})(element);
```

The result will be like this:

![Plotly.js in Jupyter Notebook](/images/nb-magic-js/eg_graph_json.png)



All the scripts and results are given together in this Jupyter Notebook:

> https://github.com/Fizzez/playground/blob/master/python/notebook-magic-html-js.ipynb

# Ref

1. [如何优雅地在 IPython Notebook 中使用 ECharts - HylaruCoder](https://www.zhihu.com/people/twocucao)
2. [Require.js bug random Failed to load resource - Stack Overflow](https://stackoverflow.com/questions/17026036/require-js-bug-random-failed-to-load-resource)