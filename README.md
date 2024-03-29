# Javascript Guide

Photopea is a set of libraries and other tools, written in Javascript. This guide shows our style of writing code, 
and mentions various related web technologies. It can be useful for writing any high-performance Javascript programs.

We expect, that you already understand programming and at least one programming language.
It is necessary to know the syntax of Javascript. You can learn it e.g. at [w3schools.com/js](https://www.w3schools.com/js/).

## Code style
We try to avoid syntactic sugar. We always follow these rules when writing code:

- use `var` instead of combining `var` / `let` / `const`
- use `"hi"` instead of `'hi'` (quotation marks for strings)
- use `==` instead of `===` (compare only values with the same data type) 
- use `null` instead of `undefined`
- use `obj["prop"]==null` to check, if an object has a specific property
- omit curly brackets if possible: `if(x) run();`  instead of `if(x) { run(); }`

## Static functions

Many of our code consists only of "static functions". These functions get all necessary data on the input. 
There is no `this`, `new`, `prototype` in such code. Functions are joined together by being defined as properties of one global object.
All our public libraries (UPNG.js, UTIF.js, Typr.js ...) have such form.

```js
var Point2D = {};  
Point2D.length = function(p) {  return Math.sqrt(p.x*p.x + p.y*p.y);  }
var Point3D = {};  
Point3D.length = function(p) {  var len = Point2D.length(p);  return Math.sqrt(len*len + p.z*p.z);  }
```
```js
var A = {x:1,y:1}, B = {x:1,y:1,z:1};  
console.log(Point2D.length(A), Point3D.length(B));
```

Such code is easy to rewrite into C (omit the global object `var ABC = {};`, change function names from `ABC.xyz` to `ABC_xyz`). 
Objects can be `struct`s. They never have methods.

## Prototypes

You should be familiar with prototype-based programming, at least to a degree to understand the following code:

```js
function Point2D(x, y) {
  this.x = x;
  this.y = y;
}
Point2D.prototype.length = function() {
  return Math.sqrt(this.x*this.x + this.y*this.y);
}
```

```js
function Point3D(x, y, z) {
  Point2D.call(this, x, y);
  this.z = z;
}
Point3D.prototype = new Point2D();
Point3D.prototype.length = function() {
  var len = Point2D.prototype.length.call(this);
  return Math.sqrt(len*len + this.z*this.z);
}
```
```js
var A = new Point2D(1,1), B = new Point3D(1,1,1);  
console.log(A.length(), B.length());
```

## Anonymous Functions

Let's say we want to have an array A of all primes between 0 and 1000. We can use a following code.

```js
var A = [];
for(var i=2; i<1000; i++) {
    var prime=true;  for(var j=2; j<i-1; j++) if((i%j)==0) prime=false;
    if(prime) A.push(i);
}
```
When the code finishes, the the environment is "polluted" with variables A, i, j, prime. We can improve it.

```js
var getPrimes = function() {
  var pr = [];
  for(var i=2; i<1000; i++) {
      var prime=true;  for(var j=2; j<i-1; j++) if((i%j)==0) prime=false;
      if(prime) pr.push(i);
  }
  return pr;
}
var A = getPrimes();
```
But still, the environment is polluted with variables A and getPrimes. It can be done better:

```js
var A = function() {
  var pr = [];
  for(var i=2; i<1000; i++) {
      var prime=true;  for(var j=2; j<i-1; j++) if((i%j)==0) prime=false;
      if(prime) pr.push(i);
  }
  return pr;
} ();
```
The function is called right after being defined. It does not have a name, so it can not be called from any other part of code. Only a variable A (array of primes) is defined in the environment after the execution.

## Typed Arrays

You should be familiar with [Typed Arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Typed_arrays), 
as they are heavily used in Photopea to store large data.


## Garbage-sensitive programming

In our programs, we may have objects with the similar structure (e.g. 2D points, matrices, buttons, windows ...).

When there are dozens or hundreds of such objects, we can represent them as JS objects / arrays:

```js
var points = [];  for(var i=0; i<30; i++) points.push( {x:i, y:3+2*i} );
function print(k) {  console.log(points[k].x, points[k].y);  }
```

```js
var points = [];  for(var i=0; i<30; i++) points.push( [i, 3+2*i] );
function print(k) {  console.log(points[k][0], points[k][1]);  }
```

When there are thousands or millions of such objects, we represent them in a "flat" array (typed array)

```js
var points = [];  for(var i=0; i<30; i++) points.push( i, 3+2*i );
function print(k) {  console.log(points[2*k], points[2*k+1]);  }
```
```js
var points = new Float32Array(60);  for(var i=0; i<30; i++) {  points[2*i]=i;  points[2*i+1]=3+2*i;  }
function print(k) {  console.log(points[2*k], points[2*k+1]);  }
```

The flat representation needs less memory and is easier to collect by GC (as there is only one JS object).
Such flat list of objects is processed faster (as there are usually less jumps in memory).

## Local HTTP server

To test your code locally, and let it load local files over HTTP protocol, you need a local HTTP server. Using it is easier than it sounds. Here is how to do it on Windows.

Download NGIX from `http://nginx.org/en/download.html` . Unzip it, open the `conf/nginx.conf` and replace its content with following:

```
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       8887;
        server_name  http://127.0.0.1;
        location / {
            root   C:/Desktop/.....;
        }
    }
}
```
- replace the `C:/Desktop/.....` with the directory, that should be accessible from a browser
- double-click the `nginx.exe` to start it. It will run in a background, using 1 MB of RAM 
- you can view `C:/Desktop/XYZ/file.html` by navigating to `http://127.0.0.1:8887/file.html` in any browser


