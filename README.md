# Rosetta Stone: Project Euler

~~~js
var tape = require("tape");
~~~

## Problem 1
> If we list all the natural numbers below 10 that are multiples of 3 or 5, we get 3, 5, 6 and 9. The sum of these multiples is 23.
>
> Find the sum of all the multiples of 3 or 5 below 1000.



~~~js
// Via http://stackoverflow.com/a/39930823/500207
function range(start, stop, step = 1) {
  if (typeof stop === 'undefined') {
    stop = start;
    start = 0;
  }
  if (start > stop && step > 0) {
    return [];
  }
  return Array.from(new Array(Math.floor((stop - start) / step)),
                    (x, i) => start + i * step);
}

~~~

~~~js
// tape("step in the wrong direction: empty", test=>{range(-3, -10);});

~~~
