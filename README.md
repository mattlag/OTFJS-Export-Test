## A quick demo of a bug in the latest code for opentype.js

In the `src` folder there is an `html` file that you can run with something like `npx http-server` so the module imports work.

It is able to generate two `otf` files, one using opentype.js v1.3.4, and another using the July 30th commit version of opentype.js

The generated font just uses the example from the docs to creatae a font object and save it out as an `otf`:

``` javascript
function saveOTFFile(old = false) {
  const opentype = old ? opentype_v1_3_4 : opentype_july_30;
  const name = `OTFJS-Export-Test-${old ? 'v1-3-4' : 'july30'}`;
  // Basically the example from the docs

  // this .notdef glyph is required.
  const notdefGlyph = new opentype.Glyph({
    name: '.notdef',
    advanceWidth: 650,
    path: new opentype.Path(),
  });

  const aPath = new opentype.Path();
  aPath.moveTo(100, 0);
  aPath.lineTo(100, 700);
  aPath.lineTo(0, 700);
  aPath.lineTo(0, 0);
  aPath.lineTo(100, 0);
  aPath.close();

  const aGlyph = new opentype.Glyph({
    name: 'A',
    unicode: 65,
    advanceWidth: 650,
    path: aPath,
  });

  const font = new opentype.Font({
    familyName: name,
    styleName: 'Medium',
    unitsPerEm: 1000,
    ascender: 800,
    descender: -200,
    glyphs: [notdefGlyph, aGlyph],
  });

  // Trigger save from browser
  const href = window.URL.createObjectURL(
    new Blob([font.toArrayBuffer()])
  );
  Object.assign(document.createElement('a'), {
    download: `${name}.otf`,
    href,
  }).click();
}
```

## Investigation

I opened the two resulting `otf` files in Fonttools TTX to see if I could spot any differences. Besides the different names of the fonts, the only thing I could find is that the new July 30th code adds the following sections.

I tried removing these via TTX and re-generating font files, but they still fail.

### platformID=0 name records in the NAME table
``` xml
  <namerecord nameID="0" platformID="0" platEncID="4" langID="0x0">
     
  </namerecord>
  <namerecord nameID="1" platformID="0" platEncID="4" langID="0x0">
    OTFJS-Export-Test-july30
  </namerecord>
  <namerecord nameID="2" platformID="0" platEncID="4" langID="0x0">
    Medium
  </namerecord>
  <namerecord nameID="3" platformID="0" platEncID="4" langID="0x0">
     :OTFJS-Export-Test-july30 Medium
  </namerecord>
  <namerecord nameID="4" platformID="0" platEncID="4" langID="0x0">
    OTFJS-Export-Test-july30 Medium
  </namerecord>
  <namerecord nameID="5" platformID="0" platEncID="4" langID="0x0">
    Version 0.1
  </namerecord>
  <namerecord nameID="6" platformID="0" platEncID="4" langID="0x0">
    OTFJS-Export-Test-july30Medium
  </namerecord>
  <namerecord nameID="7" platformID="0" platEncID="4" langID="0x0">
     
  </namerecord>
  <namerecord nameID="8" platformID="0" platEncID="4" langID="0x0">
     
  </namerecord>
  <namerecord nameID="9" platformID="0" platEncID="4" langID="0x0">
     
  </namerecord>
  <namerecord nameID="10" platformID="0" platEncID="4" langID="0x0">
     
  </namerecord>
  <namerecord nameID="11" platformID="0" platEncID="4" langID="0x0">
     
  </namerecord>
  <namerecord nameID="12" platformID="0" platEncID="4" langID="0x0">
     
  </namerecord>
  <namerecord nameID="13" platformID="0" platEncID="4" langID="0x0">
     
  </namerecord>
  <namerecord nameID="14" platformID="0" platEncID="4" langID="0x0">
     
  </namerecord>
  <namerecord nameID="16" platformID="0" platEncID="4" langID="0x0">
    OTFJS-Export-Test-july30
  </namerecord>
  <namerecord nameID="17" platformID="0" platEncID="4" langID="0x0">
    Medium
  </namerecord>
```

### the LTAG table
``` xml
  <ltag>
    <version value="1"/>
    <flags value="0"/>
    <LanguageTag tag="en"/>
  </ltag>
```


## So...
I don't know what is causing the new code to create font files that fail.
