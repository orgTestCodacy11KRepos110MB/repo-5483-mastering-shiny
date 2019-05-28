# Basic UI


```r
source("common.R")
```

## Introduction

Now that you've got a basic app under your belt, we're going to explore the details that make it tick. As you saw in the previous chapter, Shiny encourages separation of the code that generates your user interface (`ui`) from the code that drives your app's behavior (`server`). In this chapter, we'll dive deeper into the UI side of things.

All web pages are constructed out of HTML, and Shiny user interfaces are no different--although they are generally expressed using R, their ultimate job is to produce HTML. As a Shiny app author, you can use high-level layout functions in R that keep you far away from the details of HTML. You can also work with HTML tags directly to achieve any level of customization you want. These approaches are by no means exclusive; you can mix high-level functions with low-level HTML as much as you like.

This chapter will focus on some of the high-level R functions that Shiny provides for input, output, and layout, while Chapter \@ref(advanced-ui) will delve into using lower-level features for authoring HTML directly.

### Prerequisites {-}


```r
library(shiny)
```

### Outline {-}

* Section \@ref(inputs) covers input controls.

* Section \@ref(outputs) covers output containers.

* Section \@ref(layout) covers layout functions, which you can use 
  to arrange inputs and outputs.

## Inputs {#inputs}

As we saw in the previous chapter, functions like `sliderInput()`, `selectInput()`, `textInput()`, and `numericInput()` are used to insert input controls into your UI.

The first parameter of an input function is always the `inputId`; this is a simple string made up of numbers, letters, and underscores (no spaces, dashes, periods, or other special characters allowed!). Most input function have a second parameter, `label`, that is used to create a human-readable label for the control. Any remaining parameters are specific to the particular input function, and can be used to customize the input control.

For example, a typical call to `sliderInput` might look something like this:


```r
sliderInput("min", "Limit (minimum)", min = 0, max = 100, value = 50)
```

In the server function, the value of this slider would be accessed via `input$min`. Note that we omit the argument names for `inputId` and `label`, and use the full names for all other arguments.



It's absolutely vital that each input have a *unique* ID. Using the same ID value for more than one input or output in the same app will result in errors or incorrect results.

Shiny itself comes with a variety of input functions out of the box:

- To collect numeric values, you can create a slider with `sliderInput()`,
  or a constraind textbox with `numericInput()`.

- To collect free text use `textInput()`, and to input password use
  `passwordInput()`.

- To select from a set of prespecified values, use `selectInput()`
  or `radioButton()`.
  
- To collect the answer to yes/no questions, try `checkboxInput()` 
  or `checkboxGroupInput()`.

- To pick a date or a range of dates, try `dateInput()` and
  `dateRangeInput()`.

- To let the user upload a file, use `fileInput()`.

Each input function has its own unique look and functionality, and takes different arguments. But they all share the same two important properties:

* They __take__ a unique input ID,
* They __expose__ values to server function using the corresponding 
  element of the `input` object.

## Outputs {#outputs}

Output functions are used to tell Shiny _where_ and _how_ to place outputs that are defined in the app's server. Like inputs, outputs take a unique ID as their first argument, and are paired with server code. Outputs generally start out as empty rectangles and need to be fed data from the server in order to actually appear. 

Shiny comes with a number of built-in output functions that provide useful tools:

* Show the user images with `imageOutput()`, and plots with `plotOutput()`.
* Output text with `textOutput()`, and code with `verbatimTextOutput()`.
* Generate tables of data with `tableOutput()` and `dataTableOutput()` 

Note that `inputId`s and `outputId`s share the same names, and their IDs must be unique among all inputs _and_ outputs. This is because some controls can serve as both inputs _and_ outputs, as you'll learn about in Chapter XYZ.

## Layouts {#layout}

Shiny includes several classes of UI functions that are used to layout inputs and outputs within the app. This chapter focuses on the built-in components centered around `fluidPage()` and `fixedPage()` as these provide the structure seen in the most common style of shiny apps. In future chapters you'll learn about other layout families like dashboards and dialog boxes.

Layouts are composed hierarchically. When you see code like this:


```r
fluidPage(
  titlePanel("Hello Shiny!"),
  sidebarLayout(
    sidebarPanel(
      sliderInput("obs", "Observations:", min = 0, max = 1000, value = 500)
    ),
    mainPanel(
      plotOutput("distPlot")
    )
  )
)
```

First skim it by focusing on the hierarchy of the function calls:


```r
fluidPage(
  titlePanel(),
  sidebarLayout(
    sidebarPanel(
      sliderInput("obs")
    ),
    mainPanel(
      plotOutput("distPlot")
    )
  )
)
```

Even without knowing anying about the layout functions you can read the function names to guess what this app is going to look like. You can imagine that this is going to generate a classic website design: a title bar at top, followed by a sidebar (containing a slider) and a main panel containing a plot.

### Page functions

The first layout function you'll encounter in the UI is a page function. Page functions don't do anything by themselves, but set up the HTML, CSS, and JS that the page needs. The most common page function is `fluidPage()`:


```r
fluidPage(..., title = NULL, theme = NULL)
```

`fluidPage()` sets up your page to use the Bootstrap CSS framework, <https://getbootstrap.com>. Bootstrap provides your web page with attractive settings for typography and spacing, and also preloads dozens of CSS rules that can be invoked to visually organize and enhance specific areas of your UI. We'll take advantage of quite a few of these Bootstrap rules as we proceed through this chapter.

The "fluid" in `fluidPage` means the page's content may resize its width (but not height) as the size of the browser window changes. The other option is `fixedPage()`, which means the page contents will never exceed 960 pixels in width.

Technically, this is all you need: you can put inputs and outputs directly inside `fluidPage()`:


```r
fluidPage(
  sliderInput("min", "Limit (minimum)", min = 0, max = 100, value = 50),
  plotOutput("distPlot")
)
```

This is fine for very simple examples, but if you want your app to look good, you'll need to use more layout functions to define the basic structure of the app. Here I'll introduce you to two common structures: a page with sidebars, and a multirow app.

### Page with sidebar

A great structure for simple shiny apps is a two-column layout where the interactive controls appear in a sidebar on the left, and the results appear in the right. This is easy to construct with `sidebarLayout()` and friends. The basic code structure looks like this:


```r
fluidPage(
  headerPanel(
    # app title/description
  ),
  sidebarLayout(
    sidebarPanel(
      # inputs
    ),
    mainPanel(
      # outputs
    )
  )
)
```

And generates an app with this basic structure:

![](diagrams/basic-ui/sidebar.png)<!-- -->

### Multi-row

You'll need greater flexibilty for more complex apps, and the next step up in complexity in the multirow layout. This is defined by any number of `fluidRow()` calls, each of which contains one or more `column()`s. The basic structure looks like this:


```r
fluidPage(
  fluidRow(
    column(4, 
      ...
    ),
    column(8, 
      ...
    )
  ),
  fluidRow(
    column(6, 
      ...
    ),
    column(6, 
      ...
    )
  )
)
```

which generates a layout like this:

![](diagrams/basic-ui/multirow.png)<!-- -->

Column widths must add up to 12, but this still gives you substantial flexibility. You can easily create 2-, 3-, or 4- column layouts (more than that starts to get cramped), or sidebars that are narrower or wider than the default in `sidebarLayout()`.