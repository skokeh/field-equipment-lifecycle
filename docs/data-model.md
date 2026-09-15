# NorthField Services – Field Equipment Lifecycle Management

## 1. Overview

The Field Equipment Lifecycle Management solution is a Microsoft Power Platform solution designed for NorthField Services.

The system manages field equipment deployed at customer sites and supports:

- Equipment lifecycle management
- Customer and site management
- Technician management
- Equipment inspections
- Maintenance orders
- Spare parts usage
- Basic inventory history
- Equipment maintenance history

The solution is designed using Microsoft Dataverse and will be implemented as a Model-driven Power App.

---

## 2. Data Model

### Core Tables

| Table | Type | Purpose |
|---|---|---|
| Account | Standard Dataverse Table | Represents customers |
| Site | Custom Table | Represents customer locations |
| Equipment | Custom Table | Represents physical equipment installed at sites |
| Technician | Custom Table | Represents field technicians |
| Inspection | Custom Table | Records equipment inspections |
| Maintenance Order | Custom Table | Manages maintenance activities |
| Maintenance Order Technician | Custom Table | Associates technicians with maintenance orders |
| Spare Part | Custom Table | Manages spare parts and stock |
| Maintenance Order Spare Part | Custom Table | Records spare parts used during maintenance |

---

# 3. Entity Relationships

## 3.1 Account → Site

**Relationship:** 1:N

One customer can have multiple sites.

```text
Account
   |
   | 1:N
   |
   +---- Site

   Example

A customer such as "ABC Manufacturing" may have:

Brussels Production Site
Antwerp Warehouse
Ghent Distribution Center

Each site belongs to one customer.

3.2 Site → Equipment

Relationship: 1:N

One site can contain multiple pieces of equipment.

Site
   |
   | 1:N
   |
   +---- Equipment

Each equipment record belongs to one site.

3.3 Equipment → Inspection

Relationship: 1:N

One equipment item can have multiple inspections over its lifetime.

Equipment
   |
   | 1:N
   |
   +---- Inspection

This allows the system to maintain the inspection history of each equipment item.

3.4 Inspection ↔ Technician

Relationship: N:N

An inspection may involve multiple technicians, and a technician may participate in multiple inspections.

Inspection
     |
     | N:N
     |
Technician

No additional information is currently required for the relationship.

The system only needs to know which technicians participated in an inspection.

3.5 Equipment → Maintenance Order

Relationship: 1:N

One equipment item can have multiple maintenance orders throughout its lifecycle.

Equipment
   |
   | 1:N
   |
   +---- Maintenance Order

This relationship is important for maintaining the complete maintenance history of an equipment item.

3.6 Maintenance Order ↔ Technician

Relationship: N:N

A maintenance order can be assigned to multiple technicians.

A technician can work on multiple maintenance orders.

Instead of using a native many-to-many relationship, an explicit junction table is used:

Maintenance Order
       |
       | 1:N
       |
       +---- Maintenance Order Technician
                    |
                    | N:1
                    |
                Technician

The junction table currently contains only the relationship between the two records.

Additional attributes such as technician role or working hours may be added in a future version if required.

3.7 Maintenance Order ↔ Spare Part

Relationship: N:N

A maintenance order can use multiple spare parts.

A spare part can be used in multiple maintenance orders.

Because the relationship contains additional business information, an explicit junction table is required:

Maintenance Order
       |
       | 1:N
       |
       +---- Maintenance Order Spare Part
                    |
                    | N:1
                    |
                Spare Part

The junction table records the spare part usage and basic stock history.

4. Table Definitions
4.1 Account

Type: Standard Dataverse Table

The standard Dataverse Account table represents customers.

The standard Account table will be reused instead of creating a custom Customer table.

Relevant standard fields may include:

Account Name
Main Phone
Email
Address
Website

Additional fields will only be added if a real business requirement exists.

4.2 Site

Represents a physical customer location.

Fields
Field	Type	Required	Description
Site Name	Text	Yes	Name of the site
Customer	Lookup → Account	Yes	Customer owning the site
Address	Text	Yes	Site address
City	Text	Yes	City
Country	Choice/Text	Yes	Country
Active	Yes/No	Yes	Indicates whether the site is active
Relationship
Account 1 ---- N Site
4.3 Equipment

Represents physical equipment installed at a customer site.

Fields
Field	Type	Required	Description
Equipment Number	Text	Yes	Unique equipment identifier
Name	Text	Yes	Equipment name
Type	Choice	Yes	Equipment category
Manufacturer	Text	No	Equipment manufacturer
Model	Text	No	Manufacturer model
Serial Number	Text	Yes	Manufacturer serial number
Installation Date	Date	No	Date equipment was installed
Operational Status	Choice	Yes	Current operational state
Condition	Choice	Yes	Current physical condition
Site	Lookup → Site	Yes	Site where equipment is installed
Operational Status

Initial values:

Operational
Under Maintenance
Out of Service
Retired
Condition

Initial values:

Excellent
Good
Fair
Poor
Critical
Relationship
Site 1 ---- N Equipment
4.4 Technician

Represents technicians who perform inspections and maintenance activities.

Fields
Field	Type	Required	Description
Technician Name	Text	Yes	Technician name
Employee Number	Text	Yes	Internal employee identifier
Email	Email	Yes	Technician email
Phone	Phone	No	Technician phone number
Specialization	Choice/Text	No	Technical specialization
Active	Yes/No	Yes	Indicates whether the technician is active
4.5 Inspection

Represents an inspection performed on equipment.

Fields
Field	Type	Required	Description
Inspection Number	Auto Number	Yes	Unique inspection identifier
Equipment	Lookup → Equipment	Yes	Inspected equipment
Inspection Date	Date and Time	Yes	Date and time of inspection
Inspection Type	Choice	Yes	Type of inspection
Result	Choice	Yes	Inspection result
Notes	Multiple Lines of Text	No	Inspection notes
Result Choices
Passed
Passed with Warning
Failed
Relationship
Equipment 1 ---- N Inspection
Technician Relationship
Inspection N ---- N Technician

An inspection can involve multiple technicians.

4.6 Maintenance Order

Represents a maintenance activity performed on equipment.

Fields
Field	Type	Required	Description
Maintenance Order Number	Auto Number	Yes	Unique maintenance order identifier
Equipment	Lookup → Equipment	Yes	Equipment requiring maintenance
Requested By	Lookup	Yes	Person who requested the maintenance
Priority	Choice	Yes	Maintenance priority
Description	Multiple Lines of Text	Yes	Description of the required work
Status	Status / Choice	Yes	Current maintenance order status
Created Date	Date and Time	Yes	Creation date
Planned Date	Date and Time	No	Planned maintenance date
Completed Date	Date and Time	No	Actual completion date
Completion Notes	Multiple Lines of Text	No	Description of completed work
Priority Choices
Low
Normal
High
Critical
Initial Status Values
New
Assigned
In Progress
On Hold
Completed
Cancelled
Relationship
Equipment 1 ---- N Maintenance Order
Technician Relationship
Maintenance Order
        |
        | 1:N
        |
Maintenance Order Technician
        |
        | N:1
        |
Technician
4.7 Maintenance Order Technician

Junction table used to associate multiple technicians with a maintenance order.

Fields
Field	Type	Required	Description
Maintenance Order	Lookup → Maintenance Order	Yes	Related maintenance order
Technician	Lookup → Technician	Yes	Assigned technician
Relationship
Maintenance Order 1 ---- N Maintenance Order Technician N ---- 1 Technician

No additional relationship attributes are required in version 1.0.

4.8 Spare Part

Represents a spare part used during maintenance activities.

Fields
Field	Type	Required	Description
Part Number	Text	Yes	Unique part identifier
Part Name	Text	Yes	Name of the spare part
Unit Price	Currency	Yes	Price per unit
Stock Quantity	Decimal/Whole Number	Yes	Current available stock
Minimum Stock Level	Decimal/Whole Number	Yes	Minimum acceptable stock level
Active	Yes/No	Yes	Indicates whether the part is active
4.9 Maintenance Order Spare Part

Junction table used to record spare parts consumed during maintenance.

This table is intentionally used instead of creating a separate inventory transaction or stock ledger table.

Fields
Field	Type	Required	Description
Maintenance Order	Lookup → Maintenance Order	Yes	Related maintenance order
Spare Part	Lookup → Spare Part	Yes	Used spare part
Installed Date	Date and Time	Yes	Date/time when the part was installed
Quantity Used	Decimal/Whole Number	Yes	Quantity consumed
Quantity Before	Decimal/Whole Number	Yes	Stock quantity before usage
Quantity After	Decimal/Whole Number	Yes	Stock quantity after usage
Example

If a maintenance order uses 3 filters:

Quantity Before = 30
Quantity Used   = 3
Quantity After  = 27

A later maintenance order may record:

Quantity Before = 27
Quantity Used   = 3
Quantity After  = 24

This provides a basic stock usage history without introducing a separate inventory ledger.

5. Business Rules

The following business rules are part of version 1.0.

BR-01 – Retired Equipment

An inspection cannot be created for equipment whose Operational Status is:

Retired
BR-02 – Critical Maintenance Review

A maintenance order with:

Priority = Critical

requires manager review before work can proceed.

The exact implementation mechanism will be defined during the automation/security phase.

BR-03 – Maintenance Order Completion

A maintenance order cannot be marked as Completed unless:

Completed Date is populated
Completion Notes are populated
At least one technician is assigned
BR-04 – Low Stock Alert

When:

Stock Quantity < Minimum Stock Level

the system should generate a low-stock alert.

The automation mechanism will be implemented later using Power Automate.

6. Equipment History

The system should provide a complete history for each equipment item.

The history will be derived from related records:

Equipment
   |
   +---- Inspections
   |
   +---- Maintenance Orders
              |
              +---- Technicians
              |
              +---- Spare Parts Used

This allows users to review:

Previous inspections
Inspection results
Maintenance activities
Assigned technicians
Spare parts consumed
Spare part quantities
Maintenance completion information

A separate Equipment History table is not required for version 1.0.

7. Design Principles

The solution follows these principles:

Keep the data model simple

Only create a custom table when the business requirement justifies it.

Reuse standard Dataverse tables

The standard Account table is used for customers instead of creating a duplicate custom Customer table.

Use Choices for simple enumerations

Values such as Priority and Inspection Result are implemented as Dataverse Choices rather than separate tables.

Use junction tables when relationships contain business data

Maintenance Order Technician and Maintenance Order Spare Part are explicit relationship tables.

Avoid unnecessary inventory complexity

A dedicated inventory transaction/ledger table is intentionally excluded from version 1.0.

Design for future extensibility

The model should allow additional functionality to be introduced later without redesigning the core business entities.

8. Logical Data Model

High-level relationship diagram:

                         ┌───────────────┐
                         │    Account    │
                         │   Customer    │
                         └───────┬───────┘
                                 │
                                1:N
                                 │
                         ┌───────▼───────┐
                         │      Site     │
                         └───────┬───────┘
                                 │
                                1:N
                                 │
                         ┌───────▼───────┐
                         │   Equipment   │
                         └───┬───────┬───┘
                             │       │
                            1:N     1:N
                             │       │
                    ┌────────▼─┐   ┌─▼─────────────────┐
                    │Inspection│   │ Maintenance Order │
                    └────┬─────┘   └───────┬───────────┘
                         │                  │
                        N:N                1:N
                         │                  │
                    ┌────▼──────┐   ┌──────▼────────────────────┐
                    │ Technician│   │Maintenance Order Technician│
                    └───────────┘   └──────────┬─────────────────┘
                                               │
                                               │ N:1
                                               │
                                         ┌─────▼─────┐
                                         │ Technician│
                                         └───────────┘


                         Maintenance Order
                                │
                               1:N
                                │
                ┌───────────────▼─────────────────┐
                │ Maintenance Order Spare Part    │
                └───────────────┬─────────────────┘
                                │
                               N:1
                                │
                         ┌──────▼──────┐
                         │ Spare Part  │
                         └─────────────┘
9. Version

Data Model Version: 1.0

Status: Initial design

Platform: Microsoft Power Platform / Microsoft Dataverse

Application Type: Model-driven App

Solution: Field Equipment Lifecycle Management

Business: NorthField Services
