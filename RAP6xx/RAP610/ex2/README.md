[Home - RAP610](../../#exercises)

# Exercise 2: Implement the Business Logic of Your Business Object

In the previous exercise, you've built a RAP BO with transactional processing capabilities (see **[Exercise 1](../ex1/README.md)**).

You can enhance this BO by integration released RAP facades. 

In the present exercise, you will incorporate **`I_PURCHASEREQUISITIONTP`** API in the business logic of your RAP BO.


## Step 2.1: Create a New Database Table for Additional Save Feature
[^Top of page](#)

**`I_PURCHASEREQUISITIONTP`** is implemented with late numbering.   

This means the unique primary key of the business object is provided only after the application exit for save has been triggered. 
You need to implement convert key feature in your business implementation logic. Thiis can be done in implementation for addition save.
The newly created purchase requisition can be stored in a new database table. More details can be found in documentation.

For this step, first create a new database table with fields for purchase requisition and any other fields which you wish to try out.

<details>
  <summary>Click to expand!</summary>
	
1.	Right-click your package **`Z_PURCHASE_REQ_XXX`** and select **New > Other ABAP Repository Object** from the context menu.
 
    ![](ex1/images/ui7.png)
 
2.	Search for **_Database Table_**, select the entry, and click **Next >**.
    
    ![](../../images/ui8.png)
 
3. Maintain the information provided below and click **Next >**.

    -	Name: **`ZSHOP_AS_XXX`**
    -	Description: _**`Database table for additional save`**_

    ![](../../images/ui9.png)
 
4.	Select your transport request and click **Finish**.
    
    ![](../../images/ui10.png)
 
5.	Replace your default source code with following code snippet:

    ```ABAP
    @EndUserText.label : 'Database table for additional save'
    @AbapCatalog.enhancement.category : #NOT_EXTENSIBLE
    @AbapCatalog.tableCategory : #TRANSPARENT
    @AbapCatalog.deliveryClass : #A
    @AbapCatalog.dataMaintenance : #RESTRICTED
    define table zshop_as_xxx {
      key client     : abap.clnt not null;
      key order_uuid : sysuuid_x16 not null;
      purchasereqn   : abap.string(256);
      purinforecord  : abap.string(256);
      purorder       : abap.string(256);
    }
     ```

6. Save and activate the object.

</details>

## Step 2.2: Enhance the CDS View Entity `ZI_ONLINE_SHOP_XXX`
[^Top of page](#)

You will now expose the purchase requisition field from database table **`ZSHOP_AS_XXX`** in the CDS view entity **`ZI_ONLINE_SHOP_XXX`**

<details>
  <summary>Click to expand!</summary>
	
1. Create a CDS view entity for the database table **`ZSHOP_AS_XXX`**. 
   
   For that, right-click on the database table **`ZSHOP_AS_XXX`** and select **New Data Definition** from the context menu.

2. Maintain the information provided below and click **Next **. 
    - Name: **`ZI_SHOP_AS_XXX`** 
    - Description: _**`Data model for online shop`**_ .

3. Select your transport request and click **Finish**.
    
    ![](../../images/as.png)

4. Add the assosiation **`_purchase_req`** to CDS view entity **`ZI_SHOP_AS_XXX`** in the BO view **`ZI_ONLINE_SHOP_XXX`**.

    ```ABAP
     association [1..1] to ZSHOP_I_AS_HB            as _purchase_req      on  $projection.Order_Uuid = _purchase_req.OrderUuid
     ```

5. Add the assosiation **`_purchasereq`** in the field list.

    ![](../../images/Exercise2.2_addnlsave.jpg)
     
    Your CDS view entity **`ZI_ONLINE_SHOP_XXX`** should look like the following
     
    ```ABAP
    @EndUserText.label: 'Data model for online shop'
    @AccessControl.authorizationCheck: #NOT_REQUIRED
    define root view entity ZHB_I_ONLINE_STORE as select from zonlineshop_hb
    association [1..1] to ZSHOP_I_AS_HB            as _purchase_req      on  $projection.Order_Uuid = _purchase_req.OrderUuid
     {
      key order_uuid as Order_Uuid,
      order_id as Order_Id,
      ordereditem as Ordereditem,
      deliverydate as Deliverydate,
      creationdate as Creationdate  ,
      _purchase_req
    }
    ```

6. Open the Projection view **`ZC_ONLINE_SHOP_XXX`**

7. Add the following code after field **`Creation_Date`** and between the curly brackets (**`}`**).

    ```ABAP
    ,   @UI: { lineItem:       [ { position: 60,label: 'Purchase Req', importance: #HIGH }  ],
                 identification: [ { position: 60, label: 'Purchase Req' } ] }   
          _purchase_req.Purchasereqn as PurchaseRequistionNumber
    ```

    Your global class should look like shown below:
    
    ![](../../images/projection.png)

    Code snippet **`ZC_ONLINE_SHOP_XXX`**

    ```ABAP
     @EndUserText.label: 'shop projection'
     @AccessControl.authorizationCheck: #NOT_REQUIRED
     @Search.searchable: true
     @UI: { headerInfo: { typeName: 'Online Shop',
                         typeNamePlural: 'Online Shop',
                         title: { type: #STANDARD, label: 'Online Shop', value: 'order_id' } },
           presentationVariant: [{ sortOrder: [{ by: 'Creationdate',direction: #DESC }] }] }
     define root view entity ZHB_C_ONLINE_STORE provider contract transactional_query
      as projection on ZHB_I_ONLINE_STORE
     {

         @UI.facet: [          { id:                    'Orders',
                                       purpose:         #STANDARD,
                                      type:            #IDENTIFICATION_REFERENCE,
                                      label:           'Order',
                                       position:        10 }      ]
      key Order_Uuid,
          @UI: { lineItem:       [ { position: 10,label: 'order id', importance: #HIGH } ],
                   identification: [ { position: 10, label: 'order id' } ] }
          @Search.defaultSearchElement: true
          Order_Id,
          @UI: { lineItem:       [ { position: 20,label: 'Ordered item', importance: #HIGH } ],
                  identification: [ { position: 20, label: 'Ordered item' } ] }
          @Search.defaultSearchElement: true
          Ordereditem,
          Deliverydate       as Deliverydate,
          @UI: { lineItem:       [ { position: 50,label: 'Creation date', importance: #HIGH }
    //      ,                { type: #FOR_ACTION, dataAction: 'copyOrder', label: 'Duplicate Order' }
                                    ],
                identification: [ { position: 50, label: 'Creation date' } ] }
         Creationdate       as Creationdate,
          @UI: { lineItem:       [ { position: 60,label: 'Purchase Req', importance: #HIGH }
                                    ],
                 identification: [ { position: 60, label: 'Purchase Req' } ] }   
          _purchase_req.Purchasereqn as PurchaseRequistionNumber      
     }
     ```

</details>

## Step 2.3: Adapt the Behavior Implementation
[^Top of page](#)

Now, you can create and enhance business implementation logic with code snippets for internal action for triggering purchase requisition API and additional save feature. 

<details>
  <summary>Click to expand!</summary>
	
1. In your behavior definition **`ZI_ONLINE_SHOP_XXX`**, set the cursor before the implementation class **`zbp_i_online_shop_xxx`** and click **CTRL + 1**. 

    Double-click on Create behavior implementation class **`zbp_i_online_shop_xxx`** to create your implementation class. 

    ![](../../images/ui24.png)

2. Create new implementation class and click **Next >**. 
    
    ![](../../images/ui25.png)

3. Select a transport request and click **Finish**.
	 
    ![](../../images/ui26.png)

4. In your global class, add the follwing code snippet :

    ```ABAP 
    PUBLIC SECTION.
    class-DATA cv_pr_mapped TYPE RESPONSE FOR MAPPED i_purchaserequisitiontp.
    ```

    In the end, you global class will look like below:

    ```ABAP 
    CLASS zbp_i_online_shop_xxx DEFINITION PUBLIC ABSTRACT FINAL FOR BEHAVIOR OF ZI_online_SHOP_xxx.
     PUBLIC SECTION.
     class-DATA cv_pr_mapped TYPE RESPONSE FOR MAPPED i_purchaserequisitiontp.
     ENDCLASS.

     CLASS zbp_i_online_shop_xxx IMPLEMENTATION.
     ENDCLASS.
     ```

5. On the **Local Types** tab, replace your source code with following code:**

    ```ABAP
     CLASS lsc_zhb_i_online_shop_act DEFINITION INHERITING FROM cl_abap_behavior_saver.
      PROTECTED SECTION.
        METHODS save_modified REDEFINITION.
     ENDCLASS.

     CLASS lsc_zhb_i_online_shop_act IMPLEMENTATION.

     METHOD save_modified.
        DATA : lt_online_shop_as TYPE STANDARD TABLE OF Zshop_AS_xxx,
               ls_online_shop_as TYPE                   Zshop_AS_xxx.
        IF zbp_i_online_shop_xxx=>cv_pr_mapped-purchaserequisition IS NOT INITIAL.
          LOOP AT zbp_i_online_shop_xxx=>cv_pr_mapped-purchaserequisition ASSIGNING FIELD-SYMBOL(<fs_pr_mapped>).
            CONVERT KEY OF i_purchaserequisitiontp FROM <fs_pr_mapped>-%key TO DATA(ls_pr_key).
            <fs_pr_mapped>-purchaserequisition = ls_pr_key-purchaserequisition.
          ENDLOOP.
        ENDIF.
        IF create-online_shop IS NOT INITIAL.
         " Creates internal table with instance data
          lt_online_shop_as = CORRESPONDING #( create-online_shop ).
          lt_online_shop_as[ 1 ]-purchasereqn =  ls_pr_key-purchaserequisition .
          insert Zshop_AS_xxx FROM TABLE @lt_online_shop_as.
        ENDIF.
      ENDMETHOD.

    ENDCLASS.
     CLASS lhc_zbp_i_online_shop_xxx  DEFINITION INHERITING FROM cl_abap_behavior_handler. 
      PRIVATE SECTION.
         METHODS get_instance_authorizations FOR INSTANCE AUTHORIZATION
         IMPORTING keys REQUEST requested_authorizations FOR  online_shop RESULT result.

        METHODS create_pr FOR MODIFY
          IMPORTING keys FOR ACTION online_shop~create_pr.

       METHODS calculate_order_id FOR DETERMINE ON MODIFY
         IMPORTING keys FOR online_shop~calculate_order_id.

     ENDCLASS.

     CLASS lhc_zbp_i_online_shop_xxx  IMPLEMENTATION.

     METHOD get_instance_authorizations.
      ENDMETHOD.

      METHOD calculate_order_id.
        DATA:
          online_shops TYPE TABLE FOR UPDATE ZI_ONLINE_SHOP_xxx,
          online_shop  TYPE STRUCTURE FOR UPDATE ZI_ONLINE_SHOP_xxx.

        SELECT MAX( order_id ) FROM ZI_ONLINE_SHOP_xxx INTO @DATA(max_order_id).
        READ ENTITIES OF ZI_ONLINE_SHOP_xxx IN LOCAL MODE
           ENTITY Online_Shop
            ALL FIELDS
              WITH CORRESPONDING #( keys )
              RESULT DATA(lt_online_shop_result)
          FAILED    DATA(lt_failed)
          REPORTED  DATA(lt_reported).
        DATA(today) = cl_abap_context_info=>get_system_date( ).

        LOOP AT lt_online_shop_result INTO DATA(online_shop_read).
         max_order_id += 1.

          online_shop               = CORRESPONDING #( online_shop_read ).
        online_shop-order_id      = max_order_id.
          online_shop-Creationdate  = today.
         online_shop-Deliverydate  = today + 10.
          APPEND online_shop TO online_shops.
        ENDLOOP.
        MODIFY ENTITIES OF ZI_ONLINE_SHOP_xxx IN LOCAL MODE
       ENTITY ZI_ONLINE_SHOP_xxx UPDATE SET FIELDS WITH online_shops
       MAPPED   DATA(ls_mapped_modify)
       FAILED   DATA(lt_failed_modify)
       REPORTED DATA(lt_reported_modify).


        IF lt_failed_modify IS INITIAL.
              MODIFY ENTITIES OF ZI_ONLINE_SHOP_xxx IN LOCAL MODE
          ENTITY Online_Shop EXECUTE create_pr FROM CORRESPONDING #( keys )
          FAILED DATA(lt_pr_failed)
          REPORTED DATA(lt_pr_reported).
        ENDIF.

      ENDMETHOD.

    method create_pr.

    endmethod .

    ENDCLASS.
    ```

6. Save and activate the object.

7. Now, you can open the service binding **`ZSB_SHOP_XXX`** and click on **Fiori elements App Preview** for the entity **`orders`**.

8. In the Fiori elements preview of the application, create a new order for a laptop.

    > **HINT:**   
    > The option internal can be set before the action name to only provide an action for the same BO. An internal action can only be accessed from the business logic inside the business object implementation such as from a determination or from another action.

</details>
	
## Summary
[^Top of page](#)

You are through with this exercise.

You can either open the [documentation](../../documentation.md) or continue with the next exercise – **[Exercise 3: Create a Developer Extension using a PaaS API](../ex3/README.md)**

