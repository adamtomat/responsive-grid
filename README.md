# Responsive Grid

Responsive Grid is a simple but powerful grid layout tool build in `sass`. It exposes as little or as much about the underlying math as you need, providing incredible flexibility.

Precision matters to us. The main reason we use a grid is to ensure things line up vertically on a page. However, many grid frameworks only line up correctly if every _row_ is the same width. This is because columns are percentage based, relative to their container.

**A simple example (excluding gutters):**

If a container is `1200px` wide, 1 column would be ~`8.33%` wide. That's about `100px`.

If a container is `900px` wide, 1 column would still be ~`8.33%` wide. Which is about `75px`.

For things to look right, 1 column needs to be the same width regardless of the width of its container.

**How Responsive Grid can handle the previous example**

If a container is `1200px` wide, 1 column would be ~`8.33%` wide. That's about `100px`.

If a container is `900px` wide, 1 column would actually be ~`11.11%` wide. Which is about `100px`. **Now it's the same width as above**.

## Basic Usage

Responsive grids are made up of **rows** and **columns**. Let's set up our grid:

```sass
$my-grid: (
  prefix: 'custom',
  columns: 12,
  gutter: 16px
)
```

### Making a row

```sass
.my-row {
  @include grid-row($my-grid);
}
```

This row will clearfix by default, which adds a `before` and `after` to the row. If you're using `flexbox` this can get in the way, so you can turn off clearfixing by passing `false` as the 2nd parameter:

```sass
.my-row {
  // Turn off clearfixing
  @include grid-row($my-grid, true);
  
  // Using flexbox to make all columns the same height
  display: flex;
  align-items: stretch;
}
```

### Making a column

```sass
.my-column-half {
  // Six columns wide
  @include grid-col($my-grid, 6);
}
```

Here's an example of how you can nest grids

```sass
.my-row {
  @include grid-row($my-grid);
}
  
.my-column-half {
  // Six columns wide
  @include grid-col($my-grid, 6);
  
  // A column inside of a column
  .my-column-nested {
    // Three columns wide. We tell it that its container is 6 columns wide, to ensure it's the correct size
    @include grid-col($my-grid, 3, 6);
  }
}
```

### Changing the column size between media queries

There are 2 ways to achieve this:

```sass
.my-column {
  // 2 in a row on small screens
  @include grid-col($my-grid, 6);
}

@media screen and (min-width: 800px) {
  .my-column {
    // At 800px, have 3 in a row
    @include grid-col($my-grid, 4);
  }
}
```

The problem with this approach is that `grid-col` will spit out something like this:

```css
.my-column {
  // 2 in a row on small screens
  width: 50%;
  
  float: left;
  padding-left: 8px;
  padding-right: 8px;
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
  box-sizing: border-box;
}

@media screen and (min-width: 800px) {
  .my-column {
    // At 800px, have 3 in a row
    width: 33.333%;
    
    float: left;
    padding-left: 8px;
    padding-right: 8px;
    -webkit-box-sizing: border-box;
    -moz-box-sizing: border-box;
    box-sizing: border-box;
  }
}
```

Notice how the only difference in the media query is the width? Everything else is the same, and will be inherited. Doing this a lot will bloat your css unnecessarily.

The better way is just to change the width property in the media query:

```sass
.my-column {
  // 2 in a row on small screens
  @include grid-col($my-grid, 6);
}

@media screen and (min-width: 800px) {
  .my-column {
    // At 800px, have 3 in a row
    // Use the grid-col-width function to work out what the width should be for 4 columns
    // We are also passing in the number of columns that our grid has (i.e. 12)
    width: grid-col-width(4, map-get($my-grid, columns));
  }
}
```

Now the css looks something like this:

```css
.my-column {
  // 2 in a row on small screens
  width: 50%;
  
  float: left;
  padding-left: 8px;
  padding-right: 8px;
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
  box-sizing: border-box;
}

@media screen and (min-width: 800px) {
  .my-column {
    // At 800px, have 3 in a row
    width: 33.333%;
  }
}
```

Much better!

## Why do I have to write this stuff myself?

A lot of grid frameworks will spit out a bunch of classes that are ready to be used on your markup. For example:

`<div class="lg-col-6">`

You can also add a bunch of different rules easily for different screen sizes:

`<div class="xs-col-12 sm-col-6 md-col-4 lg-col-2">`

Responsive Grid lets you do this too thanks to the `grid` mixin. Here's how you can generate all the classes required:

```sass
$xs-grid: ( prefix: 'xs', columns: 12, gutter: 16px );
$sm-grid: ( prefix: 'sm', columns: 12, gutter: 16px );
$md-grid: ( prefix: 'md', columns: 12, gutter: 16px );
$lg-grid: ( prefix: 'lg', columns: 12, gutter: 16px );

@include grid($xs-grid);

@media screen and (min-width: 500px) {
    @include grid($sm-grid);
}

@media screen and (min-width: 700px) {
    @include grid($md-grid);
}

@media screen and (min-width: 900px) {
    @include grid($lg-grid);
}
```

The `grid` mixin will generate row and column classes for the grid you pass in.

```scss
// A class for your row
.xs-row {
  @include grid-row($xs-grid);
}

.xs-col-1 {
  @include grid-col($grid, 1);
}

.xs-col-2 {
  @include grid-col($grid, 2);
}

// ...etc
```

At Rareloop, we don't like this approach very much, so we only use the mixins and functions available. Here's why:

1) It spits out a lot of `css`, a lot of which you're never using

2) It forces you into **snap-points**, not **break-points**. The difference? With snap-points you're defining known pixel values at which you change layout, which is consistent across your site. However, with break-points you add custom media queries for a component _when its own layout breaks_. Break-points make your layout a lot more fluid and ensures that nothing ever looks cramped or stretched.

3) We don't like layout classes on markup
