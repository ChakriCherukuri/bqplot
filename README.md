# bqplot

[![Travis](https://travis-ci.com/bqplot/bqplot.svg?branch=master)](https://travis-ci.com/bqplot/bqplot)
[![Documentation](https://readthedocs.org/projects/bqplot/badge/?version=latest)](http://bqplot.readthedocs.org)
[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/bqplot/bqplot/stable?filepath=examples/Index.ipynb)
[![Chat](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/jupyter-widgets/Lobby)

2-D plotting library for Project Jupyter

## Introduction

`bqplot` is a 2-D visualization system for Jupyter, based on the constructs of
the *Grammar of Graphics*.

## Usage

[![Wealth of Nations](./wealth-of-nations.gif)](https://github.com/bqplot/bqplot/blob/master/examples/Applications/Wealth%20Of%20Nations/Bubble%20Chart.ipynb)

In bqplot, every component of a plot is an interactive widget. This allows the user to integrate visualizations with other Jupyter interactive widgets to create rich dashboards with a few lines of Python code.

Two plotting APIs are provided in `bqplot`
- `pyplot` Context-based API similar to Matplotlib's pyplot. `pyplot` provides sensible default choices for most parameters and should be the right starting point for newcomers to bqplot
- `Object Model` Object-oriented API inspired by the constructs of the Grammar of Graphics (figure, marks, axes, scales). This API is for advanced users who want full customization and control.

## Trying it online

To try out bqplot interactively in your web browser, just click on the binder
link:

[![Binder](docs/source/binder-logo.svg)](https://mybinder.org/v2/gh/bqplot/bqplot/stable?filepath=examples/Index.ipynb)

### Dependencies

This package depends on the following packages:

- `ipywidgets` (version >=7.0.0, <8.0)
- `traitlets` (version >=4.3.0, <5.0)
- `traittypes` (Version >=0.2.1, <0.3)
- `numpy`
- `pandas`

### Installation

Using pip:

```
$ pip install bqplot
```

Using conda

```
$ conda install -c conda-forge bqplot
```

If you are using JupyterLab <=2:

```
$ jupyter labextension install @jupyter-widgets/jupyterlab-manager bqplot
```

##### Development installation

For a development installation (requires JupyterLab (version >= 3) and yarn):

```
$ git clone https://github.com/bqplot/bqplot.git
$ cd bqplot
$ pip install -e .
$ jupyter nbextension install --py --overwrite --symlink --sys-prefix bqplot
$ jupyter nbextension enable --py --sys-prefix bqplot
```

Note for developers: the `--symlink` argument on Linux or OS X allows one to
modify the JavaScript code in-place. This feature is not available
with Windows.

For the experimental JupyterLab extension, install the Python package, make sure the Jupyter widgets extension is installed, and install the bqplot extension:

```
$ pip install "ipywidgets>=7.6"
$ jupyter labextension develop . --overwrite
```

Whenever you make a change of the JavaScript code, you will need to rebuild:

```
cd js
yarn run build
```

Then refreshing the JupyterLab/Jupyter Notebook is enough to reload the changes.

##### Running tests

You can install the dependencies necessary to run the tests with:

```bash
    conda env update -f test-environment.yml
```

And run it with for Python tests:

```bash
    pytest
```

And `cd js` to run the JS tests with:

```bash
yarn run test
```

Every time you make a change on your tests it's necessary to rebuild the JS side:

```bash
yarn run build
```

## Usage

### Pyplot

```python
import bqplot.pyplot as plt
import numpy as np

# step1: create a figure
fig = plt.figure(title="Random Walk")

# step2: Add a few marks (they'll be added to the same figure)
x = np.arange(100) # data attribute x
y = np.cumsum(np.random.randn(100)) # data attribute y
plt.plot(x, y) # line mark

# render the figure
fig
```
[![Pyplot Screenshot](/pyplot.png)](https://github.com/bqplot/bqplot/blob/master/examples/Basic%20Plotting/Pyplot.ipynb)

### Object Model
```python
from bqplot import LinearScale, Axis, Lines, Bars, Figure

# create scales 
xs = LinearScale()
ys = LinearScale()

# data attributes
x = np.arange(10)
y1 = np.random.rand(10)
y2 = y1 * 2

# create marks
line = Lines(x=x, y=y1, scales={"x": xs, "y": ys}, colors=["blue"], 
             marker="square", display_legend=True, labels=["Line"])
bar = Bars(
    x=x, y=y2, scales={"x": xs, "y": ys}, colorpadding=0.2, 
    colors=["salmon"], display_legend=True, labels=["Bar"]
)

# create axes
xax = Axis(scale=xs, label="X", grid_lines="none")
yax = Axis(
    scale=ys, orientation="vertical", tick_format="0.1f", label="Y", grid_lines="solid"
)

# create a figure and render it
Figure(marks=[bar, line], axes=[xax, yax])
```
[![Bqplot Screenshot](/bqplot.png)](https://github.com/bqplot/bqplot/blob/master/examples/Marks/Object%20Model/Lines.ipynb)

### Dashboards
`bqplot` can be seamlessly integrated with `ipywidgets` to build rich interactive dashboards. Since `bqplot` uses the same messaging protocols as `ipywidgets`, traits of each can be linked using the `link` or `observe` methods (of `ipywidgets`). Below is a simple example which shows how to use a slider to control the number of bins in a histogram.
```python
import ipywidgets
import bqplot.pyplot as plt
import numpy as np

bins_slider = ipywidgets.IntSlider(description="bins", value=20)
fig = plt.figure()
hist_mark = plt.hist(np.random.randn(1000), bins=20)
plt.grids(fig, "none")

# link the slider's value attribute to the histogram's bins attribute
_ = ipywidgets.jslink((bins_slider, "value"), (hist_mark, "bins"))

# render the slider and the figure vertically using VBox
ipywidgets.VBox([bins_slider, fig])
```
![Bqplot Screenshot](/dashboard.gif)

Examples of sophisticated interactive dashboards can be found in the `bqplot-gallery` [repo](https://github.com/bqplot/bqplot-gallery/tree/main/notebooks)

### Compound Plotting Widgets
The object-oriented API of `bqplot` can be extended to create custom plotting widgets by sub-classing the `Figure` class. These widgets can be composed of custom `Lines`, `Scatter` or other marks as needed. The attributes of composed marks, scales and axes can be modified as needed by accessing them directly.

Below is an example of creating a standalone `Circle` widget.
```python
import ipywidgets
class Circle(Figure):
    def __init__(self, *args, **kwargs):
        super(Figure, self).__init__(*args, **kwargs)
        fill_color = kwargs.get("fill_color", "green")
        self.layout = ipywidgets.Layout(
            width="100px",
            height="100px")
        self.fig_margin = dict(top=5, bottom=5, left=5, right=5)
        
        self.scales = {"x": LinearScale(), "y": LinearScale()}
        x = np.linspace(-1, 1, 500)
        y = np.sqrt(1 - x ** 2)
        self.circle = Lines(x=x, y=[-y, y], 
                            colors=[fill_color],
                            scales=self.scales,
                            fill="inside")
        self.marks = [self.circle]
```

Once the widget is created, we can use it to create custom layouts. Below is an example of traffic lights created from 3 `Circle` widgets.

```python
traffic_lights = ipywidgets.VBox([
    Circle(fill_color="red"),
    Circle(fill_color="orange"),
    Circle(fill_color="green")])

traffic_lights
```
<img src="plotting_widgets.png" style="width: 75px"/>

Some examples of plotting widgets bundled with `bqplot` can be found [here]

## Documentation

Full documentation is available at https://bqplot.readthedocs.io/

## Install a previous bqplot version (Only for JupyterLab <= 2)

In order to install a previous bqplot version, you need to know which front-end version (JavaScript) matches with the back-end version (Python).

For example, in order to install bqplot `0.11.9`, you need the labextension version `0.4.9`.

```
$ pip install bqplot==0.11.9
$ jupyter labextension install bqplot@0.4.9
```

Versions lookup table:

| `back-end (Python)` | `front-end (JavaScript)` |
|---------------------|--------------------------|
| 0.12.14             | 0.5.14                   |
| 0.12.13             | 0.5.13                   |
| 0.12.12             | 0.5.12                   |
| 0.12.11             | 0.5.11                   |
| 0.12.10             | 0.5.10                   |
| 0.12.9              | 0.5.9                    |
| 0.12.8              | 0.5.8                    |
| 0.12.7              | 0.5.7                    |
| 0.12.6              | 0.5.6                    |
| 0.12.4              | 0.5.4                    |
| 0.12.3              | 0.5.3                    |
| 0.12.2              | 0.5.2                    |
| 0.12.1              | 0.5.1                    |
| 0.12.0              | 0.5.0                    |
| 0.11.9              | 0.4.9                    |
| 0.11.8              | 0.4.8                    |
| 0.11.7              | 0.4.7                    |
| 0.11.6              | 0.4.6                    |
| 0.11.5              | 0.4.5                    |
| 0.11.4              | 0.4.5                    |
| 0.11.3              | 0.4.4                    |
| 0.11.2              | 0.4.3                    |
| 0.11.1              | 0.4.1                    |
| 0.11.0              | 0.4.0                    |

## Development

See our [contributing guidelines](CONTRIBUTING.md) to know how to contribute and set up a development environment.

## License

This software is licensed under the Apache 2.0 license. See the [LICENSE](LICENSE) file
for details.
