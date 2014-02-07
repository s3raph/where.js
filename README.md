where.js
========

library-agnostic `where()` clause for driving JavaScript tests with data tables, 
similar to Cucumber 
[`scenario-outlines`](https://github.com/cucumber/cucumber/wiki/Scenario-Outlines) 
or Spock 
[`where:` blocks](https://code.google.com/p/spock/wiki/SpockBasics#Where_Blocks). 

also inspired by:
+ JP Castro's (@jphpsf)
    [DRYing Up Your JavaScript Jasmine Tests With the Data Provider Pattern]
    (http://blog.jphpsf.com/2012/08/30/drying-up-your-javascript-jasmine-tests)
+ Richard Rodger's (@rjrodger) [mstring](https://github.com/rjrodger/mstring)

__[30 JAN - 5 FEB 2014 ~ IN PROGRESS ~ ACTUAL DOCUMENTATION]__

# [TODO](#TODO)...

# works on my machine

[![Build Status](https://travis-ci.org/dfkaye/where.js.png?branch=master)](https://travis-ci.org/dfkaye/where.js)

+ [jasmine](http://jasmine.github.io/) (v2.0.0 on browser)
  - [jasmine-node](https://github.com/mhevery/jasmine-node) which uses v1.3.1
+ [mocha](http://visionmedia.github.io/mocha/)
  - assert (on node.js) and [assert.js](https://github.com/Jxck/assert) on browser
  - [expect.js](https://github.com/LearnBoost/expect.js)
  - [should.js](https://github.com/visionmedia/should.js)
  + [chai](http://chaijs.com/) (assert, expect, should)
+ [QUnit](http://qunitjs.com/)
+ [tape](https://github.com/substack/tape) @substack's event-driven TDD-flavored 
  TAP project for [testling](http://ci.testling.com/)

+ [testling __TODO__]

# [tests](#tests)...

# documentation starts now...

# install

    npm install where.js
    
    git clone https://github.com/dfkaye/where.js.git
  
# important: global assignment

Running `where.js` adds a `where()` method to the **global** namespace:

    require('where.js');
    assert(typeof where === 'function');
    // => true
    assert(typeof global.where === 'function');
    // => true

    <script src="../where.js"></script>

    ...
    assert(typeof where === 'function');
    // => true
    assert(typeof window.where === 'function');
    // => true
    
# justify

Easier to read and modify this
    
    it('description', function () {
      where(function(){
        /*** 
          a  |  b  |  c
          1  |  2  |  2
          4  |  3  |  4
          6  |  6  |  6
        ***/
        
        expect(a + b)).toBe(c);
      });
    });

than this

    it('description', function () {
      [[1, 2, 2],
       [4, 3, 4],
       [6, 6, 6]].forEach(function(row, r, rows) {
               
        expect(Number(row[0]) + Number(row[1])).toBe(Number(row[2]));
      });
    });
    
# story

where.js merges both basic algorithms and lessons learned from my earlier 
[jasmine-where](https://github.com/dfkaye/jasmine-where) 
and [jasmine-intercept](https://github.com/dfkaye/jasmine-intercept) projects, 
which are now __deprecated__ as of this release.

In imitation of Richard Rodger's [mstring](https://github.com/rjrodger/mstring), 
`where()` accepts a function and inspects its string value, converts the 
commented data-table into an array of values, and applies the labels as argument 
names in a new Function().

`where()` accepts a second [`context`](#context) argument that allows you to 
inject other information or references into the test function.

(I'll relate some additional lessons learned from this project down below or 
elsewhere.)

# format

Similar to Cucumber and Spock, where.js data tables must contain at least two 
rows. The first row must contain the names to be used as variables in the 
expectation. The remaining rows must contain data values for each name.  Values 
must be separated by the pipe | character.

For example, `a`, `b`, and `c` are named as variables in the table, and used in 
the expectation - without having to be defined or var'd:

    it('should pass with correct data and expectation', function () {
      where(function(){/***
         a | b | c
         1 | 2 | 2
         4 | 3 | 4
         6 | 6 | 6
        ***/
        expect(Math.max(a, b)).toBe(c);
      });
    });

# borders

Tables may also optionally contain left and right edge borders, similar to 
Cucumber and Fit:

    it('should pass with left and right table borders', function () {
      where(function(){/***
        | a | b | c |
        | 1 | 2 | 2 |
        | 4 | 3 | 4 |
        | 6 | 6 | 6 |
        ***/
        expect(Math.max(a, b)).toBe(c);
      });
    });

# line comments

Table rows may contain line comments.

      where(function(){/***
        | a | b | c |
        | 1 | 2 | 2 |  // should pass
        | 4 | 3 | x |  // should fail
        ***/
        expect(Math.max(a, b)).toBe(c);
      });
      
A commented row is ignored.

      where(function(){
        /***
          | a | b | c |
          | 1 | 2 | 2 | // should pass
        //| 4 | 3 | x | (should not execute)
        ***/
        expect(Math.max(a, b)).toBe(c);
      });

numeric data
------------

__Numeric data is automatically converted to the `Number` type.__

That enables you to type `Math.max(a, b)` to avoid re-typing coercions such as 
`Math.max(Number(a), Number(b))`.

That also means every test can use strict equality, so you don't need to rely 
on matchers.

_However_, where `Math` is involved there is usually an octal, signed, comma, or 
precision bug waiting.  where.js handles all but precision automatically.  You 
can get precision into your tests by adding another column, as seen in the test 
created to verify that numeric conversions work:

    where(function(){
      /***
            a     |    b     |    c     |  p
            
            0     |    1     |    1     |  1
            0.0   |    1.0   |    1     |  1
           -1     |   +1     |    0     |  1
           +1.1   |   -1.2   |   -0.1   |  2
           08     |   08     |   16     |  2
            6     |    4     |   10.0   |  3
            8.030 |   -2.045 |    5.985 |  4
        1,000.67  | 1345     | 2345.67  |  6
        
      ***/
      
      /* 
       * using precisions for famous .1 + .2 == 0.30000000000000004 
       * and 5.985 vs 5.98499999999999999999999999 bugs 
       */
      var s = (a + b).toPrecision(p) // toPrecision() returns a string
      expect(+s).toBe(c) // but prefixed '+' uses implicit conversion to number.
    });

# null, undefined, true, false values

Truthy/falsy values are also automatically converted as per this test:

      where(function () {
        /***
          a    | b         | c     | d
          null | undefined | false | true
        ***/
        
        expect(a).toBe(null);
        expect(b).toBe(undefined);
        expect(c).toBe(false);
        expect(d).toBe(true);        

      });

# quoted values

Data with quoted strings are preserved.

      where(function () {
        /***
          a  | b  | c    | d    | e   | f        | g
          '' | "" | "''" | '""' | ' ' | ' faff ' | 'undefined'
        ***/
        expect(a).toBe('\'\'');
        expect(b).toBe('\"\"');
        expect(c).toBe('\"\'\'\"');
        expect(d).toBe('\'"\"\'');
        expect(e).toBe('\' \'');
        expect(f).toBe('\' faff \'');
        expect(f).toBe('\'undefined\'');
      });
      
# output

A passing `where()` test has no effect on the test runner's default reporter 
output.

When an expectation fails, the data-table labels plus the row of values for the 
current expectation are added to the *current* failing item. Every failed 
expectation in a `where()` test will appear similar to:

     [a | b | c] : 
     [1 | 2 | x] (Error: Expected 2 to be 'x'.)

# results

`where()` returns a `results` object with three arrays for all `values` derived 
from the data table, `passing` tests, and `failing` tests.  The first row in the 
`values` array (`results.values[0]`) is the row of names.  

The following snip shows how to refer to each array in the results:

    it('should pass with correct data and expectation', function () {
      var results = where(function(){/***
         a | b | c
         1 | 2 | 2
         4 | 3 | 4
         6 | 6 | 6
        ***/
        expect(Math.max(a, b)).toBe(c);
      });
      
      expect(results.values.length).toBe(4);
      expect(results.passing.length).toBe(3);
      expect(results.failing.length).toBe(0);
    });

# context specifier

__THIS IS THE CRITICAL "LIBRARY-AGNOSTIC" PIECE OF THE PUZZLE.__

`where` accepts up to two arguments. The first is the function containing the 
data table and assertions.  

The second, optional but _recommended_, argument is a `context` or configuration 
object that allows you to specify the test strategy (or library) in use, and 
pass non-global items to the test function (a library's assert or expect or test 
method, e.g.).

The following snip shows that `context` is itself made available for inspection 
inside the test function:

    where(function(){/***
         a | b | c
         0 | 0 | 0
        ***/
        
        expect(context.expect).toBe(expect);
        
    }, { expect: expect});

You may also use it to enable logging all test output to the console, enable 
interception of failing tests (to try preventing them appearing as failed even 
if they are expected to fail - in other words, make a fail into a pass), and/or 
add information or references to be used in the test function.

The following snip shows a test with logging and interception, specifies the 
test library as jasmine, and passes the expect function:

    it('should pass with correct data and expectation', function () {
    
      var results = where(function(){/***
         a | b | c
         1 | 2 | 2
         4 | 3 | 4
         6 | 6 | 6
        ***/
        
        expect(Math.max(a, b)).toBe(c);
        
      }, { log: 1, intercept: 1, strategy: 'jasmine', expect: expect });
      
      expect(results.values.length).toBe(4);
      expect(results.passing.length).toBe(3);
      expect(results.failing.length).toBe(0);
    });

WARNING: In event-based test libraries like QUnit and tape, pure interception is 
not always successful - an assertion that fails is always reported as failed.

# strategy

where.js comes with four strategies pre-defined for each testing library 
supported initially.  A strategy can be defined in one of three ways:

+ `{ strategy: '<quoted-name>' }`
+ `{ <name>: 1 }`
+ `{ <name>: <name-object-reference> }`

## mocha (default)

The default strategy is a basic try+catch used by mocha. *You do not need to 
specify a strategy when using mocha.* 

+ `{ strategy: 'mocha' }`
+ `{ mocha: 1 }`
+ `{ mocha: mocha }` (when mocha is defined elsewhere in your tests and you wish  
    to use it within the test function itself)

However, unless you are using should.js, you must specify which assertion method 
your test relies on:

+ `{ expect: expect }`
+ `{ assert: assert }`
+ `{ expect: chai.expect }`
+ `{ assert: chai.assert }`

*(Both mocha's should.js and chai's should.js add a `should` method to 
`Object.prototype`, brilliantly making every object assertable - except those, 
not surprisingly, created by the `Object.create()` method.)*

*The mocha `assert` browser tests in this repo rely on 
[assert.js](http://github.com/Jxck/assert), "a port of the Node.js standard 
assertion library for the browser."*

## jasmine

Because jasmine also uses a try+catch approach, you do not need to specify 
jasmine as the test strategy *unless you want to intercept 
failing tests.*  Specify the jasmine strategy with one of the following:

+ `{ strategy: 'jasmine' }`
+ `{ jasmine: 1 }`
+ `{ jasmine: jasmine }` (jasmine is defined globally in both node and browsers)

## QUnit

When using QUnit, you __must__ specify the QUnit strategy as follows:

+ `{ QUnit: QUnit }` (QUnit is defined globally in both node and browsers)

*The QUnit tests in this repo rely on 
[qunit-tap](//https://github.com/twada/qunit-tap), "A TAP Output Producer Plugin 
for QUnit."*

## tape

For use with tape, you __must__ specify the tape strategy as follows:

+ `tape: [test | t]` 

This is due to James' (@substack) approach of passing the tape `test` reference 
into the callback as `t` and ingeniously re-using that as both a results cache 
and as an emitter for 'result' events.

    var test = require('tape');

    test('should pass tape context', function(t) { // t is test, test is tape...
          
      var results = where(function(){/***
        | a | b | c |
        | 0 | 0 | 0 |
        ***/
        
        // use the context reference to 'tape' to test itself
        tape.equal(tape, context.tape, 'should find tape');

      }, { tape: t });
      
      t.equal(results.passing.length, 1);
      t.end();
    });

*A copy of my [dom-console](https://github.com/dfkaye/dom-console) library is 
included in the browser suite for tape, and can be found in the 
[test/util](/test/util) folder. The dom-console merely writes `console.log()` 
statements to a list in the DOM.*


custom strategy
---------------

__TODO__


more sophisticated example
--------------------------

__TODO__ [ object instance creation example]


coffeescript
--------------------------

__TODO__ [ multiline string delimiter issue ]


testling with mocha
-------------------

__TODO__


tests
-----

The goal of making where.js run in "any" test framework required making numerous 
versions of test suites.  Here's how they stack up:

+ where.js core functionality tests using jasmine-node
  - `npm test`
  - `node jasmine-node --verbose ./test/where/where.spec.js`

+ framework strategy comparison tests on node.js
  - `npm run jasmine`
  - `npm run mocha`
  - `npm run qunit`
  - `npm run tape`

testem
------

I'm using Toby Ho's MAGNIFICENT [testem](https://github.com/airportyh/testem) 
to drive tests in node.js and in multiple browsers.

The core tests use 
[jasmine-2.0.0](http://jasmine.github.io/2.0/introduction.html) (which requires 
testem v0.6.3 or later) in browsers, and Misko Hevery's
[jasmine-node](https://github.com/mhevery/jasmine-node) (which uses jasmine 
1.3.1 internally) on node.js.

The `testem.json` file defines launchers for the different frameworks for both 
node.js and browser suites:
  - `testem -l where` # core tests using jasmine
  - `testem -l jasmine` 
  - `testem -l mocha`
  - `testem -l qunit`
  - `testem -l tape` # runs browserify on the tape suite 

npm testem scripts
------------------

The `package.json` file defines scripts to call the testem launchers (with the 
appropriate browser test page for each):
  - `npm run testem`
  - `npm run testem-jasmine`
  - `npm run testem-mocha`
  - `npm run testem-qunit`
  - `npm run testem-tape`

browser suites
--------------

The `browser-suites` rely on browser versions of each library stored in the 
/vendor directory (mocha, expect.js, assert.js, should, chai, qunit, qunit-tap, 
and jasmine-2.0.0), rather than the /testem directory, so they can be viewed as 
standalone pages with no dependency on testem.  

You can view them directly on rawgithub:
  - [core suite](https://rawgithub.com/dfkaye/where.js/master/test/where/browser-suite.html)
  - [jasmine](https://rawgithub.com/dfkaye/where.js/master/test/jasmine/browser-suite.html)
  - [mocha et al.](https://rawgithub.com/dfkaye/where.js/master/test/mocha/browser-suite.html)
  - [QUnit with qunit-tap](https://rawgithub.com/dfkaye/where.js/master/test/qunit/browser-suite.html)
  - [tape with browserified source](https://rawgithub.com/dfkaye/where.js/master/test/tape/browser-suite.html)

  
# TODO
+ add coffeescript support with a mocha or tape example (resolve /*** vs /*!)
+ triple star comments `/***` not (easily) supported in Coffeescript - could 
    convert to `/*`
+ need more sophisticated object-creation examples
+ add testling config for tape suite
+ try testling with another test framework?
+ README documentation
  + reorganize docs - too sprawling or verbose
  + strategy 'purpose' needs explaining (try+catch vs events vs ?)
  + strategy ui needs re-visiting - strings vs objects
    - jasmine - assume global or double as 'context.jasmine'
    - QUnit - assume global or double as 'context.QUnit'
    - tape - t function serves double as 'context.tape'
    - custom strategy
+ version bump
+ npm publish
+ post

  
# License

MIT for now, JSON eventually