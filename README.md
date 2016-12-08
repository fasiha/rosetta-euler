# Rosetta Stone: Project Euler

~~~js
var tape = require("tape");
~~~

## Problem 1
> If we list all the natural numbers below 10 that are multiples of 3 or 5, we get 3, 5, 6 and 9. The sum of these multiples is 23.
>
> Find the sum of all the multiples of 3 or 5 below 1000.

### JavaScript
~~~js
function simpleRange(n) { return Array.from(Array(n), (_, i) => i); }
function sum(v) { return v.reduce((prev, curr) => prev + curr, 0); }
function euler1(max = 1000) {
  return sum(range(max).filter(x => x % 3 === 0 || x % 5 === 0));
}
var t = tape("Problem 1", test => {
  test.equal(euler1(10), 23);
  test.end();
});
t.run()
console.log(`Problem 1: ${euler1(1000)}`);
~~~

### Matlab/Octave
Matlab doesn’t allow you to define functions inside a single file, but Octave does, bless its cottonsocks!
~~~matlab
% Octave ✅. Matlab you’ll have to save the function in a file, `euler1.m`.
function solution = euler1(Nmax)
  n = 1 : (Nmax - 1); % colon range operator is inclusive
  solution = sum(n(mod(n, 3) == 0 | mod(n, 5) == 0));
end

disp(euler1(1000));
~~~

### Elm
Ok! Now for some fun.

~~~elm
-- to Euler1.elm
import Html exposing (text)
import List

euler1 nmax =
  (List.range 1 (nmax - 1))
    |> List.filter (\n -> n % 3 == 0 || n % 5 == 0)
    |> List.sum

main =
  text (toString (euler1 1000))
~~~

## Problem 2

## Unused code
~~~js
// Via http://stackoverflow.com/a/39930823/500207
function range(start, stop, step = 1) {
  if (typeof start === 'undefined') {
    throw new TypeError('At least one argument required');
  }
  if (typeof stop === 'undefined') {
    stop = start;
    start = 0;
  }
  if ((start > stop && step > 0) || (start < stop && step < 0) || step === 0) {
    return [];
  }
  return Array.from(new Array(Math.ceil((stop - start) / step)),
                    (_, i) => start + i * step);
}
~~~

~~~js
var t = tape("improper steps yield empty matrix", test => {
  test.equal(range(-10).length, 0);
  test.equal(range(0, -10).length, 0);
  test.equal(range(0, 5, -1).length, 0);
  test.end();
});
t.run();

var t = tape("no arguments throws", test => {
  test.throws(() => range(), /one argument required/);
  test.end();
});
t.run();

var t = tape("regular operation", test => {
  test.deepEqual(range(4), [0, 1, 2, 3]);
  test.deepEqual(range(-2, 2), [-2, -1, 0, 1]);
  test.deepEqual(range(-2, 2, 2), [-2, 0]);
  test.deepEqual(range(2, -2, -2), [2, 0]);
  // Floats are imperfect but should work barring pathological cases
  test.deepEqual(range(-1, 1, 0.5), [-1, -.5, 0, .5]);
  test.deepEqual(range(1, -1, -0.5), [1, .5, 0, -.5]);
  // A huge step should just return the initial
  test.deepEqual(range(0, 5, 100), [0]);
  test.end();
});
t.run();
~~~

asd
