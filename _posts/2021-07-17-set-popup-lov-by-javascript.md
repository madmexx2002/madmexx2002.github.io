---
layout: post
title: Set Apex Popup LOV Item with Javascript
subtitle: How to set the returning and display value of Item Popup LOV with Javascript
tags: [apex, javascript]
comments: true
---

The returning value of a Popup LOV can be set with 
```javascript
apex.item('P20100_STRUCTURE_ID').setValue('10');
```

As you can see in the picture below, the corresponding shuttle control is updated with the selected value of 10, but this **does not set the display value** either. The input element is still empty.

![PopupLOV without Inputitem value](https://madmexx2002.github.io/assets/img/posts/popuplov_set_without_inputitem.png)

The solution is very simple when you know both values. Just set the display value, too.

```javascript
$('#P20100_STRUCTURE_ID').val('I/PI-MA2 » I/PI-MA21');
```

![PopupLOV without Inputitem value](https://madmexx2002.github.io/assets/img/posts/popuplov_set_inputitem.png)

Now the display value is set right but the ui must be fixed to get the label over the value. Just add the class **js-show-label** to the container element from the Popup LOV item. Container elements always have the suffix **_CONTAINER**.

```javascript
$("#P20100_STRUCTURE_ID_CONTAINER").addClass("js-show-label");
```

![PopupLOV without Inputitem value](https://madmexx2002.github.io/assets/img/posts/popuplov_set.png)
