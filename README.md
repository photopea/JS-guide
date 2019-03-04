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
- never define a function inside another function

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

The flat representation needs less memory and is easier to collect by GC (as there are not as many "pointers").
Such flat list of objects is processed faster (as there are usually less jumps in memory).
