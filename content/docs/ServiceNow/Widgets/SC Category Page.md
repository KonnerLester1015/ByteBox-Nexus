---
title: "SC Category Page"
---
By default, ServiceNow includes a widget called "SC Category Page". This widget displays the popular requestable items within the Service Catalog. However, out-of-the-box it does not offer built-in functionality to exclude certain items or force include specific ones. In many scenarios, this capability is essential for tailoring the user experience and ensuring that only the most relevant items are displayed to users. 

This document will guide you through the process of customizing the "SC Category Page" widget to achieve this functionality. By the end of this guide, you will have a modified widget that allows you to specify items to exclude or force include, providing greater control over your Service Catalog presentation.

## Procedure
### Create System Properties
To ensure we don't have to go into the code everytime we want to change the item(s) we are excluding and including we can create a system property to store the list of sys_ids that the script will then reference when generating the list of catalog items.
1. Log in to your instance.
2. Click **All**.
3. Enter *sys_properties.list*.
4. Click **New**.
5. Enter *exclude_from_popular_list* into the Name field.
6. Enter the sys_id of the item you want to exclude in the Value field.
7. Click **Submit**.
8. Repeat these steps to create the `force_include_in_popular_list` property.

### Duplicate the "SC Category Page" Widget
To avoid any disruption to the existing functionality, the first step is to create a duplicate of the "SC Category Page" widget.
1. Navigate to the **sc_category** page located on the Service Portal (defualt URL https://yourinstance.service-now.com/sp?id=sc_category).
2. Right-click the **Popular Items** label. 
    {{< callout type="warning" >}}
    This will only work if you have sufficient privileges to modify the Serivce Portal Pages.
    {{< /callout >}}
3. Click **Widget in Editor ➚**.
4. Click the Hamburger icon (☰) on the top right of the **Widget Editor** page.
5. Click **Clone "SC Category Page"**.
6. Enter a name (*Custom SC Category Page*).
7. Click **Submit**.

### Modify Server Script
Replace the **getPopularItems()** function with the below code.
```js {filename="Custom SC Category Page - Server Script"}
function getPopularItems() {
        var itemsToExclude = gs.getProperty("exclude_from_popular_list");
        var forceIncludeItem = gs.getProperty("force_include_in_popular_list");
        
        var items = new SCPopularItems().useOptimisedQuery(gs.getProperty('glide.sc.portal.popular_items.optimize', true) + '' == 'true')
            .baseQuery(options.popular_items_created + '')
            .allowedItems(getAllowedCatalogItems())
            .visibleStandalone(true)
            .visibleServicePortal(true)
            .itemsLimit(5)
            .restrictedItemTypes('sc_cat_item_guide,sc_cat_item_wizard,sc_cat_item_content,sc_cat_item_producer'.split(','))
            .itemValidator(function(item, itemDetails) {
                if(itemsToExclude.indexOf(itemDetails.sys_id) > -1)
                    return false;
    
                if (!item.canView() || !item.isVisibleServicePortal())
                    return false;
    
                return true;
            })
            .responseObjectFormatter(function(item, itemType, itemCount) {
                return {
                    order: 0 - itemCount,
                    name: item.name,
                    short_description: item.short_description,
                    picture: item.picture,
                    price: item.price,
                    sys_id: item.sys_id,
                    hasPrice: item.price != 0,
                    page: itemType == 'sc_cat_item_guide' ? 'sc_cat_item_guide' : 'sc_cat_item'
                };
            })
            .generate();
    
        // Add the forced item with a very large negative order to appear at the top
        var sticky = new GlideAggregate('sc_cat_item');
        sticky.addQuery('sys_id', forceIncludeItem);
        sticky.query();
    
        while (sticky.next()) {
            if (!$sp.canReadRecord("sc_cat_item", sticky.sys_id))
                continue; // user does not have permission to see this item
    
            var item = {};
            item.name = sticky.name.getDisplayValue();
            item.short_description = sticky.short_description.getDisplayValue();
            item.picture = sticky.picture.getDisplayValue();
            item.price = sticky.price;
            item.sys_id = sticky.sys_id.getDisplayValue();
            item.order = -1000000000; // Ensure this item appears at the top
            item.page = 'sc_cat_item'; // Ensure it routes to the correct page
    
            items.unshift(item); // Add this item to the start of the list
        }
    
        return items;
    }
```

### Add New Widget to SC_Category Page
1. Navigate to the **sc_category** page located on the Service Portal (defualt URL https://yourinstance.service-now.com/sp?id=sc_category).
2. Right-click the **Popular Items** label. 
    {{< callout type="warning" >}}
    This will only work if you have sufficient privileges to modify the Serivce Portal Pages.
    {{< /callout >}}
3. Click **Page in Designer ➚**. 
4. Click the trashcan icon on the "SC Category Page" widget
5. Click **Yes**
6. Search *Custom SC Category Page* in the **Filter Widget** box
7. Drag and drop the **Custom SC Category Page** widget into the page

By following these steps, you've successfully modified the "SC Category Page" widget to include new functionality for excluding and force including requestable items. This customization can greatly enhance the usability and relevance of your Service Catalog, ensuring that users see only the most pertinent options.

For further customization or advanced use cases, consider extending the widget to support additional filtering criteria or dynamic inclusion rules based on user roles or other conditions.