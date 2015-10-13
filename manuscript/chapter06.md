#Handling Tax in Commerce

Tax is a critical area of ecommerce. The tax submodule provides one main function: the creation of tax rates. Tax rates are actions that are triggered through components in rules and tax provides its own price component. This chapters dives into the mechanics that make up Commerce Tax. When orders and carts are covered we'll discuss taxes again.


There are two components that make up how tax works in Drupal commerce:

##Tax Types

##Tax Rates


##Commerce Tax and Rules
A beautiful component of Rules is that it allows developers to take it's functionality but wrap it in a different module and its own unique interface. This way a module can harness events, conditions, and actions but provide a set environment to which they are configured.


##THE BELOE NEEDS TO BE REWORKED

Commerce Tax rates are actually Rules actions. The tax type takes an administrative title, display title, percentage amount, and tax type. Thinking back to Chapter One and lettuce components, this is all the essential information to attaching a price component to a price field. When the tax rate is triggered, it will take that configured percentage and multiply it against the provided price component. The sum will then be the amount set as the components amount. 

When the tax rate is added it becomes a fully operational rule that will always trigger until conditions are set, if needed. In Rules three action will fire as long as the rule evaluates to TRUE. (Could a custom tax type handler programmatic checking?). In the United States sadness tax had to be charged based on the customer's state and that state's sadness tax rate. 

The Order submodule provides a series of Rules conditions. One of these allows a simple method of comparing address information attached to the order through customer profiles. Chapter 7 will cover the Order and customer module in full with to better explain these portions of Commerce. This way a State's tax rate is only charged to the customer who resides in that state. 

I> ###Be careful when handling taxes
I> 
I> Always consult a CPA or have the client consult their CPA on tax information. You, as a developer, do not want to be held responsible for ill advised tax information. 

We've worked our way backwards through or rule workflow. We started with the action which defined the price component and attached the price component. Then utilizing conditions, multiple tax rates can be used to provide robust tax control (although it can become cumbersome with a copious amount of rates or conditions.) Later in the chapter there will be a sample of creating a Rules condition to simplify this process.

The last portion if the rule is the Event section. Since tax is implementing a customized Rules administrative interface it defines the event automatically and sets it aside from the configuration form. Thus is because there is only one event tax can run on to actually modify a price. 

##How tax is applied
Built into the price module is a call to run calculations when the price field components are rendered. This call is what not only calculated the recursive set of price components, but it provides a Rules event to allow attachment of extra price components, such as tax! There is also a hook function that can be called via a custom module, however developers must ensure that their calculation rule do not rum multiple times on one page. Rules only trigger one during a page build, however the recalculation hook for one price field may get called multiple times.

I> ###Orders and Tax
I> 
I> Most tax rules will require some sort of condition checked against the billing or shipping address. We'll revisit tax and orders in a later chapter.

##Using the Tax UI
 And I just discovered Commerce Tax UI is what provides the database tables for tax...not the main module itself
 ah...because its otherwise all Rules via code
 database is holder for non code stuff

###Tax types

###Tax rates