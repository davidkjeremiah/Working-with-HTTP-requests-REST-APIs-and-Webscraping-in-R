HTTP Requests in R
================
2022-08-01

## Overview of HTTP

When the client uses a web page, your browser sends an HTTP request to
the server where the page is hosted. The server tries to find the
desired resource such as the home page (index.html).

If your request is successful, the server will send the resource to the
client in an HTTP response; this includes information like the type of
the resource, the length of the resource, and other information.

In this lab, we try to gain an understanding of HTTP, and handle the
HTTP requests and response using R.

## The httr library

`httr` is a R library that allows you to build and send HTTP requests,
as well as process HTTP requests easily. We can import the package as
follows (may take less than minute to import):

``` r
library(httr)
```

You can make a GET request via the method get to `www.google.com`:

``` r
# declaring the url
url <- "http://www.google.com"

# making a GET request via the GET method
response <- GET(url)
```

We have the response object `response`. This object has information
about the response, like the status of the request.

We can view the status code using the attribute status

``` r
status_code <- response$status
print(paste('status_code:', status_code))
```

    ## [1] "status_code: 200"

You can also check the headers of the response

``` r
response_headers <- response$headers
print(response_headers)
```

    ## $date
    ## [1] "Mon, 01 Aug 2022 19:24:57 GMT"
    ## 
    ## $expires
    ## [1] "-1"
    ## 
    ## $`cache-control`
    ## [1] "private, max-age=0"
    ## 
    ## $`content-type`
    ## [1] "text/html; charset=ISO-8859-1"
    ## 
    ## $p3p
    ## [1] "CP=\"This is not a P3P policy! See g.co/p3phelp for more info.\""
    ## 
    ## $`content-encoding`
    ## [1] "gzip"
    ## 
    ## $server
    ## [1] "gws"
    ## 
    ## $`content-length`
    ## [1] "6454"
    ## 
    ## $`x-xss-protection`
    ## [1] "0"
    ## 
    ## $`x-frame-options`
    ## [1] "SAMEORIGIN"
    ## 
    ## $`set-cookie`
    ## [1] "1P_JAR=2022-08-01-19; expires=Wed, 31-Aug-2022 19:24:57 GMT; path=/; domain=.google.com; Secure"
    ## 
    ## $`set-cookie`
    ## [1] "AEC=AakniGOXQ7ybrBde7nOnKezLxztJ4kEW_4Y9FPOn1iElD1xTbvT_TEo8MlM; expires=Sat, 28-Jan-2023 19:24:57 GMT; path=/; domain=.google.com; Secure; HttpOnly; SameSite=lax"
    ## 
    ## $`set-cookie`
    ## [1] "NID=511=er8yNNpwP0P6B2baCs97JVTEyvF8XV9nT9b2LRcI75-e1WAfqDS_BdcgX8ESQ4c0pdMF5gcyfT7CZP5P4SADgvl0uvi2qsS5Y96OnTCu2_QPQfZzS10nGyS6ZbjB1izT3gLdASAkiqb3nIef-6zbIesgyRWnCCq12E-QwGqOQ5c; expires=Tue, 31-Jan-2023 19:24:57 GMT; path=/; domain=.google.com; HttpOnly"
    ## 
    ## attr(,"class")
    ## [1] "insensitive" "list"

We can obtain the date the request was sent using the key Date

``` r
print(paste('request_date:', response_headers$date))
```

    ## [1] "request_date: Mon, 01 Aug 2022 19:24:57 GMT"

Content-Type indicates the type of data (most likely the one you’ll be
scraping):

``` r
print(paste('content_type:', response_headers$`content-type`))
```

    ## [1] "content_type: text/html; charset=ISO-8859-1"

Let’s find the content-length attribute

``` r
print(response_headers$`content-length`)
```

    ## [1] "6454"

To obtain the original request, you can view it via response object:

``` r
print(response$request$headers)
```

    ##                                             Accept 
    ## "application/json, text/xml, application/xml, */*"

Now, let’s get the content of HTTP response - which is basically the
html document of `www.google.com`.

``` r
print(content(response))
```

    ## {html_document}
    ## <html itemscope="" itemtype="http://schema.org/WebPage" lang="en-NG">
    ## [1] <head>\n<meta content="text/html; charset=UTF-8" http-equiv="Content-Type ...
    ## [2] <body bgcolor="#fff">\n<script nonce="CkSOkIwfOPm6MFjyrDcbwg">(function() ...

You can load other types of data for non-text requests like images,
consider the URL of the following image:

``` r
image_url <- "https://images.pexels.com/photos/1413412/pexels-photo-1413412.jpeg?cs=srgb&dl=pexels-giorgio-de-angelis-1413412.jpg&fm=jpg"
```

``` r
response_img <- GET(image_url)
```

We can look at the response header:

``` r
response_img_headers <- response_img$headers
```

Having the header’s object, we can we can see the ‘Content-Type’, which
is an image

``` r
print(paste('Content Type:', response_img_headers$`content-type`))
```

    ## [1] "Content Type: image/jpeg"

An image is a response object that contains the image as a bytes-like
object. As a result, we must save it using a file object. First, we
specify the file path and name

``` r
image <- content(response_img, "raw")
writeBin(image, 'motorcycle.jpeg')
```

Notes: You should be able to find the ‘motocycle.jpeg’ in your current
working directory

## Get Request with URL Parameters

You can also add URL parameters to HTTP GET request to filter resources.
For example, instead of return all users from an API, I only want to get
the user with id 1. To do so, I can add a URL parameter like
`userid = 1` in my GET request.

Let’s see an GET example with URL parameters:

Suppose we have a simple GET API with base URL for `http://httpbin.org/`

``` r
url_get <- 'http://httpbin.org/get'
```

and we want to add some URL parameters to above GET API. To do so, we
simply create a named list with parameter names and values:

``` r
query_params <- list(name = "Yan", ID = "123")
```

Then passing the list `query_params` to the `query` argument of the
GET() function.

It basically tells the GET API, “I only want to get resources with name
equals Yan and id equals 123”.

OK, let’s make the GET request to ‘<http://httpbin.org/get>’ with the
two parameters

``` r
response_ <- GET(url_get, query=query_params)
print(response_)
```

    ## Response [http://httpbin.org/get?name=Yan&ID=123]
    ##   Date: 2022-08-01 19:25
    ##   Status: 200
    ##   Content-Type: application/json
    ##   Size: 424 B
    ## {
    ##   "args": {
    ##     "ID": "123", 
    ##     "name": "Yan"
    ##   }, 
    ##   "headers": {
    ##     "Accept": "application/json, text/xml, application/xml, */*", 
    ##     "Accept-Encoding": "deflate, gzip", 
    ##     "Host": "httpbin.org", 
    ##     "User-Agent": "libcurl/7.64.1 r-curl/4.3.2 httr/1.4.2", 
    ## ...

We can print out the updated `URL` and see the attached URL parameters.

``` r
print(response_$url)
```

    ## [1] "http://httpbin.org/get?name=Yan&ID=123"

After the base URL `http://httpbin.org/get`, you can see the URL
parameters `name=Yan&ID=123` are seperated by `?`

## Post Requests

Like a GET request a `POST` is used to send data to a server in a
request body. In order to send the Post Request in R in the URL we
change the route to POST, like this:

``` r
url_post <- 'http://httpbin.org/post'
```

This endpoint will expect data as a file or as a form. It is a
convenient way to configure an HTTP request to send data to a server.

To make a POST request we use the POST() function, the list body is
passed to the parameter body :

``` r
body <- list(code_love='I love R', name='David')
response_post <-POST('http://httpbin.org/post', body = body)
print(response_post)
```

    ## Response [http://httpbin.org/post]
    ##   Date: 2022-08-01 19:25
    ##   Status: 200
    ##   Content-Type: application/json
    ##   Size: 611 B
    ## {
    ##   "args": {}, 
    ##   "data": "", 
    ##   "files": {}, 
    ##   "form": {
    ##     "code_love": "I love R", 
    ##     "name": "David"
    ##   }, 
    ##   "headers": {
    ##     "Accept": "application/json, text/xml, application/xml, */*", 
    ## ...
