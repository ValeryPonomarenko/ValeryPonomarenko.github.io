---
layout: post
title:  "How to Create a Half Circle Progress Bar"
date:   2023-06-01 14:50:00 +0300
categories: article
tags: ["flutter", "custom view", "ui"]
poster: "/assets/175IcrkqXF0hdboTG0AJGRg.gif"
excerpt_separator: <!--more-->
---
I bet you all had situations when a designer made a cool-looking UI and you just thought “cool, but how to implement this”. The same happened to me when a saw this progress bar in Figma.
Good thing is that in Flutter it’s pretty easy to create custom views and they will look awesome on every platform, but sometimes you just need to remember all these things that you have learned in geometric, and math classes. Yeah, we don’t always change the button’s color as someone thinks :)
<!--more-->

## A little bit of geometry
Let’s think what we have to make to implement the progress bar. Firstly, we have to find out a way to understand how many dots should be on the half-circle. Secondly, how to place these dots on it.

### Calculating the number of dots
Let’s start simple, what if we have a straight line and we need to calculate how many points it can fit. The answer is that you take the length of the line, divide it by the diameter of the dot and add padding between them. And that’s done.
We do the same, but with a circle. If you know the radius of the circle, we can find its length. The formula is `2 * pi * r`, but we need only half of it, so for us, the formula will be:
```dart
lineLength = pi * r
```
Finally, we know the size (radius or diameter) of the dots and the spacing between them, so we can simply divide the line’s length by the size of the dots with spacing. So we have a formula like this:
```dart
dotsNumber = floor(lineLength / (dotDiameter + dotsSpacing))
```
We use floor because we can’t draw, say, half of the dot.

### Placing dots on the circle
If we had a line, that would be easy to place the dots on it. We would start from 0 and would add `dotDiameter + dotsSpacing` after the dots have been drawn.
Unfortunately, in our case, we can’t do it like that. To place dots on the circle we will use Polar Coordinates and will specify two coordinates:
1. how far away is the dot from the circle’s center (r),
2. dot’s angle (θ).

Having these coordinated we can easily convert them to Cartesian coordinates and draw afterward. The formulas are:
```dart
x = circleRadius * sin(dotAngle) + circleCenterX
y = circleRadius * cos(dotAngle) + circleCenterY
```

## Implementation
We have everything to start implementing the progress bar. We will use a `CustomPainter` class to draw the view on the canvas.

### The painters
Let’s add all arguments that we need and implement methods the `CustomPainter` requires, but the `paint` method.
```dart
class _DiagramPainter extends CustomPainter {
  _DiagramPainter({
    required this.dotRadius,
    required this.startBarRadius,
    required this.layers,
    required this.layersSpacing,
    required this.dotsSpacing,
    required this.progress,
    required Color dotColor,
    required Color progressDotColor,
  })  : _paint = Paint()
          ..color = dotColor
          ..style = PaintingStyle.fill,
        _progressPaint = Paint()
          ..color = progressDotColor
          ..style = PaintingStyle.fill,
        _dotDiameter = dotRadius * 2,
        _dotsSizeWithSpacing = dotRadius * 2 + dotsSpacing;

  final double dotRadius;
  final double startBarRadius;
  final int layers;
  final double layersSpacing;
  final double dotsSpacing;
  final double progress;

  final Paint _paint;
  final Paint _progressPaint;

  final double _dotDiameter;
  final double _dotsSizeWithSpacing;

  static const _startAngle = 270.0;
  static const _circle = 360.0;
  static const _halfCircle = 180.0;

  @override
  void paint(Canvas canvas, Size size) {
    // implement this later
  }

  @override
  bool shouldRepaint(covariant _DiagramPainter oldDelegate) =>
      dotRadius != oldDelegate.dotRadius ||
      startBarRadius != oldDelegate.startBarRadius ||
      layers != oldDelegate.layers ||
      layersSpacing != oldDelegate.layersSpacing ||
      dotsSpacing != oldDelegate.dotsSpacing ||
      progress != oldDelegate.progress;
}
```
