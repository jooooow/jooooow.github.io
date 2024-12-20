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

0. Assuming $A$(bolt) connects to $B$(nut).
1. Create a bolt using bolt factory.
2. Cut/scale it to the expected size, as $bolt_1$(make sure the thread angle larger than $90^\circ$ to keep printable).
3. Duplicate $bolt_1$ to $bolt_2$.
4. Boolean $bolt_1$ to $A$ at expected position in union mode.
5. Soldify $bolt_2$ using $\text{offset}=1.0$ and $\text{thichkness} \in0.1 \pm 0.05\text{ cm}$ to meet the print tolerance.
6. Optionally, reverse $bolt_2$ along the axial direction for a small angle(e.g. $5^\circ$) to meet the print tolerance.
7. Boolean $bolt_2$ from $B$ in difference mode.
8. Print.

## Example
![img](../img/blender_thread/printable_thread.gif)

<script src="https://utteranc.es/client.js"
        repo="jooooow/jooooow.github.io"
        issue-term="pathname"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>