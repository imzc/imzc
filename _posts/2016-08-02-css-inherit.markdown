---
layout: post
title:  "CSS Style inherit"
date:   2016-08-02 22:42:35 +0800
categories: CSS
---

Take a look at this

```css
h1{
    color: red;
}

.content h1{
    color: green;
}

.content .header h1 {
    color: blue;
}

.content .header{
    color: purple;
}
.content .header h1 {
    color: inherit;
}

```

The `h1` will take the color `purple` instead of `green`.

> The inherit CSS-value causes the element for which it is specified to take the computed value of the property from its parent element. It is allowed on every CSS property.

That's it.