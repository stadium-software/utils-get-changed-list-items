# Get Changed List Items

Pass in an original List of objects and an updated List and get back the items that were added or changed.

This has been tested with simple lists of objects, not with deep lists. See the sample application to test it with your data.

# Version 

1.0

# Global Script Setup
1. Create a Global Script called "GetChangedListItems"
2. Add the input parameters below to the Global Script
   1. IDColumn
   2. OriginalList
   3. UpdatedList
3. Add the output parameter below to the Global Script
   1. Return
4. Drag a *JavaScript* action into the script
5. Add the Javascript below unchanged into the JavaScript code property
```javascript
/* Stadium Script v1.0 https://github.com/stadium-software/utils-get-changed-list-items */
let list1 = ~.Parameters.Input.OriginalList;
let list2 = ~.Parameters.Input.UpdatedList;
let idColumn = ~.Parameters.Input.IDColumn;
return diffLists(list1, list2, idColumn);
function diffLists(listA, listB, key) {
    const mapA = new Map(listA.map(item => [item[key], item]));
    const mapB = new Map(listB.map(item => [item[key], item]));
    const updatedItems = [];
    for (const [id, itemB] of mapB) {
        const itemA = mapA.get(id);
        if (!itemA || JSON.stringify(itemA) !== JSON.stringify(itemB)) {
            updatedItems.push(itemB);
        }
    }
    return updatedItems;
}
```

6. Drag a *SetValue* action into the Global Script and place it under the *JavaScript* action
   1. Target: = ~.Parameters.Output.Return
   2. Value: = ~.Javascript

## Usage
1. Drag the script called "GetChangedListItems" into a script or event handler
2. Enter values for the script input parameters
   1. IDColumn: The name of an object that uniquely identifies an item in the two lists (e.g. ID)
   2. OriginalList1
   3. UpdatedList
4. Result: The script returns a List of items that were added or changed in the UpdatedList compared to the OriginalList.