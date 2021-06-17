```
## Configuration
- dashboard: overall_sales
  title: Overall sales
  layout: grid
  .
  .
  .

## Filters
filters:
  - name: date
    title: 'Date Range'
    type: date_filter
    .
    .
    .

## Embedded dashboard
embed_style:
    background_color:
    show_title:
    .
    .
    .

## Elements
  ## Basic parameters
  - name: [unique name for the dashboard element]
    title: [text to display as title for this element]
    type: [valid Dashboard LookML type]
  ## Layout
    row: [integer]
    col: [ integer (0-23) ]
    width: [integer]
    height: [integer]
  ## Tool tip
    note:
      text: [text to be displayed]
      state: [collapsed or expanded if the text is too big to fit on a single row within the element's width]
      display: [display format]
  ## Measures and dimensions
    model: [model]
    explore: [explore]
    type: [type]
    fields: [
      table.column_1
      , table.column_2
      , table.column_3
    ]
    dynamic_fields:
    - table_calculation: [name of table calculation]
      label: [label]
      expression: [table calculation]
      value_format:
      value_format_name:
      _kind_hint:
      _type_hint:

    - table_calculation: [name of table calculation]
      label: [label]
      expression: [table calculation]
      value_format:
      value_format_name:
      _kind_hint:
      _type_hint:
  ## Filters
    filter_expression: [Looker custom filter expression]
    filters:
      table.column_name: [filter expression]
    listen:
      dashboard_filter_name: dimension_or_measure_name
    hidden_fields: [table.column_2]
  # Axes, legend, series attributes, and sorts
    x_axis_label: [text to display on x-axis]
    series_type: [e.g., line, column, area, etc.]
    series_colors:
      column_1:
      column_2:
    y_axis_value_format:
    sorts: [
      table.column_1
      , table.column_2
    ]
```
