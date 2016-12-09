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

### Octave/Matlab
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
> Each new term in the Fibonacci sequence is generated by adding the previous two terms. By starting with 1 and 2, the first 10 terms will be:
>
> > 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, ...
>
> By considering the terms in the Fibonacci sequence whose values do not exceed four million, find the sum of the even-valued terms.

### Elm
~~~elm
import Html exposing (text)
import List
import Arithmetic

fibsAppendNext listSoFar =
  List.sum (List.take 2 listSoFar) :: listSoFar


fibsTilWorker max listSoFar =
  if (Maybe.withDefault 0 (List.head listSoFar)) > max
    then List.drop 1 listSoFar
    else fibsTilWorker max (fibsAppendNext listSoFar)


fibsTil max =
  fibsTilWorker max [2, 1]


euler2 n =
  fibsTil n
    |> List.filter Arithmetic.isEven
    |> List.sum


main =
  text (toString euler2 4000000)
~~~
### JS
Having too much fun with Elm to stop to make other implementations.

## Problem 3
> The prime factors of 13195 are 5, 7, 13 and 29.
>
> What is the largest prime factor of the number 600851475143 ?

### Octave/Matlab
~~~matlab
disp(max(factor(600851475143)))
~~~

### Elm
Alas, the `Arithemetic` library’s prime factorizer fails for numbers bigger than the largest 32-bit number, so this won’t work:
~~~elm
import Html exposing (text)
import List
import Arithmetic

euler3 n =
  Arithmetic.primeFactors n
    |> List.maximum

main =
  text (toString (euler3 600851475143))
~~~
But I don’t really want to implement my own primal sieve either:
~~~elm
maxPrimeFactorWorker target sofar =
  if floatMod0 target sofar
    then sofar
    else maxPrimeFactorWorker target (sofar - 1)


maxPrimeFactor n =
  maxPrimeFactorWorker n (floorFloat (n / 2))


floorFloat n =
  toFloat (floor n)


floatMod0 num den =
  (num / den) == (toFloat (floor (num / den)))
~~~
With this, `maxPrimeFactor 123456789` works, but must be very slow because the original problem is too big.

I have to contort myself:

- I have to manually control for integer vs floating-point. This could be hugely burdensome.
- `floor : Float -> Int` and there’s no to-float version. There should be.
- Mod and rem operate only on integers. 😳🙄😖.

## Problem 4
> A palindromic number reads the same both ways. The largest palindrome made from the product of two 2-digit numbers is 9009 = 91 × 99.
>
> Find the largest palindrome made from the product of two 3-digit numbers.

### Octave/Matlab
~~~matlab
isPalindrome = @(s) strcmp(s, fliplr(s))

mat = [100 : 999]' * [100 : 999];
vec = unique(mat(:));

for i = fliplr(vec')
  if isPalindrome(sprintf('%d', i))
    result = i;
    break;
  end
end
~~~
Again here I miss a “reduce from right but end early” function like Clojure has. Well, recursion.

Oops—only when I looked at the (prime) `factor`s of the `result` did I realize that the question asked for palindrome products of two *three-digit* numbers. Luckily, the answer is the same. (I did a double-take because only one *prime* factor has three digits, but the product of two smaller ones yielded another tridigiter.)

### JavaScript
~~~js
/**
 * Number -> Number.
 * @param  Number x
 * @return Number   the largest power-of-ten smaller than x. 0 if x is negative
 */
function floorBase10(x) {
  // 2**52 ≈ 10**15 is biggest integer that has no gaps in double-precision, but
  // doubles can go to >=1e308.
  if (x <= 0) {
    return 0;
  }
  let floor = 1;
  for (let i = 1; i < 308; i++) {
    if (floor * 10 <= x) { // this > vs < caused me ><
      floor *= 10;
    } else {
      break;
    }
  }
  return floor;
}
/**
 * Number -> Boolean. Checks if the positive integer part of the input is a
 * palindrome with just arithmetic (not string conversions).
 *
 * Caveat: JavaScript numbers are (as of late-2016) 64-bit double-precision
 * floating point numbers, and not all 64-bit integers can be represented as
 * doubles. Specifically, integers bigger than `Math.pow(2, 52) - 1` are at risk
 * of being non-existent as doubles. So if you type in 39393939393939393, this
 * function will fail because the closest double to that is 39393939393939392.
 *
 * Implementation notes:
 *
 * With just floating point arithmetic (`+ - * /`, and `floor`), you can strip
 * the first and last digits off a number, e.g., 12345 -> 234. This function
 * iteratively does this, while checking if the first and last digits at each
 * iteration are equal. At each iteration, you need to keep track of the largest
 * power-of-ten less than the number, but with that as `floorPow10`, this is the
 * math you need:
 * - first digit of x = floor(x / floorPow10)
 * - last digit of x = x % 10
 * - x stripped of its first digit = x - floor(x / floorPow10) * floorPow10
 * - x stripped of its last digit = floor(x / 10)
 *
 * @param  Number  x
 * @param  {Boolean} [info=false] If truthy, printouts for each iteration
 * @return {Boolean}
 */
function isPalindrome(x, info = false) {
  x = Math.floor(Math.abs(x));

  for (let j = floorBase10(x); j >= 1; j /= 100) {
    let firstDigit = Math.floor(x / j);
    let lastDigit = x % 10;
    if (info) {
      console.log('j', j, 'x', x, 'first', firstDigit, 'last', lastDigit,
                  'newx', Math.floor((x - Math.floor(x / j) * j) / 10));
    }
    if (firstDigit !== lastDigit) {
      return false;
    }
    x = x - firstDigit * j; // strip FIRST digit
    x = Math.floor(x / 10); // strip LAST digit
  }
  return true;
}

var t = tape("isPalindrome", test => {
  [0, 1, 5, 101, 12021, 120021, 1200021, -101, -101.5, 101.5].forEach(
      n => test.ok(isPalindrome(n)));
  [39393939393939393, 10, 100, 1000].forEach(n => test.notOk(isPalindrome(n)));
  test.end();
});
t.run();

function euler4(numDigits = 3) {
  const max1 = Math.pow(10, numDigits) - 1;
  let bestPalindrome = 0;
  for (let i = max1; i >= 100; i--) {
    for (let j = max1; j >= 100; j--) {
      const t = i * j;
      if (t > bestPalindrome && isPalindrome(t)) {
        bestPalindrome = t;
      }
    }
  }
  return bestPalindrome;
}
console.log('euler4', euler4());
~~~

### Elm

Well, with that JS, it’s easy:
~~~elm
floorBase10 x =
  let loop x powx =
      if powx * 10 <= x then loop x (10 * powx) else powx
  in
    if x <= 0 then 0 else loop x 1


floorFloat : Float -> Float
floorFloat x = toFloat (floor x)


isPalindrome x =
  let loop x floorPow10 =
    if floorPow10 < 1
      then True
      else
        let
          firstDigit = floorFloat (x / floorPow10)
          lastDigit = toFloat ((round x) % 10)
          newx = floorFloat ((x - firstDigit * floorPow10) / 10)
        in
          if firstDigit == lastDigit
            then loop newx (floorPow10 / 100)
            else False
  in
    loop x (floorBase10 x)
~~~
Other than the float/int issues, this is fine. I need to practice this with another strongly-typed statically-typed language.

### Haskell
Not too hard:
~~~haskell
floorBase10 x =
  let
    loop x powx =
      if powx * 10 <= x then loop x (10 * powx) else powx
  in
    if x <= 0 then 0 else loop x 1

isPalindrome x =
  let
    loop x floorPow10 =
      ((floorPow10 < 1) ||
         (let firstDigit = div x floorPow10
              lastDigit = x `mod` 10
              newx = div (x - firstDigit * floorPow10) 10
            in
            (firstDigit == lastDigit) && loop newx (floorPow10 `div` 100)))
  in
    loop x (floorBase10 x)

main :: IO ()
main =
  let
    n = 12321
  in
    putStrLn (show n ++ " palindrome? " ++ show (isPalindrome n) )
~~~
It has integers, and big integers! And a `div` operator for integer division! That was easy, compared to Elm!

Note: I use [Stack](https://haskell-lang.org/tutorial/stack-script): `$ stack exec -- ghc Pal.hs && ./Pal`.

Using this, I can update the Elm code a bit to use just Integers:
~~~elm
isPalindromeInt x =
  let loop x floorPow10 =
    (floorPow10 < 1) ||
      let
        firstDigit = x // floorPow10
        lastDigit = x % 10
        newx = (x - firstDigit * floorPow10) // 10
      in
        (firstDigit == lastDigit) && loop newx (floorPow10 // 100)
  in
    loop x (floorBase10 x)
~~~
This `Int`-only version is much more natural than the float-only version above. In Haskell, if we get a float, it can readily be converted to an `Integer` without loss of precision (and actually gaining precision, at the cost of runtime). In Elm, I have to choose the tortured `Float` version if I want 32-bit to 52-bit numbers, or use a less spiky `Int` version limited to 32 bits.

## Problem 5
> 2520 is the smallest number that can be divided by each of the numbers from 1 to 10 without any remainder.
>
> What is the smallest positive number that is evenly divisible by all of the numbers from 1 to 20?

That’s really interesting. Check it:

- factor(2) = [2]
- factor(3) = [3]
- factor(4) = [2, 2]
- factor(5) = [5]
- factor(6) = [2, 3]
- factor(7) = [7]
- factor(8) = [2, 2, 2]
- factor(9) = [3, 3]
- factor(10) = [2, 5]
- factor(2520) = [2, 2, 2, 3, 3, 5, 7]

I bet if you took the product of some kind of “cover set” of this set-of-sets (of factors of integers between 2 and 10, inclusive), you’d get 2520. After some poking around, this is indeed the case: each of these lists of prime factors is fully represented in 2520’s prime factors. That is, the solution only needs a single factor of 5 because for each number between 2 and 10, 5 only appears with multiplicity of 1—in each of them. (As to why this works… perhaps I do need to spend some minute–bytes on that: if each number’s list of prime factors is fully represented in the solution’s factors, down to multiplicities, then that number is a factor of the solution. Got it.)

### Octave/Matlab
(I’m using `octave-cli` throughout.) Here’s a clumsy approach—clumsy because it’s kind of tricky to emulate a “multiset” with just a list:

~~~matlab
Nmax = 20
minfactors = [];
for n = 2 : Nmax
  factors = factor(n);
  for f = unique(factors)
    needed = sum(factors == f);
    have = sum(minfactors == f);
    if have ~= needed
      minfactors = [minfactors, f * ones(1, needed - have)];
    end
  end
end
result = prod(minfactors)
~~~

### Elm

[lynn/elm-arithmetic’s `primeExponents`](http://package.elm-lang.org/packages/lynn/elm-arithmetic/2.0.2/Arithmetic#primeExponents) gives just the factors with their multiplicities, instead of just repeating repeated factors. This should help (and is why I’m doing Elm—I also don’t want to find a prime factorizer for Haskell or Rust). (I haven’t written any Rust for this yet.)

~~~elm
import Html
import List
import Dict
import Arithmetic

euler5 nmax =
  let
    loop : Int -> Dict.Dict Int Int -> Dict.Dict Int Int
    loop n accum =
      if n > nmax
        then accum
        else
          let
            factors = Arithmetic.primeExponents n
            accumNew = List.foldl
              (\(factor, mult) accumTemp ->
                Dict.update
                  factor
                  (\maybeCurrMult ->
                    let currMult = Maybe.withDefault 0 maybeCurrMult
                      in Just (if currMult >= mult then currMult else mult))
                  accumTemp)
              accum
              factors
          in
            loop (1 + n) accumNew
  in
    loop 2 Dict.empty
      |> Dict.foldl (\factor mult accum -> accum * factor ^ mult) 1

main = Html.text (toString (euler5 20))
~~~
So—confession time—as I was writing this, it used much less screenspace because I had most of it on one line (I didn’t want to risk `accumNew`’s definition breaking due to whitespace) and because I used `f`, `m`, `c`, and `d` instead of `factor`, `mult`, `currMult`, and `accumTemp`. At that point, I was kind of pleased: it wasn’t any worse than the Octave code above—it wasn’t ‘better’, it was differently good. Then I added whitespace and expressive names, and saw how much nesting there was really going on—as I was writing it, it was (and it remains, even now) very clear what needed to happen next, and so the entire `loop` recursion was very top-down indeed (which is a surprise, because I invariably do bottom-up; in this case, I only tried microscopic versions of pieces in the online Elm playground to make sure I was getting the syntax right). So, once you understand the many levels of looping needed by the algorithm, you can follow your way through the code. But *still*… there is a lot of code. As I was looking over the Octave code, before starting on the Elm code, I remember thinking, “All this ad hoccery, `needed` and `have`, obfuscating what the algorithm does. I bet a functional implementation will be more transparent, and let the algorithm shine through.” As might have been expected, that wasn’t true at all. I suspect this is more opaque. It’s not an Elm thing (the only Elmism is `Maybe`, which serves a critical purpose), it’s an immutable thing.

With all that said, the Elm code is basically doing the same thing as the Octave code above, except it’s using a `Dict`, a hash table mapping factors to multiplicities, instead of a vector of factors with repeats. I still think that data structure should make it clearer what’s going on, so maybe it’s my noob-ness at ML that makes the Elm look hard. (I do suspect that in general immutable forces more code.)

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

~~~js
function lengthNum(x) {
  // 2**52 ≈ 10**15, but doubles can go to >=1e308.
  let numDigits = 1;
  for (let i = 10; i < 1e308; i *= 10) {
    if (x > i) {
      numDigits++;
    } else {
      break;
    }
  }
  return numDigits;
}
~~~

~~~js
// Tricky buggers
function euler4_wrong2(numDigits = 3) {
  const max1 = Math.pow(10, numDigits);
  let i, j;
  for (i = max1; i >= 100; i--) {
    for (j = max1; j >= 100; j--) {
      if (isPalindrome(i * j)) {
        break;
      }
    }
  }
  return i * j;
}
function euler4_wrong(numDigits = 3) {
  const max1 = Math.pow(10, numDigits) - 1;
  const maxProd = max1 * max1;
  let i;
  for (i = maxProd; i > 0; i--) {
    if (isPalindrome(i)) {
      break
    };
  }
  return i;
}
~~~
