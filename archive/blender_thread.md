---
title: "Printable threads in Blender"
layout: post
---

<script async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml.js" id="MathJax-script"></script>
<script>
MathJax = {
  tex: {
    inlineMath: [['$', '$'],['\$', '\$']]
  }
};
</script>

## Needs
+ make printable threads in blender
+ connect objects in the way of bolts and nuts
+ keep simple and generic

## Add-on
+ bolt factory

## Process

0. Assuming A(bolt) connects to B(nut).
1. Create a bolt using bolt factory.
2. Cut/scale it to expected size as bolt1(make sure the thread angle larger than 45 degree to keep printable).
3. Copy bolt1 to bolt2.
4. Boolean bolt1 to A at expected position in union mode.
5. Soldify bolt2 using $offset=1.0, thichkness=0.2cm$ to meet the print tolerance.
6. Reverse bolt2 along the axial direction for a small angle(e.g. $5$ degree) to meet the print tolerance.
7. Boolean bolt2 from B in difference mode.
8. Print.

<script src="https://utteranc.es/client.js"
        repo="jooooow/jooooow.github.io"
        issue-term="pathname"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>