```
include: "shared_fields.view"

view: fact_orders {
sql_table_name: "FACT_ORDERS" ;;
extends: [shared_fields]

  ## Primary key
  dimension: order_id {
    primary_key: yes
    type: string
    sql: ${TABLE}."ORDER_ID" ;;
  }

  ## Parameters
  parameter: date_granularity {
    type: unquoted
    allowed_value: {
      value: "Day"
      label: "Date"
    }
    allowed_value: {
      value: "Week"
      label: "Week"
    }
    allowed_value: {
      value: "Month"
      label: "Month"
    }
    allowed_value: {
      value: "Quarter"
      label: "Quarter"
    }
    allowed_value: {
      value: "Year"
      label: "Year"
    }
    default_value: "Day"
  }

  parameter: order_type {
    label: "Order Type"
    allowed_value: {
    label: "Test order"
    value: "test"
  }
    allowed_value: {
      label: "Customer order"
      value: "order"
    }
  }

  ## Filters
  filter: state {
    type: string
    description: "Choose the states you want to report on"
    label: "Customer State"
  }

  ## Foreign keys and IDs
  ### Hidden
  dimension: address_id {
    hidden: yes
    type: string
    sql: ${TABLE}."ADDRESS_ID" ;;
  }
  dimension: customer_id {
    hidden: yes
    type: string
    sql: ${TABLE}."CUSTOMER_ID" ;;
  }

  ## Timestamps
  dimension_group: order_created {
    type: time
    timeframes: [
      raw,
      time,
      date,
      week,
      month,
      quarter,
      year
    ]
    sql: ${TABLE}."ORDER_CREATED_AT_PT" ;;
  }
  dimension_group: order_shipped {
    type: time
    timeframes: [
      raw,
      time,
      date,
      week,
      month,
      quarter,
      year
    ]
    sql: ${TABLE}."ORDER_SHIPPED_AT_PT" ;;
  }
  ### Hidden
  dimension_group: order_updated {
    hidden: yes
    type: time
    timeframes: [
      raw,
      time,
      date,
      week,
      month,
      quarter,
      year
    ]
    sql: ${TABLE}."ORDER_UPDATED_AT_PT" ;;
  }

  ## Duration
  dimension_group: order_to_ship {
    type: duration
    intervals: [
      minute
      , hour
      , day
      , week
    ]
    sql_start: ${order_created_raw} ;;
    sql_end: ${order_shipped_raw} ;;
    view_label: "Duration"
    group_label: "Order to Ship Time"
    description: "Measures the time between order confirmation and EasyPost shipping confirmation"
  }

  ## Flags
  dimension: is_delivered {
    type: yesno
    sql: ${TABLE}."IS_DELIVERED" ;;
  }
  dimension: is_domestic_order {
    type: yesno
    sql: ${TABLE}."IS_DOMESTIC_ORDER" ;;
  }

  ## Properties
  dimension: warehouse_location {
    type: string
    sql: ${TABLE}."WAREHOUSE_LOCATION_NAME" ;;
  }

  ## Statuses
  dimension: order_status {
    type: string
    sql: ${TABLE}."ORDER_STATUS"
  }

  ## Revenue
  measure: total_order_subtotal_usd {
    type: sum
    sql: ${order_subtotal_usd} ;;
    value_format_name: usd
    label: "Order Subtotal USD"
  }

  ## Revenue
  measure: total_taxes_usd {
    type: sum
    sql: ${_total_taxes_usd} ;;
    value_format_name: usd
    label: "Total Taxes USD"
  }
  ### Hidden
  dimension: order_subtotal_usd {
    hidden:yes
    type: number
    sql: ${TABLE}."ORDER_SUBTOTAL_USD" ;;
  }
  dimension: _total_taxes_usd {
    hidden: yes
    type: number
    sql: ${TABLE}."TOTAL_TAXES_USD" ;;
  }

  ## Costs
  measure: total_cart_discount {
    type:sum
    sql: ${cart_discount} ;;
    value_format_name: usd
  }
  ### Hidden
  dimension: cart_discount {
    hidden: yes
    type: number
    sql: ${TABLE}."CART_DISCOUNT" ;;
  }

  ## Ratios and Percents
  measure: domestic_to_international_orders {
    type: number
    sql: count(case when ${is_domestic_order} then order_id end)
        / count(case when not ${is_domestic_order} then order_id end) ;;
    value_format_name: decimal_1
  }

  ## Sets
  set: revenue_fields {
    fields: [
      , order_subtotal_usd
      , total_taxes_usd
    ]
  }
}
```
