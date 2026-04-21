---
inclusion: manual
name: abap-samples
description: "Curated code samples with version frontmatter: RAP/EML, CDS view entities, unit testing, ABAP Cloud development, plus an index of additional reference topics."
---

# ABAP Code Samples

Curated code samples organized by topic. Each section includes a version tag indicating the minimum SAP_BASIS release required. Cross-reference with `system-info.md` in your mirror folder to confirm compatibility before using.

> Before using any sample, check your system's `system-info.md` file. If a sample requires a higher release than your system, see the **abap-language** steering file for downgrade patterns.

Source: Samples derived from [SAP ABAP Cheat Sheets](https://github.com/SAP-samples/abap-cheat-sheets) (Apache 2.0) and [secondsky/sap-skills](https://github.com/secondsky/sap-skills) (GPL-3.0). Content rephrased for compliance with licensing restrictions.

---

## RAP and EML (Entity Manipulation Language)

<!--
abap_version: "7.50+"
topics: [rap, eml, behavior-definition, managed, unmanaged]
cloud_compatible: true
-->

### Behavior Definition (BDEF)

```cds
managed implementation in class zbp_demo_rap_m unique;
strict ( 2 );

define behavior for ZDemo_RAP_Root alias Root
persistent table zdemo_rap_tab
lock master
authorization master ( instance )
{
  create;
  update;
  delete;

  field ( readonly ) key_field;
  field ( mandatory ) required_field;

  association _Child { create; }

  action someAction;

  determination det1 on save { create; }
  validation val1 on save { create; update; }
}
```

### EML Deep Create (Parent + Child)

```abap
MODIFY ENTITIES OF root_entity
  ENTITY root
  CREATE FIELDS ( key_field field1 )
  WITH VALUE #( (
    %cid = 'cid_root'
    key_field = 1
    field1 = 'Parent' ) )

  CREATE BY \_child
  FIELDS ( child_key child_field )
  WITH VALUE #( (
    %cid_ref = 'cid_root'
    %target = VALUE #(
      ( %cid = 'cid_child1'
        child_key = 10
        child_field = 'Child 1' )
      ( %cid = 'cid_child2'
        child_key = 20
        child_field = 'Child 2' ) ) ) )

  MAPPED DATA(mapped)
  FAILED DATA(failed)
  REPORTED DATA(reported).
```

### RAP Handler Class (Local Implementation)

```abap
CLASS lhc_root DEFINITION INHERITING FROM cl_abap_behavior_handler.
  PRIVATE SECTION.
    METHODS get_instance_authorizations FOR INSTANCE AUTHORIZATION
      IMPORTING keys REQUEST requested_authorizations
      FOR root RESULT result.

    METHODS create FOR MODIFY
      IMPORTING entities FOR CREATE root.

    METHODS update FOR MODIFY
      IMPORTING entities FOR UPDATE root.

    METHODS delete FOR MODIFY
      IMPORTING keys FOR DELETE root.

    METHODS some_action FOR MODIFY
      IMPORTING keys FOR ACTION root~some_action.

    METHODS val1 FOR VALIDATE ON SAVE
      IMPORTING keys FOR root~val1.
ENDCLASS.
```

---

## CDS View Entities

<!--
abap_version: "7.50+ (view entities), 7.40+ (classic DEFINE VIEW)"
topics: [cds, view-entity, associations, compositions, aggregates]
cloud_compatible: true
-->

### Basic View Entity with Associations

```cds
define view entity ZDemo_Associations
  as select from zdemo_header as Header
  association [1..*] to zdemo_item as _Items
    on Header.header_id = _Items.header_id
  association [0..1] to zdemo_status as _Status
    on Header.status = _Status.status_code
{
  key Header.header_id,
      Header.description,
      Header.status,
      _Items,
      _Status
}
```

### Composition Hierarchy (Root + Child)

```cds
// Root entity
define root view entity ZDemo_Root
  as select from zdemo_root
  composition [0..*] of ZDemo_Child as _Children
{
  key root_id,
      description,
      _Children
}

// Child entity
define view entity ZDemo_Child
  as select from zdemo_child
  association to parent ZDemo_Root as _Root
    on $projection.root_id = _Root.root_id
{
  key root_id,
  key child_id,
      description,
      _Root
}
```

### Aggregates, Expressions, and Built-in Functions

```cds
define view entity ZDemo_Aggregates
  as select from zdemo_table
{
  key category,
  count(*) as TotalCount,
  sum( amount ) as TotalAmount,
  avg( price ) as AveragePrice,
  min( date_field ) as FirstDate,
  max( date_field ) as LastDate,

  case status
    when 'A' then 'Active'
    when 'I' then 'Inactive'
    else 'Unknown'
  end as StatusText,

  coalesce( nullable_field, 'Default' ) as FieldWithDefault
}
group by category
```

---

## ABAP Unit Testing

<!--
abap_version: "7.40+ (core), 7.50+ (TEST-SEAM), 7.52+ (SQL test doubles)"
topics: [unit-test, test-double, sql-test-environment, assertions]
cloud_compatible: true
-->

### Test Class with Given-When-Then

```abap
CLASS ltc_test_class DEFINITION
  FOR TESTING
  RISK LEVEL HARMLESS
  DURATION SHORT.

  PRIVATE SECTION.
    DATA cut TYPE REF TO zcl_class_under_test.

    CLASS-METHODS class_setup.
    METHODS setup.
    METHODS teardown.
    METHODS test_method_1 FOR TESTING.
ENDCLASS.

CLASS ltc_test_class IMPLEMENTATION.
  METHOD setup.
    cut = NEW #( ).
  ENDMETHOD.

  METHOD test_method_1.
    " Given
    DATA(input) = 5.
    " When
    DATA(result) = cut->multiply_by_two( input ).
    " Then
    cl_abap_unit_assert=>assert_equals(
      act = result
      exp = 10 ).
  ENDMETHOD.
ENDCLASS.
```

### SQL Test Double Framework

```abap
" Requires: 7.52+ (cl_osql_test_environment)

CLASS ltc_sql_test DEFINITION FOR TESTING
  RISK LEVEL HARMLESS DURATION SHORT.

  PRIVATE SECTION.
    CLASS-DATA sql_env TYPE REF TO if_osql_test_environment.

    CLASS-METHODS class_setup.
    CLASS-METHODS class_teardown.
    METHODS test_select FOR TESTING.
ENDCLASS.

CLASS ltc_sql_test IMPLEMENTATION.
  METHOD class_setup.
    sql_env = cl_osql_test_environment=>create(
      VALUE #( ( 'ZDEMO_TABLE' ) ) ).
  ENDMETHOD.

  METHOD class_teardown.
    sql_env->destroy( ).
  ENDMETHOD.

  METHOD test_select.
    sql_env->insert_test_data(
      VALUE zdemo_table( ( id = 1 name = 'Test' ) ) ).

    DATA(result) = cut->read_data( ).

    cl_abap_unit_assert=>assert_equals(
      act = lines( result )
      exp = 1 ).

    sql_env->clear_doubles( ).
  ENDMETHOD.
ENDCLASS.
```

### Manual Test Double with Constructor Injection

```abap
" Works on: 7.40+ (uses standard OO patterns)

CLASS ltd_database_access DEFINITION FOR TESTING.
  PUBLIC SECTION.
    INTERFACES zif_database_access.
    DATA returned_data TYPE zdemo_table.
ENDCLASS.

CLASS ltd_database_access IMPLEMENTATION.
  METHOD zif_database_access~get_data.
    result = me->returned_data.
  ENDMETHOD.
ENDCLASS.

" Usage in test
METHOD test_with_double.
  DATA(double) = NEW ltd_database_access( ).
  double->returned_data = VALUE #( ( id = 1 name = 'Test' ) ).
  cut = NEW zcl_processor( database = double ).

  DATA(result) = cut->process( ).

  cl_abap_unit_assert=>assert_equals( act = result exp = 'Test' ).
ENDMETHOD.
```

---

## ABAP Cloud Development

<!--
abap_version: "Cloud"
topics: [abap-cloud, released-apis, xco, classrun, migration]
cloud_compatible: true
-->

### Console Output (if_oo_adt_classrun)

```abap
" Replaces: WRITE statements (not available in ABAP Cloud)

CLASS zcl_demo DEFINITION PUBLIC FINAL CREATE PUBLIC.
  PUBLIC SECTION.
    INTERFACES if_oo_adt_classrun.
ENDCLASS.

CLASS zcl_demo IMPLEMENTATION.
  METHOD if_oo_adt_classrun~main.
    out->write( 'Hello from ABAP Cloud!' ).
    out->write( data_object ).
    out->write( itab ).
  ENDMETHOD.
ENDCLASS.
```

### XCO Library for Date/Time

```abap
" Replaces: sy-datum, sy-uzeit (not available in ABAP Cloud)

" Current date
DATA(date) = xco_cp=>sy->date( )->as( xco_cp_time=>format->iso_8601_extended )->value.

" Current time
DATA(time) = xco_cp=>sy->time( )->as( xco_cp_time=>format->iso_8601_extended )->value.

" Date calculations
DATA(tomorrow) = xco_cp=>sy->date( )->add( iv_day = 1 )->value.
DATA(next_month) = xco_cp=>sy->date( )->add( iv_month = 1 )->value.
```

### Classic-to-Cloud Migration Patterns

```abap
" Classic (NOT allowed in ABAP Cloud)     →  Cloud replacement
" ─────────────────────────────────────────────────────────────
" MOVE source TO target.                  →  target = source.
" DESCRIBE TABLE itab LINES count.        →  DATA(count) = lines( itab ).
" GET REFERENCE OF var INTO dref.         →  dref = REF #( var ).
" DATA(date) = sy-datum.                  →  Use xco_cp=>sy->date( )->value.
" WRITE 'text'.                           →  out->write( 'text' ).
" CALL FUNCTION 'xxx'.                    →  Use released class APIs instead.
```

---

## OO Design Patterns

<!--
abap_version: "7.40+"
topics: [factory, singleton, strategy, design-patterns, oo]
cloud_compatible: true
-->

### Factory Method

```abap
CLASS lcl_hello_factory DEFINITION FINAL CREATE PRIVATE.
  PUBLIC SECTION.
    CLASS-METHODS create_hello
      IMPORTING language TYPE lif_hello=>enum_langu
      RETURNING VALUE(hello) TYPE REF TO lif_hello.
ENDCLASS.

CLASS lcl_hello_factory IMPLEMENTATION.
  METHOD create_hello.
    hello = SWITCH #( language
      WHEN lif_hello=>en THEN NEW lcl_en( )
      WHEN lif_hello=>fr THEN NEW lcl_fr( )
      ELSE NEW lcl_en( ) ).
  ENDMETHOD.
ENDCLASS.
```

### Singleton

```abap
CLASS zcl_singleton DEFINITION
  PUBLIC FINAL CREATE PRIVATE.

  PUBLIC SECTION.
    CLASS-METHODS get_instance
      RETURNING VALUE(ro_instance) TYPE REF TO zcl_singleton.
    METHODS do_something.

  PRIVATE SECTION.
    CLASS-DATA go_instance TYPE REF TO zcl_singleton.
ENDCLASS.

CLASS zcl_singleton IMPLEMENTATION.
  METHOD get_instance.
    IF go_instance IS NOT BOUND.
      go_instance = NEW #( ).
    ENDIF.
    ro_instance = go_instance.
  ENDMETHOD.
ENDCLASS.
```

### Strategy Pattern

```abap
CLASS lcl_sorter DEFINITION.
  PUBLIC SECTION.
    METHODS constructor
      IMPORTING io_strategy TYPE REF TO lif_sort_strategy.
    METHODS set_strategy
      IMPORTING io_strategy TYPE REF TO lif_sort_strategy.
    METHODS execute_sort
      CHANGING ct_data TYPE STANDARD TABLE.
  PRIVATE SECTION.
    DATA mo_strategy TYPE REF TO lif_sort_strategy.
ENDCLASS.

CLASS lcl_sorter IMPLEMENTATION.
  METHOD constructor.
    mo_strategy = io_strategy.
  ENDMETHOD.
  METHOD set_strategy.
    mo_strategy = io_strategy.
  ENDMETHOD.
  METHOD execute_sort.
    mo_strategy->sort( CHANGING ct_data = ct_data ).
  ENDMETHOD.
ENDCLASS.

" Usage — swap strategies at runtime
DATA(sorter) = NEW lcl_sorter( NEW lcl_bubble_sort( ) ).
sorter->execute_sort( CHANGING ct_data = my_table ).
sorter->set_strategy( NEW lcl_quick_sort( ) ).
sorter->execute_sort( CHANGING ct_data = my_table ).
```

---

## Performance Optimization

<!--
abap_version: "7.40+ (modern syntax), any (core SQL patterns)"
topics: [performance, sql, internal-tables, buffering, for-all-entries]
cloud_compatible: true
-->

### Block Operations vs. Line-by-Line

```abap
" GOOD: Single database operation
INSERT dbtab FROM TABLE @itab.
UPDATE dbtab FROM TABLE @itab.
DELETE dbtab FROM TABLE @itab.

" BAD: Loop with individual operations (avoid!)
LOOP AT itab INTO wa.
  INSERT dbtab FROM @wa.
ENDLOOP.

" GOOD: Block append for internal tables
APPEND LINES OF source_tab TO target_tab.
```

### FOR ALL ENTRIES (Critical: Empty-Table Guard)

```abap
" CRITICAL: Always check if driver table is empty!
" An empty table causes a FULL TABLE SCAN.

IF itab IS NOT INITIAL.
  SELECT * FROM dbtab
    FOR ALL ENTRIES IN @itab
    WHERE key_field = @itab-key
    INTO TABLE @DATA(result).
ENDIF.
```

### Internal Table Type Selection

```abap
" Standard — small datasets, sequential processing
DATA std_tab TYPE STANDARD TABLE OF struct WITH EMPTY KEY.

" Sorted — frequent key access, range queries (O(log n))
DATA sorted_tab TYPE SORTED TABLE OF struct WITH UNIQUE KEY id.

" Hashed — large datasets, key-only access (O(1))
DATA hashed_tab TYPE HASHED TABLE OF struct WITH UNIQUE KEY id.

" OPTIMAL: Full primary key access
READ TABLE hashed_tab WITH TABLE KEY id = 123 INTO wa.

" Use field symbols for loop modification (no copy overhead)
LOOP AT itab ASSIGNING FIELD-SYMBOL(<line>).
  <line>-field = new_value.
ENDLOOP.

" Check existence without data transfer
READ TABLE itab TRANSPORTING NO FIELDS WITH TABLE KEY id = 123.
```

---

## Additional Reference Files

The full set of 28 reference files covering all ABAP topics is available at:
[github.com/secondsky/sap-skills/tree/main/plugins/sap-abap/skills/sap-abap/references](https://github.com/secondsky/sap-skills/tree/main/plugins/sap-abap/skills/sap-abap/references)

| Reference | Topics | Min Version |
|-----------|--------|:-----------:|
| `internal-tables.md` | Table types, operations, comprehensions, GROUP BY | 7.40+ |
| `abap-sql.md` | SELECT, JOIN, subqueries, CTE, aggregates | 7.40 SP05+ |
| `constructor-expressions.md` | VALUE, NEW, CONV, COND, SWITCH, REDUCE, FILTER | 7.40 SP02+ |
| `string-processing.md` | Templates, functions, regex, concatenation | 7.40+ |
| `object-orientation.md` | Classes, interfaces, inheritance, polymorphism | 7.40+ |
| `dynamic-programming.md` | Field symbols, data references, RTTI/RTTC | 7.40+ |
| `exceptions.md` | TRY/CATCH, class-based exceptions, RESUMABLE | 7.40+ |
| `rap-eml.md` | RAP behavior definitions, EML, handler classes | 7.50+ |
| `cds-views.md` | View entities, associations, compositions, annotations | 7.50+ |
| `unit-testing.md` | ABAP Unit, test doubles, SQL test environment | 7.40+ |
| `cloud-development.md` | ABAP Cloud, released APIs, XCO, migration | Cloud |
| `design-patterns.md` | Factory, singleton, strategy, observer, builder | 7.40+ |
| `performance.md` | SQL optimization, table types, buffering, profiling | Any |
| `abap-dictionary.md` | Domains, data elements, tables, structures | Any |
| `authorization.md` | Authority checks, DCL, access control | 7.40+ |
| `date-time.md` | Date/time types, calculations, timestamps, utclong | 7.40+ |
| `builtin-functions.md` | String, numeric, table, type functions | 7.40+ |
| `program-flow.md` | IF/CASE, loops, COND/SWITCH expressions | 7.40+ |
| `where-conditions.md` | WHERE clause patterns, ranges, dynamic conditions | 7.40+ |
| `sql-hierarchies.md` | Hierarchical queries, parent-child, CTE trees | 7.51+ |
| `numeric-operations.md` | Arithmetic, rounding, currency, decimal float | Any |
| `bits-bytes.md` | Hex, byte operations, bit manipulation | Any |
| `xml-json.md` | Serialization, transformation, XSLT, ST | 7.40+ |
| `sap-luw.md` | Logical unit of work, COMMIT, update task | Any |
| `amdp.md` | ABAP-managed database procedures, CDS table functions | 7.40 SP08+ |
| `released-classes.md` | Key released APIs for ABAP Cloud | Cloud |
| `generative-ai.md` | AI SDK, LLM integration, prompt engineering | Cloud |
| `table-grouping.md` | GROUP BY in LOOP, member access, aggregation | 7.40 SP08+ |
