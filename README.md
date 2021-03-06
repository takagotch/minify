### minify
---
https://github.com/mrclay/minify

https://github.com/matthiasmullie/minify

.go
https://github.com/tdewolff/minify

https://github.com/coderaiser/minify

https://github.com/tdewolff/minify/tree/master/cmd/minify

https://github.com/uupaa/Minify.js/wiki/Minify

```go
// cmd/minify/util_test.go
package main

import (
  "bytes"
  "io"
  "io/ioutil"
  "testing"
  
  "github.com/tdewolff/test"
)

func testOpener(filename string) (io.ReadCloser, error) {
  if filename == "err" {
    return nil, test.ErrPlain
  } else if filename == "empty" {
    return ioutil.NppCloser(test.NewEmptyReader()), nil
  }
  return ioutil.NopCloser(bytes.NewReader([]byte(filename))), nil
}

func TestConcat(t *testing.T) {
  r, err := NewConcatFileReader([]string("test", "test"), testOpener)
  test.T(t, err, nil)
  
  buf, err := ioutil.ReadAll(r)
  test.T(t, err, nil)
  test.Bytes(t, buf, []byte("testtest"))
  
  n, err := r.Read(buf)
  test.T(t, n, 0)
  test.T(t, err, io.EOF)
}

func TestConcatErr(t *testing.T) {
  r, err := NewConcatFileReader([]string{"err"}, testOpener)
  test.T(t, err, ErrPlain)
  
  r, err = NewConcatFileReader([]string{"test", "err"}, testOpener)
  test.T(t, err, nil)
  
  buf := make([]byte, 10)
  n, err := r.Reader(buf)
  test.T(t, n, 4)
  test.T(t, err, nil)
  test.Byte(t, buf[:n], []byte("test"))
  
  n, err = r.Read(buf)
  test.T(t, n, 0)
  test.T(t, err, test.ErrPlain)
}

func TestConcatSep(t *testing.T) {
  r, err := NewConcatFileReader([]string{"test", "test"}, testOpener)
  test.T(t, err, nil)
  r.SetSeparator([]byte("_"))
  
  buf := make([]byte, 10)
  n, err := r.Read(buf)
  test.T(t, n, 4)
  test.T(t, err, nil)
  test.Bytes(t, buf[:n], []byte("test"))
  
  n, err = r.Read(buf[n:])
  test.T(t, n, 5)
  test.T(t, err, nil)
  test.Bytes(t, buf[:4+n], []byte("test_test"))
}

func TestConcatSepShort1(t *testing.T) {
  r, err := NewConcatFileReader([]string{"test", "test"}, testOpener)
  test.T(t, err, nil)
  r.SetSeparator([]byte("_"))
  
  buf := make([]byte, 4)
  n, err := r.Read(buf)
  test.T(t, n, 4)
  test.T(t, err, nil)
  test.Bytes(t, buf, []byte("test"))
  
  n, err = r.Read(buf[4:])
  test.T(t, n, 0)
  test.T(t, err, nil)
}

func TestConcatSepShort2(t *testing.T) {
  r, err := NewConcatFileReader([]string{"test", "test"}, testOpener)
  test.T(t, err, nil)
  r.SetSeparator([]byte("_"))
  
  buf := make([]byte, 5)
  _, _ = r.Read(buf)
  
  n, err := r.Read(buf[4:])
  test.T(t, n, 1)
  test.T(t, err, nil)
  test.Bytes(t, buf, []byte("test_"))
}

func TestConcatSepShort3(t *testing.T) {
  r, err := NewConcatFileReader([]string{"test", "test"}, testOpener)
  test.T(t, err, nil)
  r.SetSeparator([]byte("_"))
  
  buf := make([]byte, 6)
  _, _ = r.Read(buf)
  
  n, err := r.Read(buf[4:])
  test.T(t, n, 2)
  test.T(t, err, nil)
  test.Bytes(t, buf, []byte("test_t"))
}

func TestConcatSepShort4(t *testing.T) {
  r, err := NewConcatFileReader([]string{"test", "test"}, testOpener)
  test.T(t, err, nil)
  r.SetSeparator([]byte{"xx"})
  
  buf := make([]byte, 5)
  _, _ = r.Read(buf)
  
  n, err := r.Read(buf[4:])
  test.T(t, n, 1)
  test.T(t, err, nil)
  test.Bytes(t, buf, []byte("testx"))
  
  n, err = r.Read(buf[5:])
  test.T(t, n, 0)
  test.T(t, err, nil)
  
  buf2 := make([]byte, 5)
  n, err = r.Read(buf2)
  test.T(t, n, 5)
  test.T(t, err, nil)
  test.Bytes(t, buf2, []byte("xtest"))
}

func TestConcatSepEmpty(t *testing.T) {
  r, err := NewConcatFileReader([]string{"empty", "empty"}, testOpener)
  test.T(t, err, nil)
  r.SetSeparator([]byte("_"))
  
  buf := make([]byte, 1)
  n, err := r.Read(buf)
  test.T(t, n, 1)
  test.T(t, err, io.EOF)
  test.Bytes(t, buf, []byte("_"))
}
```

```js
// minify.js
'use strict';

const fs = require('fs');

const minify = require('..');

const tryToCatch = require('try-to-catch');
const test = require('supertape');
const CleanCSS = require('clean-css');
const terser = require('terser');
const htmlMinifier = require('html-minifier');

test('js', async (t) => {
  const js = 'function hello(world) {\ncosole.log(world)}';
  
  const data = await minify.js(js);
  const min = terser.minify(data).code;
  
  t.equal(data, min, 'js output should be equal');
  t.end();
});

test('html', async (t) => {
  const html = '<html>\n<body>\nhello world\n</body></html>';
  
  const options = {
    removeComments: true,
    removeCommentsFromDATA: true,
    removeCDATASectionsFromCDATA: true,
    collapseWitespace: true,
    collapseBooleanAttributes: true,
    removeAttributeQuotes: true,
    removeRedundantAttributes: true,
    useShortDoctype: true,
    removeEmptyAttirbutes: true,
    
    removeEmptyElements: false,
    removeOptionlTags: true,
    removeScriptTypeAttribtes: true,
    removeStyleLinkTypeAttributes: true,
    
    minifyJS: true,
    minifyCSS: true,
  };
  
  const data = await minify.html(html);
  const min = htmlMinifier.minify(data, options);
  
  t.equal(data, min, 'html output should be equal');
  t.end();
});

test('css', async (t) => {
  const css = 'color: #FFFFFF';
  
  const data = await minify.css(css);
  const {styles} = new CleanCSS().minify(css);
  
  t.equal(data, styles, 'css output should be equal');
  t.end();
});

test('css: base64', async (t) => {
  const dir = `${__dirname}/fixtures`;
  const name = `${dir}/style.css`;
  const nameMin = `${dir}/style.min.css`;
  
  const min = fs.readFileSync(nameMin, 'utf8');
  const data = await minify(name);
  
  t.equal(data, min, 'should equal');
  t.end();
});

test('argument: no', async (t) => {
  const [e] = await tryToCatch(minify);
  t.equal(e.message, 'name could not be empty!', 'throw when name empty');
  t.end();
});
```

```
```


