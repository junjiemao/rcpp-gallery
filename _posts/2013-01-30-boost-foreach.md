---
title: Using Boost's foreach macro
author: Kevin Ushey
license: GPL (>= 2)
tags: basics boost
summary: Boost's BOOST_FOREACH can enable a more functional programming style.
layout: post
src: 2013-01-30-boost-foreach.Rmd
---
 
Boost provides a macro, `BOOST_FOREACH`, that allows us to easily iterate
over elements in a container, similar to what we might
do in R with `sapply`.
In particular, it frees us from having to deal with iterators as we do with 
`std::for_each` and `std::transform`. The macro is also compatible with the 
objects exposed by Rcpp.
 
Side note: C++11 has introduced a similar for-each looping construct of the form
 
    for (T &elem : X) { /*do stuff*/ } 
    
However, CRAN does not (at the time this post was initially written) allow C++11 in uploads and 
hence this Boost solution might be preferred if you want to use a for-each 
construct in a package.
 
The `BOOST_FOREACH` macro is exposed when we use `#include <boost/foreach.hpp>`.
Make sure the Boost libraries are in your includepath so that they can be found and
included easily. Because it's a header-only library we don't have to worry
about external dependencies or linking.
 
We'll use a simple example where we square each element in a vector.
 

{% highlight cpp %}
#include <Rcpp.h>
#include <boost/foreach.hpp>
using namespace Rcpp;
 
// We can now use the BH package
// [[Rcpp::depends(BH)]]

// the C-style upper-case macro name is a bit ugly; let's change it
// note: this could cause compiler errors if it conflicts with other includes
#define foreach BOOST_FOREACH
 
// [[Rcpp::export]]
NumericVector square( NumericVector x ) {
  
  // elem is a reference to each element in x
  // we can re-assign to these elements as well
  foreach( double& elem, x ) {
    elem = elem*elem;
  }
  
  return x;
}
{% endhighlight %}
 

{% highlight r %}
square( 1:10 )
{% endhighlight %}



<pre class="output">
 [1]   1   4   9  16  25  36  49  64  81 100
</pre>



{% highlight r %}
square( matrix(1:16, nrow=4) )
{% endhighlight %}



<pre class="output">
     [,1] [,2] [,3] [,4]
[1,]    1   25   81  169
[2,]    4   36  100  196
[3,]    9   49  121  225
[4,]   16   64  144  256
</pre>



{% highlight r %}
## we check that the function handles various 'special' values
x <- c(1, 2, NA, 4, NaN, Inf, -Inf)
square(x)
{% endhighlight %}



<pre class="output">
[1]   1   4  NA  16 NaN Inf Inf
</pre>
 
And a quick benchmark:
 

{% highlight r %}
x <- rnorm(1E5)
library(microbenchmark)
microbenchmark(
  square(x),
  x^2
  )
{% endhighlight %}



<pre class="output">
Unit: microseconds
      expr   min     lq median    uq  max neval
 square(x)  70.0  70.78     71  72.0 1498   100
       x^2 151.5 152.11    153 561.9 2022   100
</pre>



{% highlight r %}
all.equal( square(x), x^2 )
{% endhighlight %}



<pre class="output">
[1] TRUE
</pre>
 
If you are defining your own classes / containers and want them to be compatible 
with one of these for-each constructs, you will need to define some methods for
iteration across these objects. See 
[this post](http://stackoverflow.com/questions/7562356/c11-foreach-syntax-and-custom-iterator) 
on SO for more details.
 
For more information on `BOOST_FOREACH`, check the documentation 
[here](http://www.boost.org/doc/libs/1_52_0/doc/html/foreach.html).
