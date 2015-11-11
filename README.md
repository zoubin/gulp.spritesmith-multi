# gulp.spritesmith-multi
A wrapper for [gulp.spritesmith](https://github.com/twolfson/gulp.spritesmith) to generate multiple sprites and stylesheets.

[![npm](https://nodei.co/npm/gulp.spritesmith-multi.png?downloads=true)](https://www.npmjs.org/package/gulp.spritesmith-multi)

[![version](https://img.shields.io/npm/v/gulp.spritesmith-multi.svg)](https://www.npmjs.org/package/gulp.spritesmith-multi)
[![status](https://travis-ci.org/zoubin/gulp.spritesmith-multi.svg?branch=master)](https://travis-ci.org/zoubin/gulp.spritesmith-multi)
[![dependencies](https://david-dm.org/zoubin/gulp.spritesmith-multi.svg)](https://david-dm.org/zoubin/gulp.spritesmith-multi)
[![devDependencies](https://david-dm.org/zoubin/gulp.spritesmith-multi/dev-status.svg)](https://david-dm.org/zoubin/gulp.spritesmith-multi#info=devDependencies)

## Example

```javascript
var gulp = require('gulp')
var spritesmith = require('gulp.spritesmith-multi')

gulp.task('default', function () {
  return gulp.src('sp/**/*.png')
    .pipe(spritesmith())
    .pipe(gulp.dest('build'))
})
```

input:

```
⌘ tree sp
sp
├── hover
│   ├── sprite1--hover.png
│   ├── sprite1--hover@2x.png
│   ├── sprite1.png
│   ├── sprite1@2x.png
│   ├── sprite2.png
│   ├── sprite2@2x.png
│   ├── sprite3.png
│   └── sprite3@2x.png
├── normal
│   ├── sprite1.png
│   ├── sprite2.png
│   └── sprite3.png
└── retina
    ├── sprite1.png
    ├── sprite1@2x.png
    ├── sprite2.png
    ├── sprite2@2x.png
    ├── sprite3.png
    └── sprite3@2x.png
```

output:

```
⌘ tree build/
build/
├── hover.css
├── hover.png
├── hover@2x.png
├── normal.css
├── normal.png
├── retina.css
├── retina.png
└── retina@2x.png
```

hover.css

```css
.sp-hover {
  background-image: url(hover.png)
}

@media (-webkit-min-device-pixel-ratio: 2),
       (min-resolution: 192dpi) {
  .sp-hover {
    background-image: url(hover@2x.png)
    background-size: 150px 200px
  }
}

.sp-hover__sprite1:hover {
  background-position: -100px 0px
  width: 50px
  height: 50px
}
.sp-hover__sprite1 {
  background-position: -100px -50px
  width: 50px
  height: 50px
}
.sp-hover__sprite2 {
  background-position: -100px -100px
  width: 50px
  height: 50px
}
.sp-hover__sprite3 {
  background-position: 0px 0px
  width: 100px
  height: 200px
}
```

## Options

### to(iconFile)
Specify the name of the sprite into which the given icon should be included

Type: `Function`, `String`

If `String`, you just get one sprite.

By default, icons are grouped by their directory names.


#### spritesmith
Specify [options](https://github.com/twolfson/gulp.spritesmith#spritesmithparams)
for each sprite.

Type: `Object`, `Function`

The following fields are set by default:
```javascript
var options = {
  imgName: sprite + '.png',
  cssName: sprite + '.css',
  cssSpritesheetName: 'sp-' + sprite,
}

```

You can override them through this option.

If `Function`,
it receives the default options,
the sprite name specified by `options.to`
and the related icon files (vinyl file objects).
Modify the options object passed in, or return a new one.

## Custom templates

To specify custom templates,
create a templater through `exports.util.createTemplate`,
and set `options.spritesmith.cssTemplate` to it.

```javascript
var gulp = require('gulp')
var path = require('path')
var spritesmith = require('..')
var util = spritesmith.util

gulp.task('theme', ['clean'], function () {
  var opts = {
    spritesmith: function (options, sprite, icons){
      if (sprite.indexOf('hover--') !== -1) {
        options.cssTemplate = themeTemplate
      }
      return options
    },
  }
  var themeTemplate = util.createTemplate(
    path.join(__dirname, 'template', 'css.hbs'),
    [addTheme, util.addPseudoClass]
  )
  function addTheme(data) {
    var info = data.spritesheet_info
    var match = info.name.match(/hover--(\w+)/)
    data.theme = match && match[1]
  }
  return gulp.src('sp/**/*.png')
    .pipe(spritesmith(opts))
    .pipe(gulp.dest('build'))
})

```

Input:

The custom template

```handlebars
.{{{theme}}} .sp-hover {
  background-image: url({{{spritesheet.escaped_image}}});
}

{{#if retina_spritesheet}}
@media (-webkit-min-device-pixel-ratio: 2),
       (min-resolution: 192dpi) {
  .{{{theme}}} .sp-hover {
    background-image: url({{{retina_spritesheet.escaped_image}}});
    background-size: {{spritesheet.px.width}} {{spritesheet.px.height}};
  }
}
{{/if}}

{{#each sprites}}
.sp-hover__{{{name}}}{{pseudo_class}} {
  background-position: {{px.offset_x}} {{px.offset_y}};
  width: {{px.width}};
  height: {{px.height}};
}
{{/each}}

```

Icons

```
⌘ tree sp/hover*
sp/hover
├── sprite1--hover.png
├── sprite1--hover@2x.png
├── sprite1.png
├── sprite1@2x.png
├── sprite2.png
├── sprite2@2x.png
├── sprite3.png
└── sprite3@2x.png
sp/hover--theme
├── sprite1--hover.png
├── sprite1--hover@2x.png
├── sprite1.png
├── sprite1@2x.png
├── sprite2.png
├── sprite2@2x.png
├── sprite3.png
└── sprite3@2x.png

```

Output:

```
⌘ tree build/
build/
├── hover--theme.css
├── hover--theme.png
├── hover--theme@2x.png
├── hover.css
├── hover.png
├── hover@2x.png
├── normal.css
├── normal.png
├── retina.css
├── retina.png
└── retina@2x.png

```

hover--theme.css

```css
.theme .sp-hover {
  background-image: url(hover--theme.png);
}

@media (-webkit-min-device-pixel-ratio: 2),
       (min-resolution: 192dpi) {
  .theme .sp-hover {
    background-image: url(hover--theme@2x.png);
    background-size: 150px 200px;
  }
}

.sp-hover__sprite1:hover {
  background-position: -100px 0px;
  width: 50px;
  height: 50px;
}
.sp-hover__sprite1 {
  background-position: -100px -50px;
  width: 50px;
  height: 50px;
}
.sp-hover__sprite2 {
  background-position: -100px -100px;
  width: 50px;
  height: 50px;
}
.sp-hover__sprite3 {
  background-position: 0px 0px;
  width: 100px;
  height: 200px;
}

```



## Retina support
All retina icon files should be named like `xxx@2x.png`.

## Utils

### exports.util

Type: `Object`

Methods to work with.

#### createTemplate(tplInfo, filter)
Create a templater.

##### tplInfo
Specify template.

Type: `Object`

* `tplInfo.file`: the file path of the handlebars template
* `tplInfo.source`: the contents of the template

Either one of them should be specified.

If `tplInfo` is `String`, it is treated as a file path.

##### filter
Specify data for the template.

If `Array`, you are specifying an array of filters.

If `Object`, it will be mixed into the default [data](https://github.com/twolfson/gulp.spritesmith#templating).

If `Function`, you can modify the default data object,
or just return a new one.

#### addPseudoClass
A template data filter to support generating pseudo classes.

If the icon file is something like `name--pseduoClass.png`,
`.sp-sprite__name:pseduoClass` is created in the css,
rather than `.sp-sprite__name--pseduoClass`.

**NOTE**: for retina icons,
you should name them like `name--pseduoClass@2x.png`.