go-colorful
===========
A library for playing with colors in go (golang).

Why?
====
I love games. I make games. I love detail and I get lost in detail.
One such detail popped up during the development of [Memory Which Does Not Suck](https://github.com/lucasb-eyer/mwdns/),
when we wanted the server to assign the players random colors. Sometimes
two players got very similar colors, which bugged me. The very same evening,
[I want hue](http://tools.medialab.sciences-po.fr/iwanthue/) was the top post
on HackerNews' frontpage and showed me how to Do It Right (tm). Last but not
least, there was no library for handling color spaces available in go.

What?
=====
Go-Colorful stores colors in RGB and provides methods from converting these to various color-spaces. Currently supported colorspaces are:

- **RGB:** All three of Red, Green and Blue in [0..1].
- **HSV:** Hue in [0..360], Saturation and Value in [0..1]. You probably shouldn't use this.
- **Hex RGB:** The "internet" color format, as in #FF00FF.
- **Linear RGB:** See [gamma correct rendering](http://www.sjbrown.co.uk/2004/05/14/gamma-correct-rendering/).
- **CIE-XYZ:** CIE's standard color space, almost in [0..1].
- **CIE-L\*a\*b\*:** A *perceptually uniform* color space, i.e. distances are meaningful. L\* in [0..1] and a\*, b\* almost in [-1..1].
- **CIE-L\*u\*v\*:** Very similar to CIE-L\*a\*b\*, there is [no consensus](http://en.wikipedia.org/wiki/CIELUV#Historical_background) on which one is "better".
- **CIE-L\*C\*h° (HCL):** This is generally the [most useful](http://vis4.net/blog/posts/avoid-equidistant-hsv-colors/) one; CIE-L\*a\*b\* space in polar coordinates, i.e. a *better* HSV. H° is in [0..360], C\* almost in [-1..1] and L\* as in CIE-L\*a\*b\*.

For the colorspaces where it makes sense (XYZ, Lab), the
[D65](http://en.wikipedia.org/wiki/Illuminant_D65) is used as reference white
by default but methods for using your own reference white are provided.

A coordinate being *almost in* a range means that generally it is, but for very
bright colors and depending on the reference white, it might overflow this
range slightly. For example, C\* of #0000ff is 1.338.

Unit-tests are provided.

What not (yet)?
===============
There are a few features which are currently missing and might be useful.
I just haven't implemented them yet because I didn't have the need for it.
Pull requests welcome.

- Functions for palette generation. (This is in the making.)
- Functions for computing distances in various spaces. Note that this seems to be [a whole science on its own](http://mmir.doc.ic.ac.uk/mmir2005/CameraReadyMissaoui.pdf).
- Functions for interpolation.

So which colorspace should I use?
=================================
It depends on what you want to do. I think the folks from *I want hue* are
on-spot when they say that RGB fits to how *screens produce* color, CIE L\*a\*b\*
fits how *humans perceive* color and HCL fits how *humans think* colors.

Whenever you'd use HSV, rather go for CIE-L\*C\*h°. for fixed lightness L\* and
chroma C\* values, the hue angle h° rotates through colors of the same
perceived brightness and intensity.

How?
====

### Installing
Installing the library is as easy as

```bash
$ go get github.com/lucasb-eyer/go-colorful
```

The package can then be used through an

```go
import "github.com/lucasb-eyer/go-colorful"
```

### Basic usage

Create a beautiful blue color using different source space:

```go
// Any of the following should be the same
c := colorful.Color{0.313725, 0.478431, 0.721569}
c := colorful.Hex("#517AB8")
c := colorful.Hsv(216.0, 0.56, 0.722)
c := colorful.Xyz(0.189165, 0.190837, 0.480248)
c := colorful.Lab(0.507850, 0.040585,-0.370945)
c := colorful.Luv(0.507849,-0.194172,-0.567924)
c := colorful.Hcl(276.2440, 0.373160, 0.507849)
Printf("RGB values: %v, %v, %v", c.R, c.G, c.B)
```

And then converting this color back into various color spaces:

```go
hex := c.Hex()
h, s, v := c.Hsv()
x, y, z := c.Xyz()
l, a, b := c.Lab()
l, u, v := c.Luv()
h, c, l := c.Hcl()
```

### Comparing colors
In the RGB color space, the Euclidian distance between colors *doesn't* correspond
to visual/perceptual distance. This means that two pairs of colors which have the
same distance in RGB space can look much further apart. This is fixed by the
CIE-L\*a\*b\*, CIE-L\*u\*v\* and CIE-L\*C\*h° color spaces.
Thus you should only compare colors in any of these space.
(Note that the distance in CIE-L\*a\*b\* and CIE-L\*C\*h° are the same, since it's the same space but in cylindrical coordinates)

![Color distance comparison](doc/colordist.png)

The two colors shown on the top look much more different than the two shown on
the bottom. Still, in RGB space, their distance is the same.
Here is a little example program which shows the distances between the top two
and bottom two colors in RGB, CIE-L\*a\*b\* and CIE-L\*u\*v\* space. You can find it in `doc/colordist.go`.

```go
package main

import "github.com/lucasb-eyer/go-colorful"

func main() {
    c1a := colorful.Color{150.0/255.0, 10.0/255.0, 150.0/255.0}
    c1b := colorful.Color{ 53.0/255.0, 10.0/255.0, 150.0/255.0}
    c2a := colorful.Color{10.0/255.0, 150.0/255.0, 50.0/255.0}
    c2b := colorful.Color{99.9/255.0, 150.0/255.0, 10.0/255.0}

    fmt.Printf("DistanceRgb: c1: %v and c2: %v\n", c1a.DistanceRgb(c1b), c2a.DistanceRgb(c2b))
    fmt.Printf("DistanceLab: c1: %v and c2: %v\n", c1a.DistanceLab(c1b), c2a.DistanceLab(c2b))
    fmt.Printf("DistanceLuv: c1: %v and c2: %v\n", c1a.DistanceLuv(c1b), c2a.DistanceLuv(c2b))
}
```

Running the above program shows that you should always prefer any of the CIE distances:

```bash
$ go run colordist.go
DistanceRgb: c1: 0.3803921568627451 and c2: 0.3858713931171159
DistanceLab: c1: 0.32048907700713997 and c2: 0.24397304315853596
DistanceLuv: c1: 0.513456934258258 and c2: 0.2568727826318425
```

Note that `AlmostEqualRgb` is provided mainly for (unit-)testing purposes. Use
it only if you really know what you're doing. It will eat your cat.

### Getting random palettes/colors
TODO

### Using linear RGB for computations
There are two methods for transforming RGB<->Linear RGB: a fast and almost precise one,
and a slow and precise one.

```go
r, g, b := colorful.Hex("#FF0000").FastLinearRgb()
```

TODO: describe some more.

### Want to use some other reference point?

```go
c := colorful.LabWhiteRef(0.507850, 0.040585,-0.370945, colorful.D50)
l, a, b := c.LabWhiteRef(colorful.D50)
```

FAQ
===

### Q: I get all f!@#ed up values! Your library sucks!
A: You probably provided values in the wrong range. For example, RGB values are
expected to reside between 0 and 1, *not* between 0 and 255. Normalize your colors.

License: MIT
============
Copyright (c) 2013 Lucas Beyer

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

