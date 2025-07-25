---
title: Grid areas
slug: Glossary/Grid_Areas
page-type: glossary-definition
sidebar: glossarysidebar
---

A **grid area** is one or more {{glossary("grid cell", "grid cells")}} that make up a rectangular area on the grid. Grid areas are created when you place an item using [line-based placement](/en-US/docs/Web/CSS/CSS_grid_layout/Grid_layout_using_line-based_placement) or when defining areas using [named grid areas](/en-US/docs/Web/CSS/CSS_grid_layout/Grid_template_areas).

![Image showing a highlighted grid area](1_grid_area.png)

Grid areas _must_ be rectangular in nature; it is not possible to create, for example, a T- or L-shaped grid area.

## Example

In the example below I have a grid container with two grid items. I have named these with the {{cssxref("grid-area")}} property and then laid them out on the grid using {{cssxref("grid-template-areas")}}. This creates two grid areas, one covering four grid cells, the other two.

```css hidden
* {
  box-sizing: border-box;
}

.wrapper {
  border: 2px solid #f76707;
  border-radius: 5px;
  background-color: #fff4e6;
}

.wrapper > div {
  border: 2px solid #ffa94d;
  border-radius: 5px;
  background-color: #ffd8a8;
  padding: 1em;
  color: #d9480f;
}
```

```css
.wrapper {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: 100px 100px;
  grid-template-areas:
    "a a b"
    "a a b";
}
.item1 {
  grid-area: a;
}
.item2 {
  grid-area: b;
}
```

```html
<div class="wrapper">
  <div class="item1">Item</div>
  <div class="item2">Item</div>
</div>
```

{{ EmbedLiveSample('Example', '300', '280') }}

## See also

### Property reference

- {{cssxref("grid-template-columns")}}
- {{cssxref("grid-template-rows")}}
- {{cssxref("grid-auto-rows")}}
- {{cssxref("grid-auto-columns")}}
- {{cssxref("grid-template-areas")}}
- {{cssxref("grid-area")}}

### Further reading

- CSS grid layout Guide:
  - [Basic concepts of grid layout](/en-US/docs/Web/CSS/CSS_grid_layout/Basic_concepts_of_grid_layout)
  - [Grid template areas](/en-US/docs/Web/CSS/CSS_grid_layout/Grid_template_areas)
- [Definition of grid areas in the CSS grid layout specification](https://drafts.csswg.org/css-grid/#grid-area-concept)
