#Commerce Line Items
The goal of this book is to teach the underlying architecture and implementation of the Commerce module. We first covered how prices are handled. This chapter covers the Line Item module which provides the entity that powers the back end of how Commerce operates.

Many contributed modules that interact with Commerce provide their own line item, such as Commerce Discount and Commerce Shipping.

I> ##Tax is not a line item!
I> 
I> Tax is not handled as a line item and is treated differently. It is automatically bundled into a line item's price components. We'll cover tax later in the book.

Line items are fieldable entities with predefined components. Line items are not directly exposed for utilization. Instead, think of line items as reference bridges and ways to provide data attributes along the way. For example, line items create the foundation of what constitutes an order.

If a product is added to an order, a new line item is created that references that product. This way the order now has the product and a bridge to reference its attributes. Or a fee or shipping service and add a line item to affect the order's total.

##Components of a line item

Line item entities have specified properties that appear as attached fields. All line items also have two price fields attached, which is locked and cannot be modified. 

![Default properties of a Commerce Line Item entity](images/Commerce_Line_Item_Diagram.png)

Certain properties are attached as pseudo fields (label, quantity) from the entities properties. Others are used programmatically within the system and contributed modules.

* **Line item types** are the entity bundle to provide different instances of line items. This allows representation of products, discounts, shipping with unique fields or property manipulation.
* The **Order ID** is a foreign key to the Commerce Order which owns the line item. As stated previously, line items are designed to be the background bridge connecting the components of Commerce, constructing an order.
* **Data** is simple a serialized array stored in the database. Unique use case data can be attached to line items without requiring a new field or altering the module's schema.
* **Line item labels** are the human readable identifier of the line item. Commerce Product sets this as the product's SKU, for example.
* **Quantity** is the total amount of this particular line item that is attached to the order. Commerce line item does not provide checks to combine line items and increment a quantity count, that is the duty of the module (such as Commerce Cart.)
* **Unit Price** is the actual price of the line item.
* The **Total** is the sum of the line item's quantity and its unit price.

The data property is useful for providing information flags. For example, Commerce Cart adds a value for storing a line item's product's display path. Commerce Shipping embeds its service component information as a data attribute. It is not meant to replace fields, but a way to contain data that doesn't quite belong as a field and would require altering of the database schema.

###Use of line item types
Since line items are entities, line items can have different types. Similar to the way one would have multiple content types. Using different line item types can involve typical use cases of shipping and discounts to very customized products.

I> ###Line item types
I>
I>  Fees and discounts could be their own individual line item that appear on orders, instead of merely being a price component on individual line item price's.

The Product module, as we'll learn more about later, provides a line item with a reference to the product it was created from. The Commerce Shipping module provides a new type if line item to provide shipping service information and charges. Promotional contributed can also provide their line item types to describe discounts applied to the order. Fees can also play a role as line items.  

Line item types can only be created through the line item module's API, or product based line items can be created through the contributed Commerce Customizable Products module (think of it as more an extension of the Line Item UI module.) The process of defining a custom line item will be covered in this chapter, as well in the Product & Product References chapter. 

###Fields on a line item
The fact that line items are entities allow for more robust data types to be associated with  a line item. Developers do not have to new data structures for storing unique information. Any kind of field can be attached to a line item to provide a fit for any use case.

This allows Commerce site builders and developers to build out robust products. Commerce Product, which we'll learn about later, adds a Product Reference field to a line item. Instead of duplicating a product's information into the line item, it builds a bridge to the existing item to be displayed within an order.


####Unit price and Total price
As stated previously, the unit price and total price are two price fields which are automatically added to all line item types. The line item entity controller handles population of the `commerce_total` field whenever the entity is saved.

When the line item entity is saved it extracts the price component stored in the `commerce_unit_price` field. The amounts value is multiplied against the quantity value.

      // Update the total of the line item based on the quantity and unit price.
      $unit_price = commerce_price_wrapper_value($wrapper, 'commerce_unit_price', TRUE);

      $wrapper->commerce_total->amount = $line_item->quantity * $unit_price['amount'];
      $wrapper->commerce_total->currency_code = $unit_price['currency_code'];

A price component is made up of an amount, currency code, and data. The data value is an array of price components that created the calculated amount value. These components are updated as well per the quantity amount and stored in `commerce_total`'s data value.

If an amount of a price component is adjusted, all of its components must be updated as well. A helper function `commerce_line_item_rebase_unit_price()` is provided to simplify this process for developers and other processes. It follows a similar process to the one triggered on an entity's save event. However, the function provides a hook to allow other modules to interact with a line item's unit price. 

Modules are able to add price components to a price field through `hook_commerce_line_item_rebase_unit_price()`. So the field's original price components are stored aside. Without doing so the calculates amounts may be stale, such as calculated tax.

I> ###Line items without base_price
I> 
I> In the event your line item uses a price field without base_price, the function will preserve the component. It assumes that the first price component in the component's data array is to represent the base_price.

The new price field is set with its base price component. Other modules are now given a chance to add extra price components to a line item through the provided hook.

    // Give other modules a chance to add components to the array.
    foreach (module_implements('commerce_line_item_rebase_unit_price') as $module) {
      $function = $module . '_commerce_line_item_rebase_unit_price';
      $function($price, $old_components, $line_item);
    }

Modules implementing `hook_commerce_line_item_rebase_unit_price` are passed the new price component by reference, the old components, and the current line item. The module can check for stale price components, or review the line item and attach new components to the price component.

For example, a Line Item Manager is provided for adding and managing line items attached to an entity. Whenever the form is saved and a price is changed the rebase function is called before saving the entity.

When we reach handling tax we'll revisit this hook again and how it handles recalculating a product's tax amount on price change.


###Tokens and Line Items
Since Commerce integrates with Rules, the Line Item module provides tokens to expose line item data to rules. The basic information is available: ID, type, name, label, quantity, and so forth.

The most important token,however, is the *order* token. This allows a line item to reference the order it belongs to. As we go through the book, or just through using Commerce, you'll realize how useful this ability is.

##Line Item Reference Field
The line item reference field provides a way to associate line item entity's to another entity.  If you're familiar with the entity reference field, you can associate this in much the same way. 

Why not use the entity reference field? Commerce utilizes it's own reference fields to provide greater control and customization with it's behaviors. This allows for a specialized field widget and field instance. 

###Line Item Manager
Commerce line item reference fields have a Line Item Manager field widget. The line item manager provides a form that allows the addition and manipulation of line items attached to another entity.

INSERT PICTURE OF THE FIELD WIDGET 

You will be presented with a table of line items and specific data properties and fields. One of which will be an editable unit price and quantity field. Prices can of a line item can be set when manually added through the form. 

Line items are created by selecting the line item type and clicking **add**. The line item widget plays a large role in order management through the user interface. 


###Line Item Summary
The line item summary is the only display formatter available for the line item reference field. The display formatter harnesses a View for its display, treating each line item as a view of its own. Commerce Line Item module provided a default view,  which can be changed on a per field instance case. 

Your line item summary powers the itemized list of line items attached to a field. When viewing an order this is how customers know what products and fees are in the order.  

INSERT PICTURE OF A COMMERCE LINE ITEM TABLE 

As mentioned previously, line items can have fields. Using the power of Views,  this output is customizable to provide a solution to any e-commerce need. 

The default view is `commerce_line_item_table`.  It is highly recommended that you create a new View or programmatically alter the existing one. Why? This allows you to upgrade Commerce and not worry about having to lose customizations to access component enhancements. 

##Rules Integration

##Line Item API
Commerce line items have their own API that allow modules to create their own line item types. The Commerce Line Item UI module does not expose a way to create new types.
