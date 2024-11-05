📓 ABAP NEW SYNTAX
=============

## 🔎 Contents
1. [Class Construct Operator](#1-class-construct-operator)
2. [MOVE-CORRESPONDING](#2-move-corresponding)
3. [Alpha Conversion](#3-alpha-conversion)
4. [Concatenate String](#4-concatenate-string)
5. [Create & Assign to ITAB](#5-create--assign-to-itab)
6. [Simple Read Table](#6-simple-read-table)
7. [Filter ITAB with condition](#7-filter-itab-with-condition)
---

## 1. Class Construct Operator

#### Define Class

```
CLASS person DEFINITION.
PUBLIC SECTION.
    DATA: name(5) TYPE c,
        age     TYPE i.
    METHODS constructor IMPORTING i_init_name TYPE char5.
    METHODS eat IMPORTING p_food TYPE char20.
    METHODS talk IMPORTING p_friend TYPE char5.
    METHODS sleep.
ENDCLASS.

CLASS person IMPLEMENTATION.
    METHOD constructor.
        name = i_init_name.
    ENDMETHOD.
    METHOD eat.
        WRITE:/ name, ' is eating ', p_food, '.'.
    ENDMETHOD.
    METHOD sleep.
        WRITE:/ name, ' is sleeping.'.
    ENDMETHOD.
ENDCLASS.
```
#### Create Instance
- Old Syntax

```
CREATE OBJECT p1
    EXPORTING i_init_name = 'SH'.
```
- New Syntax

```
DATA: p1 TYPE REF TO person.

p1 = NEW #( i_init_name = 'SH' ).
DATA(p2) = New person( i_init_name = 'SH2' ).

p1->eat( 'Apple' ).
p2->sleep( ).
```

## 2. MOVE-CORRESPONDING
#### Old Syntax
```
DATA: is_vbak TYPE vbak,
    gs_vbak TYPE vbak.

MOVE-CORRESPONDING is_vbak TO gs_vbak.
```
#### New Syntax
```
DATA(gs_vbak) = CORRESPONDING #( is_vbak ).
```
## 3. Alpha Conversion
- <span >IN = leading Zero / OUT = remove leading Zero</span>
#### Old Syntax
```
CALL FUNCTION 'CONVERSION_EXIT_ALPHA_INPUT/OUTPUT'
EXPORTING
    input         = iv_input.
IMPORTING
    output        = iv_output.
```
#### New Syntax
```
iv_output = |{ iv_input ALPHA = IN/OUT }|.
```
## 4. Concatenate String
#### Old Syntax
```
CONCATENATE iv_input iv_input2 '!' INTO iv_output [SEPARATED BY {str} / RESPECTING BLANKS].
```

#### New Syntax
```
* Simple Ver
iv_output = iv_input && iv_input2 && '!'.

* Normal Ver
iv_output = |{ iv_input } { iv_input2 }!|.

* Conditional Ver
DATA(iv_cond) = 10.
iv_output = |{(iv_cond = 10) ? 'Y' : 'N' }|.
```
- In new syntax, Quotes(') is recongized as string.  
```
iv_output = | 'AMKOR' |.
WRITE: 'RESULT : ', iv_output.
* result
RESULT : 'AMKOR'
```
## 5. Create & Assign to ITAB

#### Old Syntax
```
TYPES ty_tab TYPE STANDARD TABLE OF i WITH DEFAULT KEY.
DATA it_tab TYPE ty_tab.
APPEND: 10 TO it_tab,
        20 TO it_tab,
        30 TO it_tab.
```
#### New Syntax
```
DATA(it_tab) = VALUE ty_tab( ( 10 ) ( 20 ) ( 30 ) ).
```

## 6. Simple Read table
#### Old Syntax
```
READ TABLE it_tab INTO DATA(is_tab)
                  WITH KEY iv_cond  = 'A'
                           iv_cond2 = 'B'.
```
#### New Syntax
```
DATA(is_tab) = it_tab[ iv_cond = 'A' iv_cond2 = 'B' ].
```
- If data doesn't exist, it will raise **ITAB_LINE_NOT_FOUND** Exception
- Therefore, You should use **TRY-CATCH** or **VALUE #( itab[cond] OPTIONAL )**.
    ```
    DATA(is_tab) = VALUE #( it_tab[ iv_cond = 'A' ] OPTIONAL ).
    ```
- Additionally, The New Syntax dosen't support **BINARY SEARCH**. Instead You could use **SECONDARY KEY**. ( cf. [Performance READ TABLE with Secondary Key vs Binary Search](https://abaptechnicalhelp.blogspot.com/2019/07/secondary-table-key-vs-binary-search.html))
    ```
    TYPES: ty_tab TYPE STANDARD TABLE OF t_tab
                  WITH NON-UNIQUE SORTED KEY k1 COMPONENTS k2.
    ```
 
 ### Check data existence
 #### Old Syntax
 ```
 READ TABLE it_tab WITH KEY iv_cond = 'A' TRANSPORTING NO FIELDS.
 If sy-subrc = 0.
 ENDIF.
 ```
 #### New Syntax
 ```
 IF line_exists( it_tab[ iv_cond = 'A' ]).
 ENDIF.
 ```
### Get index of data
#### Old Syntax
```
READ TABLE it_tab WITH KEY iv_cond = 'A' TRANSPORTING NO FIELDS.
DATA(iv_idx) = sy-tabix.
```
#### New Syntax
```
DATA(iv_idx) = line_index( it_tab[ iv_cond = 'A' ]).
```
### Change data
#### Old Syntax
 ```
 READ TABLE it_tab WITH KEY iv_cond = 'A' ASSIGNING FIELD-SYMBOL(<fs>).
 <fs>-iv_cond = 'NEW'.
 ```
 #### New Syntax
 ```
 DATA(iv_idx) = line_index(it_tab[ iv_cond = 'A' ]).
 IF iv_idx > 0.
    it_tab[ idx ]-iv_cond = 'NEW'.
 ENDIF.
 ```

 ## 7. Filter ITAB with condition
 - When moving <span style='color:red; font-weight:bold;'>Data that meets the conditions</span> from an internal table to another.
 - This is similar to **FOR ALL ENTRIES**.
 ```
 DATA(it_ntab) = FILTER #( it_tab USING KEY cond WHERE cond = 'A' ).
 ```
