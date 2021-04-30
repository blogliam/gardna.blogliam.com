---
title: SpriteScript
layout: portfolio
source: https://github.com/wg4568/spritescript
---

Spritescript is a serialization library for HTML5 canvas draw commands. It was written to allow graphics to be generated server-side, without the need to transmit large image files. "Why not use a vector format?" Because SpriteScript allows for variable rendering, and integrates nicely into an HTML5 canvas project.

![SpriteScript embedded editor](https://gardna.net/f/cdn/spritescript/editor.png)

## Example

Write the following code in the built-in editor:

```
# Tank Sprite
# username, angle, red, green, blue

LABEL FONT "bold 32px Comic Sans MS"
LABEL USERNAME "William"

PUSH 0              # rotate to angle
ROTATE

PUSH 180            # light grey fill color
PUSH 180
PUSH 180
PUSH 1
SET_FILL_COLOR

PUSH 5              # 5px stroke width
SET_LINE_WIDTH

BEGIN_PATH          # draw turret
PUSH 0
PUSH -20
PUSH -60
PUSH 40
RECT
FILL
STROKE

PUSH 255           # set player color
PUSH 0
PUSH 0
PUSH 1
SET_FILL_COLOR

BEGIN_PATH          # draw tank body
PUSH 0
PUSH 0
PUSH 40
PUSH 0
PUSH PI
PUSH 2
MULTIPLY
ARC
FILL
STROKE

PUSH 0              # restore angle
PUSH -1
MULTIPLY
ROTATE

PUSH FONT           # set font
SET_FONT

PUSH 2              # 2px line width
SET_LINE_WIDTH

PUSH USERNAME       # draw username
PUSH USERNAME
MEASURE_TEXT
PUSH -2
DIVIDE
PUSH 70
FILL_TEXT
STROKE_TEXT
```

Click compile, and the base64 will be copied.

```
Ahdib2xkIDMycHggQ29taWMgU2FucyBNUwdXaWxsaWFtAAAAAAAnAAAANEMAAAA0QwAAADRDAAAAgD8UAAAAoEAPLAAAAAAAAAAAoMEAAABwwgAAACBCHTAxAAAAf0MAAAAAAAAAAAAAAAAAgD8ULAAAAAAAAAAAAAAAAAAgQgAAAAAAANsPSUAAAAAAQAUcMDEAAAAAAAAAAIC/BScAAAAAABkAAAAAQA8AAACAPwAAAIA/KwAAAADABgAAAIxCIyQ=
```

This blob encapsulates the entire instruction set. Note how it could be easily transmitted or stored over the network either as raw binary or in a string format. Load up a canvas object in JavaScript and do the following:

```javascript
var sprite = SpriteScript.FromBase64("...");
sprite.render(canvasContext, { x: 100, y: 100 });
```

Your sprite will be rendered!

![Sprite with coordinate axis rendered](https://gardna.net/f/cdn/spritescript/william.png)

SpriteScript also supports arguments passed at render, using the `ARG` keyword. These are not yet supported by the editor, but can be used as follows:

```
PUSH ARG0
PUSH ARG1
PUSH ARG2
SET_FILL_COLOR

BEGIN_PATH
PUSH 0
PUSH 0
PUSH 100
PUSH 100
RECT
FILL
```

```javascript
var args = [255, 0, 0]; // [ ARG0, ARG1, ARG2 ]
var sprite = SpriteScript.FromBase64("...");
sprite.render(canvasContext, { x: 100, y: 100 }, args);
```

This will draw a rectangle with the color specified by the first three arguments passed to the render function. Args can also be strings, granted they are used in the correct context.

## Full API

```
SET_LINE_CAP            butt round square
SET_LINE_DASH_OFFSET    number
SET_LINE_JOIN           bevel round miter
SET_LINE_WIDTH          number
SET_MITER_LIMIT         number

SET_SHADOW_BLUR         number
SET_SHADOW_COLOR        r, g, b, a
SET_SHADOW_OFFSET       x, y

SET_FILL_COLOR          r, g, b, a
SET_STROKE_COLOR        r, g, b, a

SET_GLOBAL_ALPHA                number
SET_GLOBAL_COMPOSITE_OPERATION  <operation>
SET_IMAGE_SMOOTHING_ENABLED     boolean

SET_FONT                string
SET_TEXT_ALIGN          left right center start end
SET_TEXT_BASELINE       top hanging middle alphabetic ideographic bottom

ARC		    x, y, radius, startAngle, endAngle
RECT		  x, y, width, height
ELLIPSE		x, y, radiusX, radiusY, rotation, startAngle, endAngle

ARC_TO		    x1, y1, x2, y2, radius
BEZIER_TO	    cp1x, cp1y, cp2x, cp2y, x, y
QUADRATIC_TO	cpx, cpy, x, y

DRAW_IMAGE	image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight

FILL_TEXT	      text, x, y
STROKE_TEXT     text, x, y

LINE_TO		x, y
MOVE_TO		x, y

ROTATE		 angle
SCALE		   x, y
TRANSFORM	 a, b, c, d, e, f
TRANSLATE	 x, y

MEASURE_TEXT	text -> width

ADD		      a, b -> a+b
SUBTRACT	  a, b -> a-b
MULTIPLY	  a, b -> a*b
DIVIDE		  a, b -> a/b
MODULO		  a, b -> a%b
EXPONENT    a, b -> Math.pow(a, b)
ABS		      a -> |a|
FLOOR		    a -> Math.floor(a)
CEIL		    a -> Math.ceil(a)

PUSH <Val>  -> Val
POP

BEGIN_PATH
CLOSE_PATH

RESTORE
CLIP
FILL
STROKE

<operations>
    source-over
    source-in
    source-out
    source-atop
    destination-over
    destination-in
    destination-out
    destination-atop
    lighter
    copy
    xor
    multiply
    screen
    overlay
    darken
    lighten
    color-dodge
    color-burn
    hard-light
    soft-light
    difference
    exclusion
    hue
    saturation
    color
    luminosity
```
