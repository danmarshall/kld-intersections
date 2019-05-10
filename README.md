# kld-intersections

- [Installation](#installation)
- [Importing](#importing)
- [Usage](#usage)
    - [Intersection API](#intersection-api)
    - [Shapes API](#shapes-api)
    - [Affine Shapes API](#affine-shapes-api)
    - [SVG Shapes API](#svg-shapes-api)
    - [Use Your Own Objects](#use-your-own-objects)
- [Queries](#queries)
- [Known Issues](#known-issues)
- [Links and Related Projects](#links-and-related-projects)

---

A library of intersection algorithms covering all permutations for any two of the following SVG shapes:

- Quadratic Bézier
- Cubic Bézier
- Circle
- Ellipse
- Arcs
- Line
- Polygon
- Polyline
- Rectangle
- Paths

# Installation

```
npm install kld-intersections
```

# Importing

The following sections indicate how you can import the code for use in various environments.

## Node

```javascript
import {AffineShapes, Shapes, Intersection} = require("kld-intersections");
```

## Browsers

```html
<script src="./node_modules/kld-intersections/dist/index-umd.js"></script>
<script>
  var AffineShapes = KldIntersections.AffineShapes;
  var Shapes = KldIntersections.Shapes;
  var Intersection = KldIntersections.Intersection;
</script>
```

## Modern Browsers (ESM)

```javascript
import {AffineShapes, Shapes, Intersection} from './node_modules/kld-intersections/dist/index-esm.js';
```

## Bundlers

```javascript
import {AffineShapes, Shapes, Intersection} from "kld-intersections";
```

# Usage

kld-intersection allows you to find intersections using two general approaches:

- By calling the intersection methods directly
- By creating simple descriptors of shapes and then letting the library determine which method to invoke

## Intersection API

At the lowest level, you can invoke intersection methods directly in the `Intersection` class. In order to determine which intersection routine to invoke, you will need to determine two bits of information for each curve involved in the intersection:

1. The name the library uses to refer to the given curve type
2. The arguments used to represent the curve parameters to the library

Below is a table listing each of the supported curve types. The `Name` column is the name you will need to use to refer to the shape of the given type. `Arguments` lists the parameters you will use to describe the shape of the given curve.

| Shape            | Name      | Arguments                                                                    |
| ---              | ---       | ---                                                                          |
| Quadratic Bézier | Bezier2   | **c1** : *Point2D*, **c2** : *Point2D*, **c3**: *Point2D*                    |
| Cubic Bézier     | Bezier3   | **c1** : *Point2D*, **c2** : *Point2D*, **c3**: *Point2D*, **c4**: *Point2D* |
| Circle           | Circle    | **center** : *Point2D*, **radius** : Number                                  |
| Ellipse          | Ellipse   | **center** : *Point2D*, **radiusX** : Number, **radiusY** : Number           |
| Arc              | Arc       | **center** : *Point2D*, **radiusX** : Number, **radiusY** : Number, **startRadians** : Number, **endRadians** : Number |
| Line             | Line      | **p1** : *Point2D*, **p2** : *Point2D*                                       |
| Polygon          | Polygon   | **points** : *Array< Point2D >*                                              |
| Polyline         | Polyline  | **points** : *Array< Point2D >*                                              |
| Rectangle        | Rectangle | **topLeft** : *Point2D*, **bottomRight** : *Point2D*                         |
| Path             | Path      | **segments** : *Array< IntersectionArgs >*                                   |

Once you've determined the names and arguments, you can build the intersection method name like so:

1. All intersections begin with `intersect`
2. Append the `Name` of the first curve type
3. Append the `Name` of the second curve type

It is important to note that not all combinations of names are available in the API. The current implementation supports 10 types of curves. If we count all combinations of any two curves, we end up needing `10 * 10 = 100` methods to cover all cases. Fortunately, the order in which we specify the curves is not important.

If we intersect `a rectangle with a circle` or `a circle with a rectangle`, we will get the same results, regardless of their order. Recognizing this allows us to reduce the number of methods to `10 + 9 + 8 + 7 + 6 + 5 + 4 + 3 + 2 + 1 = 54`. Due to this reduction, one more restriction applies when determining the intersection method name:

> Shape names must be appended to the method name in alphabetical order

In our earlier example of intersecting a `Rectangle` and a `Circle`, we will need to append `Circle` before `Rectangle` giving us `intersectCircleRectangle` instead of `intersectRectangleCircle`.

Now that you have determined the method name, you need to pass in arguments to describe the shape of each curve. You do that as below:

1. Review the `Arguments` list in the table above
2. Add all arguments for the first curve in the specified order to the method call
3. Add all arguments for the second curve in the specified order to the method call

### Example

Lets intersect a `circle` and a `rectangle`. From the table above, we see that the circle's name is `Circle`, and the name of the rectangle is `Rectangle`. Following the rules stated above, this means our method name is:

`intersectCircleRectangle`

A circle is described with a center point and a radius. Those are our first two arguments.

A rectangle is described with two points: the top-left corner and the bottom-right corner. Those are our final arguments.

Putting this all together, we end up with something like the following:

```javascript
const {Point2D, Intersection} = require("kld-intersections");

const circle = {
    center: new Point2D(0, 0),
    radius: 50
};

const rectangle = {
    topLeft: new Point2D(0, 0),
    bottomRight: new Point2D(60, 30)
};

const result = Intersection.intersectCircleRectangle(
    circle.center, circle.radius,
    rectangle.topLeft, rectangle.bottomRight
);

console.log(result);
```

Note that the `circle` and `rectangle` variables were used to make the code more readable. You could easily remove those objects, passing in their arguments directly. That would make a minimal example like the following:

```javascript
const {Point2D, Intersection} = require("kld-intersections");

const result = Intersection.intersectCircleRectangle(
    new Point2D(0, 0), 50,
    new Point2D(0, 0), new Point2D(60, 30)
);

console.log(result);
```

### Result

```
Intersection {
  status: 'Intersection',
  points:
   [ Point2D { x: 50, y: 0 },
     Point2D { x: 40.00000000000001, y: 30.000000000000004 } ] }
```

Note that `x` and `y` in the second point are not exactly 40 and 30 due to floating point imprecision.

### Visual Representation

If we were to plot the results, we would end up with an image like the following.

![Example image 1](./images/example-1.png)

## Shapes API

The Shapes API allows you to create descriptions of shapes and curves which the intersection library can then use to determine which intersection method to invoke. This API allows you to create the following shapes:

- quadraticBezier
- cubicBezier
- circle
- ellipse
- arc
- line
- polygon
- polyline
- rectangle
- path

To create these shapes, invoke the method name from the list above, passing in the arguments as described in the table in [Intersection API](#intersection-api). However, when passing in, for example, a `Point2D`, you will need to pass in the `x` and `y` values separately. This allows your code to utilize the intersection library without having to commit to the kld-affine classes.

To find the intersections of these shapes, invoke `Intersection.intersect` passing in your two shape descriptors, in any order.

### Example

```javascript
const {Point2D, Intersection, Shapes} = require("kld-intersections");

const circle    = Shapes.circle(0, 0, 50);
const rectangle = Shapes.rectangle(0, 0, 60, 30);
const result    = Intersection.intersect(circle, rectangle);

console.log(result);
```

### Result

```
Intersection {
  status: 'Intersection',
  points:
   [ Point2D { x: 50, y: 0 },
     Point2D { x: 40.00000000000001, y: 30.000000000000004 } ] }
```

### Visual Representation

![Example image 2](./images/example-2.png)


### Working with Path Data

Building paths from the other atomic shapes can be tedious and error-prone. In order to simplify this process, the Shapes API includes a `pathData` method. Simply pass in a string of path data and you will get a path shape with all of its segments populated for you.

```javascript
import {Shapes} from "kld-intersections";

// create a path from path data
const path = Shapes.pathData("M0,0 C100,0 100,100 0,100z m20,0 c100,0 100,100 0,100z");

// create a line
const line = Shapes.line(0, -10, 110, 110);

// intersect
const result = Intersection.intersect(path, line);
```

## Affine Shapes API

The Affine Shapes API is very similar to the Shapes API but it allows you to use a slightly higher level of abstraction by utilizing the kld-affine classes. This API allows you to create the following shapes:

- quadraticBezier
- cubicBezier
- circle
- ellipse
- arc
- line
- polygon
- polyline
- rectangle
- path

To create these shapes, invoke the method name from the list above, passing in the arguments as described in the table in [Intersection API](#intersection-api).

To find the intersections of these shapes, invoke `Intersection.intersect` passing in your two shape descriptors, in any order.

### Example

```javascript
const {Point2D, Vector2D, Intersection, AffineShapes} = require("kld-intersections");

const circle    = AffineShapes.circle(new Point2D(0, 0), 50);
const rectangle = AffineShapes.rectangle(new Point2D(0, 0), new Vector2D(60, 30));
const result    = Intersection.intersect(circle, rectangle);

console.log(result);
```

### Result

```
Intersection {
  status: 'Intersection',
  points:
   [ Point2D { x: 50, y: 0 },
     Point2D { x: 40.00000000000001, y: 30.000000000000004 } ] }
```

### Visual Representation

![Example image 3](./images/example-3.png)

### Working with Path Data

Building paths from the other atomic shapes can be tedious and error-prone. In order to simplify this process, the AffineShapes API includes a `pathData` method. Simply pass in a string of path data and you will get a path shape with all of its segments populated for you.

```javascript
import {AffineShapes} from "kld-intersections";

// create a path from path data
const path = AffineShapes.pathData("M0,0 C100,0 100,100 0,100z m20,0 c100,0 100,100 0,100z");

// create a line
const line = AffineShapes.line(0, -10, 110, 110);

// intersect
const result = Intersection.intersect(path, line);
```

## SVG Shapes API

The SVG Shapes API allows you to create descriptions of shapes and curves using SVG elements from the DOM. With this API, you can create shape descriptions for the following elements:

- circle
- ellipse
- line
- path
- polygon
- polyline
- rect

When converting path elements, the element's path data will be used to generate shape descriptions, one shape description per path segment. All SVG path commands are supported.

In order to simplify code, `SvgShapes` includes an `element` method. Pass in an SVG element and this method will dispatch to the appropriate conversion method. An exception will be thrown for unsupported SVG element types.

The example below shows how you might convert and intersect two arbitrary SVG elements.

### Example

```javascript
const {SvgShapes, Intersection} = require("kld-intersections");

const shape1 = SvgShapes.element(document.getElementById("a"));
const shape2 = SvgShapes.element(document.getElementById("b"));
const result = Intersection.intersect(shape1, shape2);

console.log(result);
```

For more complete examples, have a look at the SVG files in the `visual-test` directory. These cover all combinations of shape intersections.

## Use Your Own Objects

If you have a look at the various API classes, you'll find that they are quite simple. These APIs create instances of the `IntersectionArgs` class. In turn the `IntersectionArgs` class is quite simple too, consisting of two properties: `name` and `args`. `name` is the name of the shape or curve as defined in the table describing the [Intersection API](#intersection-api). Likewise, `args` is an array of the arguments described in the arguments column in that same table. So, you can pass in any object to `Intersection.intersect` as long as it contains those two properties which need to follow the name and argument conventions just described.

# Queries

In the original intersection code written for kevlindev.com, there were some functions that allowed one to determine if a point was contained within a given shape. That code has been extracted into a separate class named `IntersectionQuery`. Those methods are currently limited to the following list:

* IntersectionQuery.pointInCircle(point, center, radius)
* IntersectionQuery.pointInEllipse(point, center, radiusX, radiusY)
* IntersectionQuery.pointInPolyline(point, points)
* IntersectionQuery.pointInPolygon(point, points)
* IntersectionQuery.pointInRectangle(point, topLeft, bottomRight)

The first argument is the `Point2D` you wish to test. The remaining parameters follow the same convention as described in [Intersection API](#intersection-api). All methods return a boolean value indicating whether or not the given point is contained within the shape.

Note that currently bézier curves are not included in this list. As a workaround, bézier shapes can be approximated using polylines and then tested with `pointInPolyline` or `pointInPolygon`. See [tessellate-cubic-beziers.js](examples/tessellate-cubic-beziers.js) as an example of how you might tesselate bézier curves for this purpose.

# Known Issues

Please note that the bézier-bézier intersections may not be well-behaved. There is work being done to make these more stable, but no eta is available at this time. As a workaround, bézier shapes can be approximated using polylines and then intersected with an appropriate polyline intersection method. See [tessellate-cubic-beziers.js](examples/tessellate-cubic-beziers.js) as an example of how you might do this.

# Links and Related Projects

- [http://www.quazistax.com](http://www.quazistax.com/testIntersection.html)
- [kld-path-parser](https://github.com/thelonious/kld-path-parser)
- [kld-transform-parser](https://github.com/thelonious/kld-transform-parser)
- [kld-affine](https://github.com/thelonious/kld-affine)
- [kld-polynomial](https://github.com/thelonious/kld-polynomial)
- [kld-contours](https://github.com/thelonious/kld-contours)
