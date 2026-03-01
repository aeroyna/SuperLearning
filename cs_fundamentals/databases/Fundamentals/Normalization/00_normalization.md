# Normalization

## Overview

Database normalization is the process of organizing data to reduce redundancy and improve data integrity. It involves dividing large tables into smaller, well-structured tables and defining relationships between them.

## Topics Covered

1. **[Functional Dependencies](01_functional_dependencies.md)** - Understanding data dependencies
2. **[First Normal Form](02_first_normal_form.md)** - Atomic values and no repeating groups
3. **[Second Normal Form](03_second_normal_form.md)** - Full functional dependency
4. **[Third Normal Form](04_third_normal_form.md)** - No transitive dependencies
5. **[BCNF and Higher Forms](05_bcnf_and_higher_forms.md)** - Boyce-Codd, 4NF, 5NF
6. **[Denormalization Strategies](06_denormalization_strategies.md)** - When and how to denormalize

## Normal Forms Summary

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         NORMAL FORMS PROGRESSION                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   UNF (Unnormalized) ────▶ 1NF ────▶ 2NF ────▶ 3NF ────▶ BCNF ────▶ 4NF    │
│         │                   │        │        │          │          │       │
│         ▼                   ▼        ▼        ▼          ▼          ▼       │
│   Repeating groups      Atomic   No partial  No       Every      No multi  │
│   nested tables         values   dependency  transitive determinant valued  │
│                                              dependency is candidate dependency│
│                                                         key                  │
│                                                                              │
│   ─────────────────────────────────────────────────────────────────────────  │
│                                                                              │
│   MOST DATABASES AIM FOR 3NF OR BCNF                                        │
│                                                                              │
│   Higher forms (4NF, 5NF) are for special cases with complex dependencies   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Quick Example

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    NORMALIZATION EXAMPLE                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   UNNORMALIZED (Problems: redundancy, update anomalies)                     │
│   ┌───────────────────────────────────────────────────────────────────────┐ │
│   │ OrderID │ Customer │ CustomerAddr │ Products                          │ │
│   │    1    │  Alice   │  123 Main St │ Widget($10), Gadget($20)          │ │
│   │    2    │  Alice   │  123 Main St │ Gadget($20)                       │ │
│   └───────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│   NORMALIZED (3NF)                                                           │
│   ┌─────────────────┐  ┌──────────────────┐  ┌────────────────────────────┐ │
│   │ Customers       │  │ Orders           │  │ OrderItems                 │ │
│   ├─────────────────┤  ├──────────────────┤  ├────────────────────────────┤ │
│   │ CustomerID (PK) │  │ OrderID (PK)     │  │ OrderID (FK)               │ │
│   │ Name            │  │ CustomerID (FK)  │  │ ProductID (FK)             │ │
│   │ Address         │  │ OrderDate        │  │ Quantity                   │ │
│   └─────────────────┘  └──────────────────┘  │ Price                      │ │
│                                              └────────────────────────────┘ │
│   ┌─────────────────┐                                                        │
│   │ Products        │   Benefits:                                           │
│   ├─────────────────┤   • No redundant customer data                        │
│   │ ProductID (PK)  │   • Update address in one place                       │
│   │ Name            │   • Consistent product prices                         │
│   │ Price           │   • Clear relationships                               │
│   └─────────────────┘                                                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Learning Objectives

After completing this section, you will be able to:
- Identify functional dependencies in data
- Apply normalization rules step by step
- Recognize and fix common normalization problems
- Determine the appropriate level of normalization
- Make informed decisions about denormalization
- Balance data integrity with query performance
