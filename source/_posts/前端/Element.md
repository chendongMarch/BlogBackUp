---
abbrlink: '0'
hide: true
---
# Element 

## Table hover 时高亮

`.table-content` 是给 `el-table` 加的 `class`

```css
.table-content.el-table .el-table__row:hover {
  background: transparent;
}
.table-content.el-table tr:hover {
  background: transparent;
}
.el-table tbody tr:hover > td {
  background-color: transparent !important
}
```