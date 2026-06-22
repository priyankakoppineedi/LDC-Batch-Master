# LDC-Batch-Master

**Include for BATCH UPD Fm ZOPS_PRINT_BATCH_UPD**

*&---------------------------------------------------------------------*
*& Include ZOPS_PRINT_BATCH_UPD
*&---------------------------------------------------------------------*
**                                                                     *
*Development ID:  ZDDE-00098005                                        *
*Functional Spec: ZFSE-00098004                                        *
*Description: This Include is for the EXIT framework and called from   *
*             DERIVATION BADI for calculating SLED/BBD of master batch *
*             This Include is called via below ABAP Stack              *
*              BADI : DERIVATION                                       *
*              Enh Implementation : ZOPS_EI_DERIVATION                 *
*              Class Implementation : ZCL_IM_OPS_EI_DERIVATION         *
*              Method : IF_EX_DERIVATION~RECIPIENT_VALUES_AND_STATUS   *
*----------------------------------------------------------------------*

CONSTANTS: lc_prog1  TYPE programm  VALUE 'ZOPS_PRINT_BATCH_UPD',
           lc_stack  TYPE dbglprog  VALUE 'SAPLDRVN',
           lc_caufvd TYPE syrepid   VALUE '(SAPLZOPS_FG_BEFORE_UPDATE_BDG)GT_HEADER',
           lc_key1   TYPE string    VALUE 'WERKS'.

DATA: lv_val1     TYPE string,
      lt_func1    TYPE fbname,
      lt_sys_stck TYPE sys_callst,
      lt_mcha     TYPE TABLE OF mcha,
      ls_caufvd   TYPE caufvd,
      lv_werks    TYPE werks_d.

CALL FUNCTION 'SYSTEM_CALLSTACK'
  IMPORTING
    et_callstack = lt_sys_stck.
*Get Material and batch from system stack
IF lt_sys_stck[] IS NOT INITIAL.
  ASSIGN lt_sys_stck[ progname = lc_stack ] TO FIELD-SYMBOL(<lfs_stck>).
  IF <lfs_stck> IS ASSIGNED.
    ASSIGN (lc_caufvd) TO FIELD-SYMBOL(<lfs_caufvd>).
    IF <lfs_caufvd> IS ASSIGNED.
      lt_mcha = CORRESPONDING #( <lfs_caufvd> ).
      lv_werks = VALUE #( lt_mcha[ 1 ]-werks OPTIONAL ).

      CALL FUNCTION 'ZEXIT_CHECK'
        EXPORTING
          p_include   = lc_prog1
          p_key1      = lc_key1
          p_val1      = lv_werks
        CHANGING
          func_module = lt_func1
        EXCEPTIONS
          not_active  = 1
          OTHERS      = 2.
      IF sy-subrc IS INITIAL.
        LOOP AT lt_func1 ASSIGNING FIELD-SYMBOL(<fs_func1>).
          CALL FUNCTION <fs_func1>
            EXPORTING
              i_recipient_condition_record = i_recipient_condition_record
              it_sender_values             = it_sender_values
            IMPORTING
              et_messages                  = et_messages
            CHANGING
              c_error                      = c_error
              ct_recipient_values          = ct_recipient_values
              ct_messages                  = ct_messages
            EXCEPTIONS
              stop_derivation              = 1
              OTHERS                       = 2.
          IF sy-subrc IS NOT INITIAL.
            RAISE stop_derivation.
          ENDIF.
        ENDLOOP.
      ENDIF.
    ENDIF.
  ENDIF.
ENDIF.

**FM - ZOPS_FM_PRINT_BATCH_UPD**

FUNCTION zops_fm_print_batch_upd.
*"----------------------------------------------------------------------
*"*"Local Interface:
*"  IMPORTING
*"     REFERENCE(I_RECIPIENT_CONDITION_RECORD) TYPE  KONDRPR
*"     REFERENCE(IT_SENDER_VALUES) TYPE  DRV_FIELDVAL_TAB
*"  EXPORTING
*"     REFERENCE(ET_MESSAGES) TYPE  DRVN_MSG_TAB
*"  CHANGING
*"     REFERENCE(C_ERROR) TYPE  XFLAG
*"     REFERENCE(CT_RECIPIENT_VALUES) TYPE  DRV_FIELDVAL_TAB
*"     REFERENCE(CT_MESSAGES) TYPE  DRVN_MSG_TAB OPTIONAL
*"  EXCEPTIONS
*"      STOP_DERIVATION
*"----------------------------------------------------------------------
*----------------------------------------------------------------------*
*Development ID:  ZDDE-00064879                                        *
*Functional Spec: ZFSE-00064878                                        *
*Include : ZOPS_PRINT_BATCH_UPD                                        *
*          This Include is called via below ABAP Stack                 *
*          BADI : DERIVATION                                           *
*          Enh Implementation : ZOPS_EI_DERIVATION                     *
*          Class Implementation : ZCL_IM_OPS_EI_DERIVATION             *
*          Method : IF_EX_DERIVATION~RECIPIENT_VALUES_AND_STATUS       *
*Description:The purpose of this FM is to Update Print batch Charact   *
* As Header Batch If EXIT1 is not maintained in DVR3 for ZX0G_PRINTBAT *
*"----------------------------------------------------------------------
  DATA: lt_sys_stck        TYPE sys_callst,
        lt_mch1            TYPE TABLE OF mch1,
        lt_mcha            TYPE TABLE OF mcha,
        lv_objecttable     TYPE bapi1003_key-objecttable,
        lv_classnum        TYPE bapi1003_key-classnum,
        lv_classtype       TYPE bapi1003_key-classtype,
        lc_classnum        TYPE bapi1003_key-classnum    VALUE 'Z0X_BATCH_GLOBAL',
        lc_classtype       TYPE bapi1003_key-classtype   VALUE '023',
        lv_objectkey_long  TYPE bapi1003_key-object_long,
        lt_allocvaluesnum  TYPE TABLE OF bapi1003_alloc_values_num,
        lt_allocvaluescurr TYPE TABLE OF bapi1003_alloc_values_curr,
        lt_allocvalueschar TYPE TABLE OF bapi1003_alloc_values_char,
        lt_object          TYPE STANDARD TABLE OF bapi1003_object_keys,
        lr_werks           TYPE RANGE OF werks,
        lt_return          TYPE TABLE OF bapiret2.

  CONSTANTS: lc_stack       TYPE dbglprog   VALUE 'SAPLDRVN',
             lc_e(1)        TYPE c          VALUE 'E',
             lc_charact     TYPE drvfl      VALUE 'ZX0G_PRINTBAT',
             lc_matnr       TYPE mara-matnr VALUE 'MATNR',
             lc_charg       TYPE mch1-charg VALUE 'CHARG',
             lc_objecttable TYPE bapi1003_key-objecttable VALUE 'MCH1',
             lc_caufvd      TYPE syrepid   VALUE '(SAPLZOPS_FG_BEFORE_UPDATE_BDG)GT_ITEM',
             lc_caufvd1     TYPE syrepid   VALUE '(SAPLZOPS_FG_BEFORE_UPDATE_BDG)GT_HEADER',
             lc_repid       TYPE progname VALUE 'ZOPS_FM_PRINT_BATCH_UPD',
             lc_werks       TYPE zparname VALUE 'WERKS'.

  CALL FUNCTION 'SYSTEM_CALLSTACK'
    IMPORTING
      et_callstack = lt_sys_stck.
*Get Material and batch from system stack
  IF lt_sys_stck[] IS NOT INITIAL.
    ASSIGN lt_sys_stck[ progname = lc_stack ] TO FIELD-SYMBOL(<lfs_stck>).
    IF <lfs_stck> IS ASSIGNED.
      ASSIGN (lc_caufvd) TO FIELD-SYMBOL(<lfs_caufvd>).
      IF <lfs_caufvd> IS ASSIGNED.
        lt_mch1 = CORRESPONDING #( <lfs_caufvd> ).
        DATA(ls_mch1) = VALUE #( lt_mch1[ 1 ] OPTIONAL ).
      ENDIF.
      ASSIGN (lc_caufvd1) TO FIELD-SYMBOL(<lfs_caufvd1>).
      IF <lfs_caufvd1> IS ASSIGNED.
        lt_mcha = CORRESPONDING #( <lfs_caufvd1> ).
        DATA(lv_werks) = VALUE #( lt_mcha[ 1 ]-werks OPTIONAL ).
      ENDIF.
      APPEND VALUE #( key_field = lc_matnr value_int = ls_mch1-matnr ) TO lt_object.
      APPEND VALUE #( key_field = lc_charg value_int = ls_mch1-charg ) TO lt_object.


      SELECT 'I' AS sign, 'EQ' AS constant, value AS low ,
      CAST( @space AS CHAR( 80 ) ) AS high
      FROM zgq_parameter
      INTO TABLE @lr_werks
      WHERE repid EQ @lc_repid
     AND param EQ @lc_werks.
      IF sy-subrc = 0 AND  lv_werks IN lr_werks.

        lv_objecttable = lc_objecttable.

        CALL FUNCTION 'BAPI_OBJCL_CONCATENATEKEY'
          EXPORTING
            objecttable         = lv_objecttable
          IMPORTING
            objectkey_conc_long = lv_objectkey_long
          TABLES
            objectkeytable      = lt_object
            return              = lt_return.

        IF lv_objectkey_long IS NOT INITIAL.
          lv_classnum       = lc_classnum.
          lv_classtype      = lc_classtype.

          CALL FUNCTION 'BAPI_OBJCL_GETDETAIL'
            EXPORTING
              objectkey_long   = lv_objectkey_long
              objecttable      = lv_objecttable
              classnum         = lv_classnum
              classtype        = lv_classtype
              unvaluated_chars = abap_true
            TABLES
              allocvaluesnum   = lt_allocvaluesnum
              allocvalueschar  = lt_allocvalueschar
              allocvaluescurr  = lt_allocvaluescurr
              return           = lt_return.
          IF NOT line_exists( lt_return[ type = lc_e ] ).
*Set Header batch only after validating SAPLZOPS_FG_BEFORE_UPDATE_BDG in STACK
            ASSIGN lt_allocvalueschar[ charact = lc_charact ] TO FIELD-SYMBOL(<lfs_charac>) ELSE UNASSIGN .
            IF <lfs_charac> IS ASSIGNED.
              DATA(lv_printbat) = <lfs_charac>-value_char.
            ENDIF.
            IF ct_recipient_values IS NOT INITIAL.
              IF i_recipient_condition_record-drvfl = lc_charact.
                ct_recipient_values[ 1 ]-drvchar = lv_printbat.
              ENDIF.
            ENDIF.
          ENDIF.
        ENDIF.
      ENDIF.
    ENDIF.
  ENDIF.
ENDFUNCTION.

**Include - ZOPS_INC_PO_BEFORE_UPDATE**

*----------------------------------------------------------------------*
*Development ID:  ZDDE-00064879                                        *
*Functional Spec: ZFSE-00064878                                        *
*Description: Update PO number in Batch Classification                 *
*Path: BADI WORKORDER_UPDATE -> ZPO_WORKORDER_UPDATE                   *
* -> ZCL_IM_PO_WORKORDER_UPDATE -> IF_EX_WORKORDER_UPDATE~BEFORE_UPDATE*
*     -> ZOPS_INC_PO_BEFORE_UPDATE -> ZOPS_FM_PO_BEFORE_UPDATE         *
*-----------------------------------------------------------------------
*Change Log:                                                           *
*----------------------------------------------------------------------*
* Who       Date           CR            Text                          *
* MOHAMVI2  02-Dec-2025  1100281693    Initial Development             *
*----------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*& Include          ZOPS_INC_PO_BEFORE_UPDATE
*&---------------------------------------------------------------------*

DATA: lt_fm_name TYPE fbname.

CONSTANTS: lc_include TYPE string VALUE 'ZOPS_INC_PO_BEFORE_UPDATE',
           lc_key1    TYPE string VALUE 'WERKS'.

IF lines( it_header ) > 0.
  CALL FUNCTION 'ZEXIT_CHECK'
    EXPORTING
      p_include   = lc_include
      p_key1      = lc_key1
      p_val1      = it_header[ 1 ]-werks
    CHANGING
      func_module = lt_fm_name
    EXCEPTIONS
      not_active  = 1
      OTHERS      = 2.
  IF sy-subrc IS INITIAL.
    LOOP AT lt_fm_name ASSIGNING FIELD-SYMBOL(<fs_fm_name>).
      CALL FUNCTION <fs_fm_name>
        EXPORTING
          it_header             = it_header
          it_header_old         = it_header_old
          it_item               = it_item
          it_item_old           = it_item_old
          it_sequence           = it_sequence
          it_sequence_old       = it_sequence_old
          it_operation          = it_operation
          it_operation_old_afvc = it_operation_old_afvc
          it_operation_old_afvv = it_operation_old_afvv
          it_operation_old_afvu = it_operation_old_afvu
          it_component          = it_component
          it_component_old      = it_component_old
          it_relationship       = it_relationship
          it_relationship_old   = it_relationship_old
          it_pstext             = it_pstext
          it_pstext_old         = it_pstext_old
          it_milestone          = it_milestone
          it_milestone_old      = it_milestone_old
          it_planned_order      = it_planned_order
          it_status             = it_status
          it_status_old         = it_status_old
          it_opr_relations      = it_opr_relations
          it_opr_relations_old  = it_opr_relations_old
          it_doclink            = it_doclink
          it_doclink_old        = it_doclink_old
          it_prt_allocation     = it_prt_allocation
          it_prt_allocation_old = it_prt_allocation_old
          it_pmpartner          = it_pmpartner
          it_pmpartner_old      = it_pmpartner
          it_piinstruction      = it_piinstruction
          it_piinstructionvalue = it_piinstructionvalue.
    ENDLOOP.
  ENDIF.
ENDIF.

FM
