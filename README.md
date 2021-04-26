# reveal-md

[reveal.js](https://revealjs.com) on steroids! Get beautiful reveal.js presentations from Markdown files.

## Installation

```bash
npm install -g reveal-md
```

## Usage

```bash
reveal-md path/to/my/slides.md
```

This starts a local server and opens any Markdown file as a reveal.js presentation in the default browser.

## Reveal.js v4

Thanks to [hugo-vrijswijk](https://github.com/hugo-vrijswijk) we are now using
[Reveal.js v4](https://github.com/hakimel/reveal.js/releases) in reveal-md v4!

## Docker

You can use Docker to run this tool without needing Node.js installed on your machine. Run the public Docker image,
providing your markdown slides as a volume. A few examples:

```bash
docker run --rm -p 1948:1948 -v <path-to-your-slides>:/slides webpronl/reveal-md:latest
docker run --rm -p 1948:1948 -v <path-to-your-slides>:/slides webpronl/reveal-md:latest --help
```

The service is now running at [http://localhost:1948](http://localhost:1948).

To enable live reload in the container, port 35729 should be mapped as well:

```bash
docker run --rm -p 1948:1948 -p 35729:35729 -v <path-to-your-slides>:/slides webpronl/reveal-md:latest /slides --watch
```

## Features

- [Installation](#installation)
- [Usage](#usage)
- [Reveal.js v4](#revealjs-v4)
- [Docker](#docker)
- [Features](#features)
  - [Markdown](#markdown)
  - [Theme](#theme)
  - [Highlight Theme](#highlight-theme)
  - [Custom Slide Separators](#custom-slide-separators)
  - [Custom Slide Attributes](#custom-slide-attributes)
  - [reveal-md Options](#reveal-md-options)
  - [Reveal.js Options](#revealjs-options)
  - [Speaker Notes](#speaker-notes)
  - [YAML Front matter](#yaml-front-matter)
  - [Live Reload](#live-reload)
  - [Custom Scripts](#custom-scripts)
  - [Custom CSS](#custom-css)
  - [Custom Favicon](#custom-favicon)
  - [Pre-process Markdown](#pre-process-markdown)
  - [Print to PDF](#print-to-pdf)
    - [1. Using Puppeteer](#1-using-puppeteer)
    - [2. Using Docker & DeckTape](#2-using-docker--decktape)
  - [Static Website](#static-website)
  - [Disable Auto-open Browser](#disable-auto-open-browser)
  - [Directory Listing](#directory-listing)
  - [Custom Port](#custom-port)
  - [Custom Template](#custom-template)
- [Scripts, Preprocessors and Plugins](#scripts-preprocessors-and-plugins)
- [Related Projects & Alternatives](#related-projects--alternatives)
- [Thank You](#thank-you)
- [License](#license)

### Markdown

The Markdown feature of reveal.js is awesome, and has an easy (and configurable) syntax to separate slides. Use three
dashes surrounded by two blank lines (`\n---\n`). Example:

```text
# Title

* Point 1
* Point 2

---

## Second slide

> Best quote ever.

Note: speaker notes FTW!
```

### Theme

Override theme (default: `black`):

```bash
reveal-md slides.md --theme solarized
```

See [available themes](https://github.com/hakimel/reveal.js/tree/master/css/theme/source).

Override reveal theme with a custom one. In this example, the file is at `./theme/my-custom.css`:

```bash
reveal-md slides.md --theme theme/my-custom.css
```

Override reveal theme with a remote one (use rawgit.com because the url must allow cross-site access):

```bash
reveal-md slides.md --theme https://rawgit.com/puzzle/pitc-revealjs-theme/master/theme/puzzle.css
```

### Highlight Theme

Override highlight theme (default: `zenburn`):

```bash
reveal-md slides.md --highlight-theme github
```

See [available themes](https://github.com/isagalaev/highlight.js/tree/master/src/styles).

### Custom Slide Separators

Override slide separator (default: `\n---\n`):

```bash
reveal-md slides.md --separator "^\n\n\n"
```

Override vertical/nested slide separator (default: `\n----\n`):

```bash
reveal-md slides.md --vertical-separator "^\n\n"
```

### Custom Slide Attributes

Use the [reveal.js slide attributes](https://revealjs.com/markdown/#slide-attributes) functionality to add HTML
attributes, e.g. custom backgrounds. Alternatively, add an HTML `id` attribute to a specific slide and style it with
CSS.

Example: set the second slide to have a PNG image as background:

```text
# slide1

This slide has no background image.

---

<!-- .slide: data-background="./image1.png" -->
# slide2

This one does!
```

### reveal-md Options

Define options similar to command-line options in a `reveal-md.json` file that must be located at the root of the
Markdown files. They'll be picked up automatically. Example:

```json
{
  "separator": "^\n\n\n",
  "verticalSeparator": "^\n\n"
}
```

### Reveal.js Options

Define Reveal.js [options](https://revealjs.com/config/) in a `reveal.json` file that must be located at the root
directory of the Markdown files. They'll be picked up automatically. Example:

```json
{
  "controls": true,
  "progress": true
}
```

### Speaker Notes

Use the [speaker notes](https://revealjs.com/speaker-view/) feature by using a line starting with `Note:`.

### YAML Front matter

Set Markdown (and reveal.js) options specific to a presentation with YAML front matter:

```
---
title: Foobar
separator: <!--s-->
verticalSeparator: <!--v-->
theme: solarized
revealOptions:
    transition: 'fade'
---
Foo

Note: test note

<!--s-->

# Bar

<!--v-->
```

### Live Reload

Using `-w` option changes to markdown files will trigger the browser to reload and thus display the changed presentation
without the user having to reload the browser.

### Custom Scripts

Inject custom scripts into the page:

```bash
reveal-md slides.md --scripts script.js,another-script.js
```

Don't use absolute paths, files should be in adjacent or descending folders.

### Custom CSS

Inject custom CSS into the page:

```bash
reveal-md slides.md --css style.css,another-style.css
```

Don't use absolute paths, files should be in adjacent or descending folders.

### Custom Favicon

If the directory with the markdown files contains a `favicon.ico` file, it will automatically be used as a favicon
instead of the [default favicon](lib/favicon.ico).

### Pre-process Markdown

`reveal-md` can be given a markdown preprocessor script via the `--preprocessor` (or `-P`) option. This can be useful to
implement custom tweaks on the document format without having to dive into the guts of the Markdown parser.

For example, to have headers automatically create new slides, one could have the script `preproc.js`:

```javascript
// headings trigger a new slide
// headings with a caret (e.g., '##^ foo`) trigger a new vertical slide
module.exports = (markdown, options) => {
  return new Promise((resolve, reject) => {
    return resolve(
      markdown
        .split('\n')
        .map((line, index) => {
          if (!/^#/.test(line) || index === 0) return line;
          const is_vertical = /#\^/.test(line);
          return (is_vertical ? '\n----\n\n' : '\n---\n\n') + line.replace('#^', '#');
        })
        .join('\n')
    );
  });
};
```

and use it like this

```bash
$ reveal-md --preprocessor preproc.js slides.md
```

### Print to PDF

There are (at least) two options to export a deck to a PDF file.

#### 1. Using Puppeteer

Create a (printable) PDF from the provided Markdown file:

```bash
reveal-md slides.md --print slides.pdf
```

The PDF is generated using Puppeteer. Alternatively, append `?print-pdf` to the url from the command-line or in the
browser (make sure to remove the `#/` or `#/1` hash). Then print the slides using the browser's (not the native) print
dialog. This seems to work in Chrome.

By default, paper size is set to match options in your [`reveal.json`](#revealjs-options) file, falling back to a
default value 960x700 pixels. To override this behaviour, you can pass custom dimensions or format in a command line
option `--print-size`:

```bash
reveal-md slides.md --print slides.pdf --print-size 1024x768   # in pixels when no unit is given
reveal-md slides.md --print slides.pdf --print-size 210x297mm  # valid units are: px, in, cm, mm
reveal-md slides.md --print slides.pdf --print-size A4         # valid formats are: A0-6, Letter, Legal, Tabloid, Ledger
```

In case of an error, please try the following:

- Analyze debug output, e.g. `DEBUG=reveal-md reveal-md slides.md --print`
- See `reveal-md help` for Puppeteer arguments (`puppeteer-launch-args` and `puppeteer-chromium-executable`)
- Use Docker & DeckTape:

#### 2. Using Docker & DeckTape

The first method of printing does not currently work when running reveal-md in a Docker container, so it is recommended
that you print with [DeckTape](https://github.com/astefanutti/decktape) instead. Using DeckTape may also resolve issues
with the built-in printing method’s output.

To create a PDF of a presentation using reveal-md running on your localhost using the DeckTape Docker image, use the
following command:

```bash
$ docker run --rm -t --net=host -v $OUTPUT_DIR:/slides astefanutti/decktape $URL $OUTPUT_FILENAME
```

Replace these variables:

- `$OUTPUT_DIR` is the folder you want the PDF to be saved to.
- `$OUTPUT_FILENAME` is the name of the PDF.
- `$URL` is where the presentation can be accessed in your browser (without the `?print-pdf` suffix). If you are not
  running reveal-md in Docker, you will need to replace `localhost` with the IP address of your computer.

For a full list of export options, please see the the [DeckTape github](https://github.com/astefanutti/decktape), or run
the Docker container with the `-h` flag.

### Static Website

This will export the provided Markdown file into a stand-alone HTML website including scripts and stylesheets. The files
are saved to the directory passed to the `--static` parameter (default: `./_static`):

```bash
reveal-md slides.md --static _site
```

This should copy images along with the slides. Use `--static-dirs` to copy directories with other static assets to the
target directory. Use a comma-separated list to copy multiple directories.

```bash
reveal-md slides.md --static --static-dirs=assets
```

Providing a directory will result in a stand-alone overview page with links to the presentations (similar to a
[directory listing](#directory-listing)):

```bash
reveal-md dir/ --static
```

By default, all `*.md` files in all subdirectories are included in the generated website. Provide a custom
[glob pattern](https://github.com/isaacs/node-glob) using `--glob` to generate slides only from matching files:

```bash
reveal-md dir/ --static --glob '**/slides.md'
```

Additional `--absolute-url` and `--featured-slide` parameters could be used to generate [OpenGraph](http://ogp.me)
metadata enabling more attractive rendering for slide deck links when shared in some social sites.

```bash
reveal-md slides.md --static _site --absolute-url https://example.com --featured-slide 5
```

### Disable Auto-open Browser

To disable auto-opening the browser:

```bash
reveal-md slides.md --disable-auto-open
```

### Directory Listing

Show (recursive) directory listing of Markdown files:

```bash
reveal-md dir/
```

Show directory listing of Markdown files in current directory:

```bash
reveal-md
```

### Custom Port

Override port (default: `1948`):

```bash
reveal-md slides.md --port 8888
```

### Custom Template

Override reveal.js HTML template
([default template](https://github.com/webpro/reveal-md/blob/master/lib/template/reveal.html)):

```bash
reveal-md slides.md --template my-reveal-template.html
```

Override listing HTML template
([default template](https://github.com/webpro/reveal-md/blob/master/lib/template/listing.html)):

```bash
reveal-md slides.md --listing-template my-listing-template.html
```

## Scripts, Preprocessors and Plugins

- [reveal-md-scripts](https://github.com/amra/reveal-md-scripts)

## Related Projects & Alternatives

- [Slides](https://slides.com/) is a place for creating, presenting and sharing slide decks.
- [Sandstorm Hacker Slides](https://github.com/jacksingleton/hacker-slides) is a simple app that combines Ace Editor and
  RevealJS.
- [Tools](https://github.com/hakimel/reveal.js/wiki/Plugins,-Tools-and-Hardware#tools) in the Plugins, Tools and
  Hardware section of Reveal.js.
- [Org-Reveal](https://github.com/yjwen/org-reveal) exports Org-mode contents to Reveal.js HTML presentation.
- [DeckTape](https://github.com/astefanutti/decktape) is a high-quality PDF exporter for HTML5 presentation frameworks.
- [GitPitch](https://gitpitch.com) generates slideshows from PITCHME.md found in hosted Git repos.

## Thank You

Many thanks to all [contributors](https://github.com/webpro/reveal-md/graphs/contributors)!

## License

[MIT](http://webpro.mit-license.org)
