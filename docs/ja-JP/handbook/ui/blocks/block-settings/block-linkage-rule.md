# Block Linkage Rules

> **Note**: This feature is supported starting from v1.7.0-beta.24.

## Introduction

Block linkage rules allow users to dynamically control the visibility of blocks, enabling the management of element display at the block level. As containers for fields and action buttons, blocks enable users to flexibly control the visibility of the entire view from the block's perspective.

![20250427140619](https://static-docs.nocobase.com/20250427140619.png)

![20250427144259](https://static-docs.nocobase.com/20250427144259.png)

> **Note**: Before executing the block linkage rules, the block's visibility must first pass through **ACL permission checks**. Only when the user has the corresponding access rights can the block linkage rule logic be applied. In other words, the block linkage rules take effect only after the ACL view permission requirements are met. Without any block linkage rules, the block is displayed by default.


### Control Blocks with Global Variables

Block linkage rules support controlling the display of blocks through global variables. For example, different roles may have view permissions for the same order collection, but the content they can view can be personalized and differentiated. By configuring the fields and actions  displayed in the block according to the role's permissions, flexible data display and action permissions control can be achieved.
#### Customer Service Role

- **Data Scope**: Can only view orders with the status "Pending Shipment" and hide sensitive data such as payment information and discount customer data.
- **Actions**:
  - View Order Details
  - Process Returns/Exchanges
  - Generate Refund Request
- **Viewable Fields**: OrderNumber, OrderDate, OrderStatus, ShippingAddress (excluding sensitive fields like TotalAmount, Discount, Customers, etc.).

  ![20250427141800](https://static-docs.nocobase.com/20250427141800.png)

#### Finance Role

- **Data Scope**: Can only view orders with the status "Paid" or "Refunding" and does not need to view product details or customer information.
- **Actions**:
  - Audit Refund
  - Generate Invoice
- **Viewable Fields**: OrderNumber, OrderDate, OrderStatus, ShippingAddress, TotalAmount (excluding sensitive fields like Items, Customers, etc.).

  ![20250427142420](https://static-docs.nocobase.com/20250427142420.png)

### Control Blocks with Context Variables

Blocks can also be controlled by variables in the context. For example, you can use "current record", "current form", or "current popup record" context variables to dynamically show blocks.

Example: The "Shipping Information" block will only be displayed when the order status is "Shipped".

![20250427143707](https://static-docs.nocobase.com/20250427143707.png)

![20250427143951](https://static-docs.nocobase.com/20250427143951.png)

### Markdown in Blocks

Example: Configure Markdown to display tracking information in the details block.
![20250427150236](https://static-docs.nocobase.com/20250427150236.png)

![20250427150308](https://static-docs.nocobase.com/20250427150308.png)
The Markdown information will only be displayed when the order status is "Shipped".
![20250427150341](https://static-docs.nocobase.com/20250427150341.png)

The effect is as follows:
<video width="100%" height="440" controls>
  <source src="https://static-docs.nocobase.com/20250427150738.mp4" type="video/mp4">
</video>

For more details on linkage rules, refer to [Linkage Rules](/handbook/ui/linkage-rule).
