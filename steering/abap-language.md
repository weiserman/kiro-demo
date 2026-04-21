# ABAP Language Reference

Comprehensive ABAP development reference covering syntax from 7.40 SP08 through ABAP Cloud. Use this when writing ABAP code, working with internal tables, structures, ABAP SQL, object-oriented programming, RAP, CDS views, EML, string processing, dynamic programming, or unit testing.

Based on [SAP ABAP Cheat Sheets](https://github.com/SAP-samples/abap-cheat-sheets) and **[SAP ABAP Keyword Documentation for 8.16](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm)** — this link is pinned to your project's release; use it as the authoritative source for syntax questions.

> **Before writing code:** Check the `system-info.md` file in the target system's mirror folder (e.g., `BRE/system-info.md`). It contains the SAP_BASIS release and a list of available features. Only use syntax that is supported by that release. If no `system-info.md` exists, query the system first — see the **abap-workflows** steering file for instructions.

---

## Version Compatibility

Features requiring a specific release are noted inline. The table below summarizes key version boundaries.

| Feature | 7.40 SP02 | 7.40 SP05 | 7.40 SP08 | 7.50 | 7.51 | 7.52 | 7.54 |
|---------|:---------:|:---------:|:---------:|:----:|:----:|:----:|:----:|
| Inline declarations `DATA(...)` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Constructor operators (VALUE, NEW, CONV, COND, SWITCH, REF, CAST) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Table expressions `itab[...]` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| String templates `\|...\|` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| ABAP SQL: `@` host variables | | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| ABAP SQL: comma-separated lists | | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| CORRESPONDING operator | | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Table comprehensions (FOR) | | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| LET expressions | | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| REDUCE operator | | | ✓ | ✓ | ✓ | ✓ | ✓ |
| FILTER operator | | | ✓ | ✓ | ✓ | ✓ | ✓ |
| LOOP AT ... GROUP BY | | | ✓ | ✓ | ✓ | ✓ | ✓ |
| CDS views with parameters | | | ✓ | ✓ | ✓ | ✓ | ✓ |
| `FINAL(...)` inline declaration | | | | ✓ | ✓ | ✓ | ✓ |
| Host expressions `@( expr )` | | | | ✓ | ✓ | ✓ | ✓ |
| UNION in SELECT | | | | ✓ | ✓ | ✓ | ✓ |
| IS INSTANCE OF | | | | ✓ | ✓ | ✓ | ✓ |
| Common Table Expressions (WITH) | | | | | ✓ | ✓ | ✓ |
| Enumerated types | | | | | ✓ | ✓ | ✓ |
| Internal tables as data source `FROM @itab` | | | | | | ✓ | ✓ |
| WITH PRIVILEGED ACCESS | | | | | | ✓ | ✓ |
| `utclong` type and functions | | | | | | | ✓ |

On a 7.40 system: replace `FINAL(...)` with `DATA(...)` and avoid 7.50+ features. Most modern ABAP syntax (VALUE, NEW, CONV, inline declarations, table expressions, REDUCE, FILTER, GROUP BY) is available since 7.40 SP08.

---

## Quick Reference

### Data Types and Declarations

```abap
" Elementary types
DATA num TYPE i VALUE 123.
DATA txt TYPE string VALUE `Hello`.
DATA flag TYPE abap_bool VALUE abap_true.

" Inline declarations
DATA(result) = some_method( ).
FINAL(immutable) = `constant value`.              " [7.50+] Use DATA(...) on 7.40

" Structures
DATA: BEGIN OF struc,
        id   TYPE i,
        name TYPE string,
      END OF struc.

" Internal tables
DATA itab TYPE TABLE OF string WITH EMPTY KEY.
DATA sorted_tab TYPE SORTED TABLE OF struct WITH UNIQUE KEY id.
DATA hashed_tab TYPE HASHED TABLE OF struct WITH UNIQUE KEY id.
```

### Internal Tables — Essential Operations

```abap
" Create with VALUE
itab = VALUE #( ( col1 = 1 col2 = `a` )
                ( col1 = 2 col2 = `b` ) ).

" Read operations
DATA(line) = itab[ 1 ].                    " By index
DATA(line2) = itab[ col1 = 1 ].            " By key
READ TABLE itab INTO wa INDEX 1.
READ TABLE itab ASSIGNING FIELD-SYMBOL(<fs>) WITH KEY col1 = 1.

" Modify operations
MODIFY TABLE itab FROM VALUE #( col1 = 1 col2 = `updated` ).
itab[ 1 ]-col2 = `changed`.

" Loop processing
LOOP AT itab ASSIGNING FIELD-SYMBOL(<line>).
  <line>-col2 = to_upper( <line>-col2 ).
ENDLOOP.

" Delete
DELETE itab WHERE col1 > 5.
DELETE TABLE itab FROM VALUE #( col1 = 1 ).
```

### ABAP SQL Essentials

```abap
" SELECT into table
SELECT * FROM dbtab INTO TABLE @DATA(result_tab).

" SELECT with conditions
SELECT carrid, connid, fldate
  FROM zdemo_abap_fli
  WHERE carrid = 'LH'
  INTO TABLE @DATA(flights).

" Aggregate functions
SELECT carrid, COUNT(*) AS cnt, AVG( price ) AS avg_price
  FROM zdemo_abap_fli
  GROUP BY carrid
  INTO TABLE @DATA(stats).

" JOIN operations
SELECT a~carrid, a~connid, b~carrname
  FROM zdemo_abap_fli AS a
  INNER JOIN zdemo_abap_carr AS b ON a~carrid = b~carrid
  INTO TABLE @DATA(joined).

" Modification statements
INSERT dbtab FROM @struc.
UPDATE dbtab FROM @struc.
MODIFY dbtab FROM TABLE @itab.
DELETE FROM dbtab WHERE condition.
```

### Constructor Expressions

```abap
" VALUE — structures and tables
DATA(struc) = VALUE struct_type( comp1 = 1 comp2 = `text` ).
DATA(itab) = VALUE itab_type( ( a = 1 ) ( a = 2 ) ( a = 3 ) ).

" NEW — create instances
DATA(dref) = NEW i( 123 ).
DATA(oref) = NEW zcl_my_class( param = value ).

" CORRESPONDING — structure/table mapping
target = CORRESPONDING #( source ).
target = CORRESPONDING #( source MAPPING target_field = source_field ).

" COND/SWITCH — conditional values
DATA(text) = COND string( WHEN flag = abap_true THEN `Yes` ELSE `No` ).
DATA(result) = SWITCH #( code WHEN 1 THEN `A` WHEN 2 THEN `B` ELSE `X` ).

" CONV — type conversion
DATA(dec) = CONV decfloat34( 1 / 3 ).

" FILTER — table filtering
DATA(filtered) = FILTER #( itab WHERE status = 'A' ).

" REDUCE — aggregation
DATA(sum) = REDUCE i( INIT s = 0 FOR wa IN itab NEXT s = s + wa-amount ).
```

### Object-Oriented ABAP

```abap
" Class definition
CLASS zcl_example DEFINITION PUBLIC FINAL CREATE PUBLIC.
  PUBLIC SECTION.
    METHODS constructor IMPORTING iv_name TYPE string.
    METHODS get_name RETURNING VALUE(rv_name) TYPE string.
    CLASS-METHODS factory RETURNING VALUE(ro_instance) TYPE REF TO zcl_example.
  PRIVATE SECTION.
    DATA mv_name TYPE string.
ENDCLASS.

CLASS zcl_example IMPLEMENTATION.
  METHOD constructor.
    mv_name = iv_name.
  ENDMETHOD.
  METHOD get_name.
    rv_name = mv_name.
  ENDMETHOD.
  METHOD factory.
    ro_instance = NEW #( `Default` ).
  ENDMETHOD.
ENDCLASS.

" Interface implementation
CLASS zcl_impl DEFINITION PUBLIC.
  PUBLIC SECTION.
    INTERFACES zif_my_interface.
ENDCLASS.
```

### Exception Handling

```abap
TRY.
    DATA(result) = risky_operation( ).
  CATCH cx_sy_zerodivide INTO DATA(exc).
    DATA(msg) = exc->get_text( ).
  CATCH cx_root INTO DATA(any_exc).
    " Handle any exception
  CLEANUP.
    " Cleanup code
ENDTRY.

" Raising exceptions
RAISE EXCEPTION TYPE zcx_my_exception
  EXPORTING textid = zcx_my_exception=>error_occurred.

" With COND/SWITCH
DATA(val) = COND #( WHEN valid THEN result
                    ELSE THROW zcx_my_exception( ) ).
```

### String Processing

```abap
" Concatenation
DATA(full) = first && ` ` && last.
txt &&= ` appended`.

" String templates
DATA(msg) = |Name: { name }, Date: { date DATE = ISO }|.

" Functions
DATA(upper) = to_upper( text ).
DATA(len) = strlen( text ).
DATA(found) = find( val = text sub = `search` ).
DATA(replaced) = replace( val = text sub = `old` with = `new` occ = 0 ).
DATA(parts) = segment( val = text index = 2 sep = `,` ).

" FIND/REPLACE statements
FIND ALL OCCURRENCES OF pattern IN text RESULTS DATA(matches).
REPLACE ALL OCCURRENCES OF old IN text WITH new.
```

### Dynamic Programming

```abap
" Field symbols
FIELD-SYMBOLS <fs> TYPE any.
ASSIGN struct-component TO <fs>.
ASSIGN struct-(comp_name) TO <fs>.  " Dynamic component

" Data references
DATA dref TYPE REF TO data.
dref = REF #( variable ).
CREATE DATA dref TYPE (type_name).
dref->* = value.

" RTTI — Get type information
DATA(tdo) = cl_abap_typedescr=>describe_by_data( dobj ).
DATA(components) = CAST cl_abap_structdescr( tdo )->components.

" RTTC — Create types dynamically
DATA(elem_type) = cl_abap_elemdescr=>get_string( ).
CREATE DATA dref TYPE HANDLE elem_type.
```

---

## Common Patterns

### Safe Table Access (Avoid Exceptions)

```abap
" Using VALUE with OPTIONAL
DATA(line) = VALUE #( itab[ key = value ] OPTIONAL ).

" Using VALUE with DEFAULT
DATA(line) = VALUE #( itab[ 1 ] DEFAULT VALUE #( ) ).

" Check before access
IF line_exists( itab[ key = value ] ).
  DATA(line) = itab[ key = value ].
ENDIF.
```

### Functional Method Chaining

```abap
DATA(result) = NEW zcl_builder( )
  ->set_name( `Test` )
  ->set_value( 123 )
  ->build( ).
```

### FOR Iteration Expressions

```abap
" Transform table
DATA(transformed) = VALUE itab_type(
  FOR wa IN source_itab
  ( id = wa-id name = to_upper( wa-name ) ) ).

" With WHERE
DATA(filtered) = VALUE itab_type(
  FOR wa IN source WHERE ( status = 'A' )
  ( wa ) ).

" With INDEX INTO
DATA(numbered) = VALUE itab_type(
  FOR wa IN source INDEX INTO idx
  ( line_no = idx data = wa ) ).
```

### ABAP Cloud Compatibility

```abap
" Use released APIs only
DATA(uuid) = cl_system_uuid=>create_uuid_x16_static( ).

" Output in cloud (if_oo_adt_classrun)
out->write( result ).

" Avoid in ABAP Cloud: sy-datum, sy-uzeit, DESCRIBE TABLE, WRITE, MOVE...TO
```

### ABAP 7.40 Compatibility

When targeting 7.40 systems, replace 7.50+ syntax:

```abap
" Instead of FINAL (7.50+):
FINAL(value) = `constant`.              " 7.50+
DATA(value) = `constant`.               " 7.40 compatible

" Instead of host expressions (7.50+):
SELECT * FROM dbtab WHERE col = @( lv_val ).   " 7.50+
SELECT * FROM dbtab WHERE col = @lv_val.        " 7.40 compatible

" Instead of UNION (7.50+) — use two SELECTs and combine in ABAP:
SELECT a FROM tab1 INTO TABLE @DATA(r1).
SELECT a FROM tab2 INTO TABLE @DATA(r2).
DATA(combined) = VALUE itab_type( FOR l1 IN r1 ( l1 )
                                  FOR l2 IN r2 ( l2 ) ).

" Instead of IS INSTANCE OF (7.50+) — use CAST with exception handling:
TRY.
    DATA(lo) = CAST zcl_my_class( oref ).  " 7.40+
  CATCH cx_sy_move_cast_error.
    " oref is not compatible with zcl_my_class
ENDTRY.
```

---

## Error Catalog

| Exception | Cause | Solution |
|-----------|-------|----------|
| `CX_SY_ITAB_LINE_NOT_FOUND` | Table expression access to non-existent line | Use `OPTIONAL`, `DEFAULT`, or check with `line_exists()` |
| `CX_SY_ZERODIVIDE` | Division by zero | Check divisor before operation |
| `CX_SY_RANGE_OUT_OF_BOUNDS` | Invalid substring access or array bounds | Validate offset and length before access |
| `CX_SY_CONVERSION_NO_NUMBER` | String cannot be converted to number | Validate input format before conversion |
| `CX_SY_REF_IS_INITIAL` | Dereferencing unbound reference | Check `IS BOUND` before dereferencing |

---

## Performance Tips

- Use SORTED/HASHED tables for frequent key access
- Prefer field symbols over work areas in loops for modification
- Use `PACKAGE SIZE` for large SELECT results
- Avoid SELECT in loops — use `FOR ALL ENTRIES` or JOINs
- Use secondary keys for different access patterns
- Minimize `CORRESPONDING` calls — explicit assignments are faster

---

## Source Documentation

Content based on and attributed to:
- [SAP ABAP Cheat Sheets](https://github.com/SAP-samples/abap-cheat-sheets) (Apache 2.0)
- [SAP Help — ABAP (latest)](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm)
- [SAP Help — ABAP 7.40](https://help.sap.com/doc/abapdocu_740_index_htm/7.40/en-US/index.htm)
- [ABAP Release News](https://github.com/SAP-samples/abap-cheat-sheets/blob/main/33_ABAP_Release_News.md)
- [secondsky/sap-skills](https://github.com/secondsky/sap-skills) (GPL-3.0)

Content was rephrased and restructured for compliance with licensing restrictions.
