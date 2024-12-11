# Projective Transformation Estimation

This repository provides a JavaScript implementation for estimating and applying a plane projective (homography) transformation given four source points and their corresponding four destination points. By computing a transformation matrix **H**, you can map any point `(x, y)` in the source plane to its transformed location `(x', y')` in the destination plane.

**Table of Contents**
- [Overview](#overview)
- [Key Concepts](#key-concepts)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [API Reference](#api-reference)
- [Mathematical Background](#mathematical-background)
- [Examples](#examples)
- [Performance Considerations](#performance-considerations)
- [References](#references)
- [License](#license)

---

## Overview

In computer vision, projective transformations (or homographies) are widely used to correct perspective distortions, stitch images, and perform tasks such as camera calibration, augmented reality overlays, and shape/pose estimation. A homography maps straight lines to straight lines and is represented by a 3x3 matrix with 8 degrees of freedom.

This code:
- Takes in four known correspondences between source and destination points.
- Solves the linear equation system `aH = b` to find the homography matrix **H**.
- Provides utility functions to apply the computed transformation to arbitrary points.

---

## Key Concepts

- **Homography Matrix (H)**: A 3x3 matrix that encodes how a point in one plane maps to a point in another plane under a projective transform. It is often written as:

  ```
  H = [ h11 h12 h13
        h21 h22 h23
        h31 h32 h33 ]
  ```

- **Point Mapping**: Given a point `(x, y)` in the source plane, its transformed coordinates `(x', y')` are obtained by:
  
  ```
  x' = (h11*x + h12*y + h13) / (h31*x + h32*y + h33)
  y' = (h21*x + h22*y + h23) / (h31*x + h32*y + h33)
  ```

- **Estimation of H**: To compute **H**, you need at least four non-collinear point correspondences. Each correspondence `(x, y) -> (u, v)` yields two equations. With four correspondences, you have 8 equations to solve for the 8 unknowns of **H** (since `h33` can be normalized to 1).

---

## Prerequisites

- **Sylvester.js**: This library relies on [Sylvester](http://sylvester.jcoglan.com/), a JavaScript vector and matrix math library. Make sure to include it before using these functions.

  ```html
  <script src="path/to/sylvester.js"></script>
  ```

- A modern JavaScript runtime environment (e.g., Node.js or a recent browser).

---

## Installation

This code is standalone and does not require a complex build process. Simply:

1. Download or copy `estimate_H`, `get_xprime`, and `get_yprime` functions into a `.js` file.
2. Include the file in your HTML or load it into your JavaScript environment.
3. Ensure you have Sylvester loaded beforehand.

For example:

```html
<script src="path/to/sylvester.js"></script>
<script src="path/to/projective_transform.js"></script>
```

---

## Usage

1. Gather four pairs of corresponding points. For example:
   ```js
   // Source points: (x1, y1), (x2, y2), (x3, y3), (x4, y4)
   // Destination points: (u1, v1), (u2, v2), (u3, v3), (u4, v4)
   ```
   
2. Compute the transformation matrix **H**:
   ```js
   var H = estimate_H(
     x1, y1, x2, y2, x3, y3, x4, y4,
     u1, v1, u2, v2, u3, v3, u4, v4
   );
   ```
   
3. Use `get_xprime` and `get_yprime` to transform any other point `(x, y)`:
   ```js
   var xPrime = get_xprime(x, y, H);
   var yPrime = get_yprime(x, y, H);
   ```

---

## API Reference

### `estimate_H(...)`

**Signature:**
```js
function estimate_H(
  x1, y1, x2, y2, x3, y3, x4, y4,
  u1, v1, u2, v2, u3, v3, u4, v4
) 
```

**Description:**
Computes the homography matrix **H** that maps each `(x_i, y_i)` to `(u_i, v_i)`.

**Parameters:**
- `x1, y1, x2, y2, x3, y3, x4, y4` (Number): Source coordinates.
- `u1, v1, u2, v2, u3, v3, u4, v4` (Number): Destination coordinates.

**Returns:**
- A Sylvester `Matrix` object of size 8x1 representing `[h11, h12, h13, h21, h22, h23, h31, h32]^T`.

---

### `get_xprime(x, y, H)`

**Signature:**
```js
function get_xprime(x, y, H)
```

**Description:**
Given a point `(x, y)` and the homography parameters **H**, returns the transformed x-coordinate `x'`.

**Parameters:**
- `x, y` (Number): The original point coordinates.
- `H` (Matrix): The transformation matrix returned by `estimate_H`.

**Returns:**
- `x'` (Number): The transformed x-coordinate.

---

### `get_yprime(x, y, H)`

**Signature:**
```js
function get_yprime(x, y, H)
```

**Description:**
Given a point `(x, y)` and the homography parameters **H**, returns the transformed y-coordinate `y'`.

**Parameters:**
- `x, y` (Number): The original point coordinates.
- `H` (Matrix): The transformation matrix returned by `estimate_H`.

**Returns:**
- `y'` (Number): The transformed y-coordinate.

---

## Mathematical Background

A projective transformation (homography) in 2D is defined by 8 independent parameters. Given four known point correspondences, we form a linear system:

```
[x1 y1 1  0  0  0 -x1*u1 -y1*u1]   [h11]   [u1]
[0  0  0  x1 y1 1 -x1*v1 -y1*v1] * [h12] = [v1]
[x2 y2 1  0  0  0 -x2*u2 -y2*u2]   [h13]   [u2]
[0  0  0  x2 y2 1 -x2*v2 -y2*v2]   [h21]   [v2]
[x3 y3 1  0  0  0 -x3*u3 -y3*u3]   [h22]   [u3]
[0  0  0  x3 y3 1 -x3*v3 -y3*v3]   [h23]   [v3]
[x4 y4 1  0  0  0 -x4*u4 -y4*u4]   [h31]   [u4]
[0  0  0  x4 y4 1 -x4*v4 -y4*v4]   [h32]   [v4]
```

Once **H** is found (with `h33` normalized to 1), we can apply it to transform any point `(x, y)` to `(x', y')`.

---

## Examples

### Basic Example

```js
// Source points
var x1 = 0,   y1 = 0;
var x2 = 100, y2 = 0;
var x3 = 100, y3 = 100;
var x4 = 0,   y4 = 100;

// Destination points (e.g., a slight rotation or perspective shift)
var u1 = 10,  v1 = 20;
var u2 = 120, v2 = 30;
var u3 = 110, v3 = 130;
var u4 = 20,  v4 = 120;

// Compute H
var H = estimate_H(
  x1, y1, x2, y2, x3, y3, x4, y4,
  u1, v1, u2, v2, u3, v3, u4, v4
);

// Transform a new point using the computed H
var newX = get_xprime(50, 50, H);
var newY = get_yprime(50, 50, H);

console.log("Transformed point:", newX, newY);
```

---

## Performance Considerations

- Computing a matrix inverse is a relatively expensive operation. However, since the matrix here is fixed-size (8x8 from the linear system), the computation is still quite fast for most use cases.
- Sylvester is implemented in JavaScript, so performance may be lower compared to compiled languages like C++.
- For large-scale or real-time applications (e.g., live video), consider optimizing or using a specialized linear algebra library or GPU-accelerated solutions.

---

## References

- [Leptonica projective transform source](http://tpgit.github.io/Leptonica/projective_8c_source.html)
- [Homography Estimation (CVOnline)](http://homepages.inf.ed.ac.uk/rbf/CVonline/LOCAL_COPIES/EPSRC_SSAZ/epsrc_ssaz.html)
- [Sylvester.js Library](http://sylvester.jcoglan.com)
- Multiple-view Geometry in Computer Vision by Hartley & Zisserman (for theoretical background)

---

## License

This code is released under the [MIT License](LICENSE). Feel free to use, modify, and distribute it as you see fit.
