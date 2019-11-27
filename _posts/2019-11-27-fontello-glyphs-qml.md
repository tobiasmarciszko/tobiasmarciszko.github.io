---
layout: post
title: Fontello glyph fonts in QML
date: 2019-11-27 20:24
category: 
author: 
tags: [Qt, QML, Fontello]
summary: 
image: qt-logo-big.png
---

I have used "glyphs" as icons in several projects in the past for web, mobile and desktop. That is, you have a font file with specific characters that acts as icons on your buttons, labels and pretty much whatever you like. I won't discuss the pros and cons here in using them, but I can strongly recommend this approach as an alternative to using standard images in your applications. 

The "glyph" font, or "iconic" font, is a font with custom symbols. Once imported you use the corresponding code to type the symbol, just as typing out unicode characters.

I started using [Fontello](http://fontello.com/) which is an amazing open source online tool for importing or selecting and packaging symbols into a ready made package for download. This package includes the font to include and use, but also a web page that showcases all icons and their respective codes in your font. That makes it very easy to use and lookup the symbols!

So, head over to [Fontello](http://fontello.com/), create your font and download the package!

Once downloaded, unpack the archive and add the fontello.ttf font to one of your resources files, using Qt Creator: 

![fontello in qrc](/img/fontello_qrc_ttf.png)

When adding files to your resource file you can also add an alias to your file for easier reference later on. I've decided to call my font resource "fontello".

Next step is to start using your font file in QML. What you need to do is to load the font using the [FontLoader](https://doc.qt.io/qt-5/qml-qtquick-fontloader.html) QML type.

```qml
FontLoader {
    id: glyphs
    source: "fontello"
}
```

By doing this you can start using the font in your components: 

```qml
RoundButton {
        font.family: glyphs.name
        font.pointSize: 18
        text: "\uf1cb"
}
```

Here, the text is the symbol's code which can be seen in the demo page provided in the package:

![fontello demo codes](/img/fontello_demo_codes.png)

Below is a screenshot from the Freeside Raytracer with added glyphs for the buttons and labels used. The highlighted button is the button in the example above. 

![raytracer glyphs](/img/raytracer_glyphs.png)

If you don't want to, or can't, use Fontello that is just fine. Create your own font in any way you like, and use the FontLoader type to load it and start using the font in your components! :)

That's all folks! :)

