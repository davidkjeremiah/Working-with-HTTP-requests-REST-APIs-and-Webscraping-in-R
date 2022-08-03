webscraping_in_R
================
David Jeremiah
2022-08-02

## Objective

This lab is aimed at performing basic webscraping using the `rvest`
package

## Table of Contents

1.  Overview of HTML
2.  The `rvest` library

## Overview of HTML

-   HTML stands for `Hypertext Markup Language` and it is used mainly
    for writing web pages.

-   An HTML page consists of many organized HTML nodes or elements that
    tell a browser how to render its content.

-   Each node or element has a `start tag` and an `end tag` with the
    same name and wraps some textual content.

One key feature of HTML is that nodes can be nested within other nodes,
organizing into a tree-like structure like the folders in a file system.
Below is a basic HTML node structure:

-   `<html>` node is the root node,

-   `<html>` node has two children: `<head>` and `<body>`.

-   Since the `<head>` and `<body>` nodes have the same parent `<html>`
    node they are siblings to each other.

-   Similarly, the `<body>` node has two child nodes, the `<h1>` and
    `<p>` nodes.

## The `revest` library

The `rvest` package is a popular web scraping package for R.

After rvest reads an HTML page, you can use the tag names to find the
child nodes of the current node.

``` r
library(rvest)
```

We also need to import httr library to get some HTML pages by sending
HTTP GET request

``` r
library(httr)
```

First let’s warm-up by reading HTML from the following character
variable simple_html_text

``` r
# A simple HTML document in a character variable
simple_html_text <- "
<html>
    <body>
        <p>This is a test html page</p>
    </body>
</html>"
```

Then use the read_html function to create the HTML root node, i.e., the
html node by loading the simple_html_text

``` r
root_node <- read_html(simple_html_text)
print(root_node)
```

    ## {html_document}
    ## <html>
    ## [1] <body>\n        <p>This is a test html page</p>\n    </body>

You can also check the type of root_node

``` r
print(class(root_node))
```

    ## [1] "xml_document" "xml_node"

You can see the class is xml_node because rvest load HTML pages and
convert them using XML format internally. XML has similar parent-child
tree structure but more suitable for storing and tranporting data than
HTML.

Next, let’s try to create a HTML node by loading a remote HTML page
given a URL

``` r
ibm_html_node <- read_html("http://google.com")
ibm_html_node
```

    ## {html_document}
    ## <html itemscope="" itemtype="http://schema.org/WebPage" lang="en-NG">
    ## [1] <head>\n<meta content="text/html; charset=UTF-8" http-equiv="Content-Type ...
    ## [2] <body bgcolor="#fff">\n<script nonce="LVrmI7FY_HDQCREAXXlEDQ">(function() ...

Sometimes you want to download some HTML pages and analyze them offline,
you could use download.file to do so:

``` r
download.file('https://www.r-project.org', destfile = 'r.html')
```

``` r
html_node <- read_html('r.html')
print(html_node)
```

    ## {html_document}
    ## <html lang="en">
    ## [1] <head>\n<meta http-equiv="Content-Type" content="text/html; charset=UTF-8 ...
    ## [2] <body>\r\n    <div class="container page">\r\n      <div class="row">\r\n ...

Awesome!

Next, let’s see how to parse and extract data from a specific node(s)
starting from the root node

``` r
simple_html_text <- "
<html>
    <body>
        <p style=\"color:red;\">This is a test html page</p>
    </body>
</html>"

root_node <- read_html(simple_html_text)
print(root_node)
```

    ## {html_document}
    ## <html>
    ## [1] <body>\n        <p style="color:red;">This is a test html page</p>\n    < ...

Get the
<body>
node by using its parent node
<html>

``` r
body_node <- html_node(root_node, "body")
print(body_node)
```

    ## {html_node}
    ## <body>
    ## [1] <p style="color:red;">This is a test html page</p>

You can see it has a child node paragraph
<p>
Let’s get the content of the
<p>

``` r
p_node <- html_node(body_node, "p")
p_content<-html_text(p_node)
print(p_content)
```

    ## [1] "This is a test html page"

The `<p>` node also has `style` attribute with value `color:red;`, which
means we want the browser to render its text using red color. To get an
attribute of a node, we can use a function called
`html_attr("attribute name")`

``` r
style_attr <- html_attr(p_node, "style")
print(style_attr)
```

    ## [1] "color:red;"

In the code cell below, the downloaded `r.html` file (from
`https://www.r-project.org`) has an `<img>` node representing an image
URL to R logo image (a relative path on its web server), let’s try to
find the image URL and download it.

Your need to paste the relative path in `<img>` with the the
`https://www.r-project.org` to get the full URL of the image, and use
the `GET` function to request the image as bytes in the response

``` r
url <- 'https://www.r-project.org'
html_node <- read_html('r.html')

# Get the image node using its root node
img_node <- html_node(html_node, "img")

# Get the "src" attribute of img node, representing the image location
img_relative_path <- html_attr(img_node, "src")
img_relative_path
```

    ## [1] "/Rlogo.png"

``` r
# Paste img_relative_path with 'https://www.r-project.org'
image_path <- paste(url, img_relative_path, sep="")

# Use GET request to get the image
image_response <- GET(image_path)
```

Then use writeBin() function to save the returned bytes into an image
file.

``` r
image <- content(image_response, "raw")
```

``` r
writeBin(image, "r.png")
```

Now, in your currently working directory, you should be able to find a
saved `r.png file`.

In HTML, many tabluar data are stored in `<table>` nodes. Thus, it is
very important to be able to extract data from `<table>` nodes and
preferably convert them into R data frames.

Below is a sample HTML page contains a color table showing the supported
HTML colors, and we want to load it as a R data frame so we can analyze
it using data frame-related operations.

``` r
table_url <- "https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DA0321EN-SkillsNetwork/labs/datasets/HTMLColorCodes.html"
```

Like other HTML nodes, let’s first get the
<table>

node using `html_node` function

``` r
root_node <-read_html(table_url)
table_node <- html_node(root_node, "table")
print(table_node)
```

    ## {html_node}
    ## <table border="1" class="main-table">
    ##  [1] <tr>\n<td>Number </td>\n      <td>Color</td>\n      <td>Color Name</td>\ ...
    ##  [2] <tr>\n<td>1</td>\n      <td style="background:lightsalmon;"> </td>\n     ...
    ##  [3] <tr>\n<td>2</td>\n      <td style="background:salmon;"> </td>\n      <td ...
    ##  [4] <tr>\n<td>3</td>\n      <td style="background:darksalmon;"> </td>\n      ...
    ##  [5] <tr>\n<td>4</td>\n      <td style="background:lightcoral;"> </td>\n      ...
    ##  [6] <tr>\n<td>5</td>\n      <td style="background:coral;"> </td>\n      <td> ...
    ##  [7] <tr>\n<td>6</td>\n      <td style="background:tomato;"> </td>\n      <td ...
    ##  [8] <tr>\n<td>7</td>\n      <td style="background:orangered;"> </td>\n       ...
    ##  [9] <tr>\n<td>8</td>\n      <td style="background:gold;"> </td>\n      <td>g ...
    ## [10] <tr>\n<td>9</td>\n      <td style="background:orange;"> </td>\n      <td ...
    ## [11] <tr>\n<td>10</td>\n      <td style="background:darkorange;"> </td>\n     ...
    ## [12] <tr>\n<td>11</td>\n      <td style="background:lightyellow;"> </td>\n    ...
    ## [13] <tr>\n<td>12</td>\n      <td style="background:lemonchiffon;"> </td>\n   ...
    ## [14] <tr>\n<td>13</td>\n      <td style="background:papayawhip;"> </td>\n     ...
    ## [15] <tr>\n<td>14</td>\n      <td style="background:moccasin;"> </td>\n       ...
    ## [16] <tr>\n<td>15</td>\n      <td style="background:peachpuff;"> </td>\n      ...
    ## [17] <tr>\n<td>16</td>\n      <td style="background:palegoldenrod;"> </td>\n  ...
    ## [18] <tr>\n<td>17</td>\n      <td style="background:khaki;"> </td>\n      <td ...
    ## [19] <tr>\n<td>18</td>\n      <td style="background:darkkhaki;"> </td>\n      ...
    ## [20] <tr>\n<td>19</td>\n      <td style="background:yellow;"> </td>\n      <t ...
    ## ...

Notes: the table node in a messy HTML format. Fortunately, you dont need
to parse it by yourself, `rvest` provides a handy function called
html_table() to convert
<table>

node into R dataframe

``` r
# Extract content from table_node and convert the data into a dataframe
color_data_frame <- html_table(table_node)
head(color_data_frame)
```

    ## # A tibble: 6 x 5
    ##   X1     X2      X3          X4              X5                 
    ##   <chr>  <chr>   <chr>       <chr>           <chr>              
    ## 1 Number "Color" Color Name  Hex Code#RRGGBB Decimal Code(R,G,B)
    ## 2 1      ""      lightsalmon #FFA07A         rgb(255,160,122)   
    ## 3 2      ""      salmon      #FA8072         rgb(250,128,114)   
    ## 4 3      ""      darksalmon  #E9967A         rgb(233,150,122)   
    ## 5 4      ""      lightcoral  #F08080         rgb(240,128,128)   
    ## 6 5      ""      coral       #FF7F50         rgb(255,127,80)

But you could see the table headers were parsed as the first row, no
worries, we could manually fix that

``` r
names(color_data_frame)
```

    ## [1] "X1" "X2" "X3" "X4" "X5"

``` r
names(color_data_frame) <- as.matrix(color_data_frame[1, ])
```

``` r
data_frame <- color_data_frame[-1, ]

head(data_frame)
```

    ## # A tibble: 6 x 5
    ##   Number Color `Color Name` `Hex Code#RRGGBB` `Decimal Code(R,G,B)`
    ##   <chr>  <chr> <chr>        <chr>             <chr>                
    ## 1 1      ""    lightsalmon  #FFA07A           rgb(255,160,122)     
    ## 2 2      ""    salmon       #FA8072           rgb(250,128,114)     
    ## 3 3      ""    darksalmon   #E9967A           rgb(233,150,122)     
    ## 4 4      ""    lightcoral   #F08080           rgb(240,128,128)     
    ## 5 5      ""    coral        #FF7F50           rgb(255,127,80)      
    ## 6 6      ""    tomato       #FF6347           rgb(255,99,71)

``` r
names(color_data_frame)
```

    ## [1] "Number"              "Color"               "Color Name"         
    ## [4] "Hex Code#RRGGBB"     "Decimal Code(R,G,B)"

That’s it for webscraping in R!
