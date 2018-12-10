---
title: Editable cells in Angular UI Grid not working
tags:
  - angularjs
  - ui-grid
url: 13.html
id: 13
categories:
  - Angular
  - Quick solutions
  - Web
date: 2015-10-17 15:38:00
---

This week at work I spent too much time on this small issue I had with [UI Grid](http://ui-grid.info/) so I thought I will share the solution. Angular [documentation tells you that](http://ui-grid.info/docs/#/api/ui.grid.edit.api:ColumnDef) it is possible to make a column editable by setting `enableCellEdit` to `true` in `columnDef`. What wasn't obvious for me, is that for this to work you also need to load the `ui.grid.edit` module and add the `uiGridEdit` directive to the element on which you enabled `uiGrid`. Code example below.

```javascript
$scope.gridOptions.columnDefs = [ 
  { name: 'name', enableCellEdit: true }, 
  // ...
]
```

```html
<div ui-grid="gridOptions" ui-grid-edit class="grid"></div><br>
```