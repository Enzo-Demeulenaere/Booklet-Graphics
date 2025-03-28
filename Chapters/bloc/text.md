## About text 

### Introduction

Bloc comes with a full API to deal with text. Not only you can deal with
raw text, but you can apply styles as well.

Before being displayed, you define your text as an instance of `BlRopedText`,
which you can then style with **attributes** such as:

- font name
- font size
- font foreground and background color
- font style (normal, italic, bold).

```smalltalk
labelText := 'hello from bloc' asRopedText
foreground: Color orange;
fontSize: 16;
fontName: 'Helvetica';
thin.

label := BlTextElement new.
label text: labelText.

label openInNewSpace
```

Another way to define attributes is to pass them as
a collection:

```smalltalk
BlTextElement new
        position: 5 @ 5;
        text: ('Rainbow!' asRopedText attributes:
        { (BlTextForegroundAttribute paint: Color black)})

```

Take a look at `BlText` method for a full list of available text attributes.

Fonts are managed directly by Alexandrie. To get the list of available fonts,
take a look at the result of `AeFontManager globalInstance familyNames`

text is like a collection, and you can apply different attributes to different
part of your text:

```smalltalk
    labelText := 'a := A->B->>C <= c|=>d~~>e.' asRopedText
                        background: Color orange;
                        fontSize: 25;
                        fontName: 'Cascadia Code';
                            underline;
                            underlineColor: Color red.

    (labelText from: 1 to: 5) foreground: Color blue.
    (labelText from: 7 to: 11) foreground: Color white.
    (labelText from: 12 to: 15) foreground: Color red.

    label := (BlTextElement text: labelText)
                    position: 50 @ 10;
                    background: Color yellow;
                    margin: (BlInsets all: 10);
                    padding: (BlInsets all: 5);
                    outskirts: BlOutskirts centered;
                    border: (BlBorder paint: Color red width: 5).
    label tightMeasurement.
```

![Multiple attributes.](figures/multipleTextAttributes.png width=60)


### Text size

The size of your text will depend on the font you have selected. This font will
constrain aspect of the size of letters, words, and text. 
Let's familiarize ourselves with those basic measures. 
Bloc will get the measures (in `BATextParagraphSpan >> measure`)
to get the size of your text, and position it into your element.

Figure *@MulAttri@* describes the information listed below: 

**width**
:  This is the width of the glyph image's bounding box.

**height**
: This is the height of the glyph image's bounding box.

**advance**
:  Distance to increment the pen position when the glyph is drawn as part of a string of text.

**bearingX**
:  Distance from the current cursor position to the leftmost border of the glyph image's bounding box.

**bearingY**
: Distance from the current cursor position (on the baseline) to the topmost border of the glyph image's bounding box.

![Describing various font elements.](figures/glyph-metrics-3.png width=40&label=MulAttri)

**ascent**
:    portion of letter that extends above the mean line of a font.

**descent**
:    portion of a letter that extends below the baseline of a font.

**baseline**
:    line upon which most letters sit and below which descenders extend.

![Typograpy.](figures/2880px-TypographyLineTerms.pdf width=60)

References:

- [wikipedia](https://en.wikipedia.org/wiki/Ascender_(typography))
- [freetype](https://freetype.org/freetype2/docs/tutorial/step2.html)

Internally, your text will be split into a collection of *spans*. 
A span is a homogeneous styled piece of text where every character has the same set of
attributes.

### Text bounds

Text can after be rendered as `BlTextElement`, which will take care of
displaying properly the text with all properties defined on it.

Once you have a `BlTextElement`, all properties of `BlElement` apply. 
You can add your element to any existing `BlElement`, and integrate it easily in
your graphical interface; it'll follow the same layout rules.

`BlTextElement` has 3 available measures by default that determine its bounds.

- tight measurement: Exact width and height of the used glyphs.
- label measurement: Same width that tight measurement. The height will add to itself the *ascent* and *descent* of the glyph.
- editor measurement. Same height as label measurement. The width will add to itself the *advance* of the glyph.

![Text measures.](figures/textMeasure.png width=80)

By default, *BlTextElement* will follow the *tightMeasurement* measure.

### Examples

Rectangle surrounded by number:

```smalltalk
BlElement new
    layout: (BlGridLayout horizontal columnCount: 3);
    constraintsDo: [ :c |
        c horizontal matchParent.
        c vertical matchParent ];
    addChildren: {
        "top row"
        (BlTextElement new text: '5,0' asRopedText).
        (BlElement new size: 0@0).
        (BlTextElement new text: '13,0' asRopedText).
        
        "middle row"
        (BlElement new size: 0@0).
        (BlElement new
            constraintsDo: [ :c |
                c horizontal matchParent.
                c vertical matchParent ];
            border: (BlBorder paint: Color gray width: 1)).
        (BlElement new size: 0@0).
        
        "bottom row"
        (BlTextElement new text: '5,25' asRopedText).
        (BlElement new size: 0@0).
        (BlTextElement new text: '13,25' asRopedText). }.

```

whose result is shown in Figure *@rect@*.

![Rectangle with numbers.](figures/rectangleWithNumbers.png width=60&label=rect)



### BlText vs. BlTextElement

You have 2 different levels to manage text in Bloc.
- `BlText` and its subclass `BlRopedText` create a text model where you can specify its attributes and style.
- `BlTextElement` and its subclasses will properly display the text inside a Bloc element.

A small example. You can notice that `BlText` background is different from `BlTextElement` background. 

```smalltalk
| labelText label |
labelText := 'hello from bloc' asRopedText
             background: Color orange ;
             fontSize: 75;
             fontName: 'Source Code Pro';
             italic;
             underline;
             underlineColor: Color red;
             vertical.

(labelText from: 1 to: 5) foreground: Color blue.
(labelText from: 7 to: 11) foreground: Color white.
(labelText from: 12 to: 15) foreground: Color red.

label := (BlTextElement text: labelText) position: 50 @ 10; background: Color yellow.
```

you can define the style of your text through BlTextAttributesStyler

````smalltalk
text := 'Hi John' asRopedText.

styler := BlTextAttributesStyler on: (text from: 3 to: 7).
styler
bold;
italic;
fontSize: 30;
fontName: 'Roboto';
monospace;
foreground: Color green.
styler style.
```

or using a fluent API

````smalltalk
text := 'Hi John' asRopedText.
(text from: 3 to: 7) stylerDo: [ :aStyler | aStyler bold italic foreground: Color red ].
````

As you may have noticed, this gives you a very fine-grained control over the style of your text.
You also need to re-specify attributes when your text changes.
If you want all your text to use the same attribute, you can then use `BlAttributedTextElement`. 
You can then change your text, `BlAttributedTextElement` will reuse its attributes.


```smalltalk
text := 'Hi John' asRopedText.

element := BlAttributedTextElement new.
attributes := element attributesBuilder
              foreground: Color green;
              monospace;
              bold;
              italic;
              fontSize: 30;
              fontName: 'Roboto';
              monospace.

label := (element text: text)
         position: 50 @ 10;
         background: Color yellow;
         margin: (BlInsets all: 2);
         padding: (BlInsets all: 2);
         outskirts: BlOutskirts centered;
         border: (BlBorder paint: Color red width: 2).

element text: 'hello world' asRopedText.
label.
```

### upload a font to bloc

you can load a directory with fonts (e.g. .ttf and .otf files) this way: 
`AeFontManager globalInstance scanDirectory: '../../fonts' asFileReference`

There is also 
`AeFontManager globalInstance scanDefaultDirectories`
which takes a second to find font files in the typical font directories in current platform (OS)

For example, say you have a font called "Minecraft" in a relative directory. 
This font has only "Medium" type (this is important, too):

```smalltalk 
AeFontManager globalInstance scanDirectory: '../../fonts' asFileReference.

BlTextElement new
    text: ('Hola, Bonjour, Hello!' asRopedText
            fontName: 'Minecraft' "<--- family name";
            medium; "<--- weight"
            fontSize: 50;
            yourself);
    openInNewSpace.
```

This can be tricky: You must set carefully not only the "family name", but also
3 font properties: "slant" (e.g.  normal/italic), "weight" (e.g. regular, bold,
light) and "stretch". In my example, when I set the Minecraft family name but
don't set the medium weight, then by default it will lookup for a Minecraft
regular weight font, it won't find it (the TTF only comes with a Medium weight
face), and then will use the default font.
