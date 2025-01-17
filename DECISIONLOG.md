# Decision Log

In creating this package, which is meant for a wide range of use cases, we had to take opinionated stances on a few different questions we came across during development. We've consolidated significant choices we made here, and will continue to update as the package evolves. 

## Creating Empty Tables for Refunds, Order Line Refunds, and Order Adjustments 

Source tables related to `refunds`, `order_line_refunds`, and `order_adjustments` are created in the Shopify schema dyanmically. For example, if your shop has not incurred any refunds, you will not have a `refund` table yet until you do refund an order. 

Thus, the source package will create empty (1 row of all `NULL` fields) staging models if these source tables do not exist in your Shopify schema yet, and the transform package will work seamlessly with these empty models. Once `refund`, `order_line_refund`, or `order_adjustment` exists in your schema, the source and transform packages will automatically reference the new populated table(s). ([example](https://github.com/fivetran/dbt_shopify_source/blob/main/models/tmp/stg_shopify__refund_tmp.sql)). 

> In previous versions of the package, you had to manually enable or disable transforms of `refund`, `order_line_refund`, or `order_adjustment` through variables. Because this required you to monitor your Shopify account/schema and update the variable(s) accordingly, we decided to pursue a more automated solution.

## Keeping Deleted Entities 

Instead of automatically filtering out records where `_fivetran_deleted` is `true`, the package keeps these soft-deleted records, as they may persist as foreign keys in other tables. The package merely renames the deleted-flag to `is_deleted`, which you can filter out if you choose.

## Accepted Value Test Severity

We test the following columns for accepted values because their values are hard-coded to be pivoted out into columns and/or used as `JOIN` conditions in downstream models.
- `stg_shopify__price_rule.target_type`: accepted values are `line_item`, `shipping_line`
- `stg_shopify__price_rule.value_type`: accepted values are `percentage`, `fixed_amount`
- `stg_shopify__fulfillment.status`: accepted values are `pending`, `open`, `success`, `cancelled`, `error`, `failure`

We have chosen to make the severity of these tests `warn`, as non-accepted values will be filtered out in the transformation models. They will not introduce erroneous data.