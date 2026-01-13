---
title: A Streamlined Typst Workflow for Academic Plotting
categories: [技术笔记]
tags: [Typst]
description: A lightweight Typst utility for maintaining a single figure library while seamlessly exporting individual, auto-cropped vector graphics for LaTeX or PowerPoint.
---


[Typst](https://typst.app/), combined with [CeTZ](https://cetz-package.github.io/), is a powerful alternative to TikZ or Matplotlib for creating academic illustrations. However, when working on a paper or a presentation (LaTeX/PPT), managing dozens of separate `.typ` files for different figures becomes a headache.

The most efficient way is to maintain a **single figure library** and export exactly what you need. This snippet provides a specialized environment to do just that.

## Key Features

Our realization offers two distinct modes to bridge the gap between "designing" and "exporting":

| Feature        | Debug Mode (`debug: true`)                       | Output Mode (`debug: false`)                          |
| -------------- | ------------------------------------------------ | ----------------------------------------------------- |
| **Visibility** | Renders all figures in the file (multiple pages) | Renders **only one** specified figure (a single page) |
| **Canvas**     | Auto-cropped to figure size with breaks          | Auto-cropped to figure size (perfect for PDF/SVG)     |
| **Metadata**   | Displays Index & Caption above each plot         | Clean output (no text labels)                         |
| **Use Case**   | Organizing and identifying figures               | Final export for LaTeX/PPT                            |

---

## How to Use

1. Save the `plot-export.typ` code (found in the [GitHub Repo](https://github.com/itpyi/typst-plot)) in your project directory. (We may publish it later.)
2. Create your library file and use the `#show` rule to toggle between modes:

```typst
#import "plot-export.typ": *

#show: plot-export.with(
  debug: true,            // Set to false for final export
  output: "my-plot",      // Target by caption...
  // output-num: 1        // ...or target by index (0, 1, 2...)
)

#fig("caption for the first figure")[
  // Your CeTZ code or diagram here
]

#fig("caption for the second figure")[
  // Another figure
]

```

## The Workflow

1. **Design in Debug Mode:** Keep `debug: true`. You will see all your figures listed with their index numbers and captions. See below for an example for the debug info:
  > Debug mode. Switch to output mode in show rules. <br>
  > Figure No. 0 with caption: `caption for the first figure`


1. **Export for Publication:** Once ready, specify the `output` caption or `output-num` and set `debug: false`. Typst will render a single page perfectly cropped to your figure's boundaries, ready to be imported as a vector graph. Note that the recommended workflow is to keep `debug: true` while you find the figure you need. Once you've copied the caption/index into the show rule, simply flip `debug` to `false` for a clean export.

## Error Handling

To prevent "ghost" exports or accidental overwrites, the snippet includes built-in guards:

* **Conflict Prevention:** If you accidentally set both `output` and `output-num`, the output file will warn you: 
  > Please specify the rendered figure by caption *or* number, but not by both!
* **Index Bounds:** If you request an index that doesn't exist, you'll receive an error: 
  > Error: `output-num` larger than total figure number!

Interestingly, we directly output these messages in the rendered file, since the common workflow with Typst is to type and view the output file simultaneously. Reading the file is faster than reading the compiler message elsewhere.