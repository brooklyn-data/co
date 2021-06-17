```
## Model configuration
label: marketing

connection: "snowflake_db"

datagroup: datagroup_name {
  label: "desired label"
  description: "desired description"
  max_cache_age: "6 hours"
  sql_trigger: select max(id) from table_name ;;
}

persist_with: datagroup_name

access_grant: access_grant_name {
  user_attribute: user_attribute_name
  allowed_values: [
    "value_1"
    , "value_2"
  ]
}

include: "/views/*.view.lkml"
include: "/dashboards/*.dashboard.lkml"

## Explores
explore: fact_retail_product_summary
  label: "Available Products"
  description: "Use this explore to answer questions about products we have in stock"
  always_filter: {
    filters: [is_in_stock: "yes"]
  }
  sql_always_where: ${created_date} >= '2017-01-01' ;;
  view_label: product_performance
  fields: [
    product_performance.product_id
    , product_performance.product_cost
    , product_performance.gross_sales_usd
    .
    .
    .
  ]
  join: dim_products {
    view_label: product_summary
    fields: [
      dim_products.product_id
      , dim_products.size
      , dim_products.vendor_name
      .
      .
      .
    ]
    relationship: many_to_one
    type: left_outer
    sql_on: ${product_performance.product_id} = ${dim_products.product_id}
  }

explore: ...

explore: ...
```
