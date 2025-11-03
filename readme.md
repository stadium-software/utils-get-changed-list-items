# Get Changed List Items

Pass in an original List of objects and an updated List and get back the items that were added or changed.

This has been tested with simple lists of objects, not with deep lists. See the sample application to test it with your data.

# Version 

1.1 - improved date comparisons logic

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
/* Stadium Script v1.1 https://github.com/stadium-software/utils-get-changed-list-items */
let originalList = ~.Parameters.Input.OriginalList;
let updatedList = ~.Parameters.Input.UpdatedList;
let key = ~.Parameters.Input.IDColumn;
return getChangedListItems(originalList, updatedList, key);
function getChangedListItems(originalList, updatedList, key) {
    const changedItems = [];
    const addedItems = [];
    const removedItems = [];
    const original = Array.isArray(originalList) ? originalList : [];
    const updated = Array.isArray(updatedList) ? updatedList : [];
    const mapOriginal = new Map(original.map((item) => [item[key], item]));
    const mapUpdated = new Map(updated.map((item) => [item[key], item]));
    for (const [id, origItem] of mapOriginal.entries()) {
        const updatedItem = mapUpdated.get(id);
        if (!updatedItem) {
            removedItems.push(origItem);
            continue;
        }
        if (hasDifferences(origItem, updatedItem)) {
            changedItems.push(updatedItem);
        }
    }
    for (const [id, updItem] of mapUpdated.entries()) {
        if (!mapOriginal.has(id)) {
            addedItems.push(updItem);
        }
    }
    return {
        changed: changedItems,
        added: addedItems,
        removed: removedItems,
        allDifferences: [...changedItems, ...addedItems, ...removedItems],
    };
}

function hasDifferences(a, b) {
    const dayjsFormat = "YYYY-MM-DDTHH:mm:ss.SSSZ";
    const keys = new Set([...Object.keys(a), ...Object.keys(b)]);
    for (const key of keys) {
        const valA = a[key];
        const valB = b[key];
        if (valA == null && valB == null) continue;
        if (valA == null || valB == null) return true;
        if (isDateLike(valA) && isDateLike(valB)) {
            const d1 = dayjs(valA).format(dayjsFormat);
            const d2 = dayjs(valB).format(dayjsFormat);
            if (d1 !== d2) return true;
            continue;
        }
        if (typeof valA === "object" || typeof valB === "object") {
            if (JSON.stringify(valA) !== JSON.stringify(valB)) return true;
            continue;
        }
        if (valA !== valB) return true;
    }
    return false;
}

function isDateLike(value) {
    return value instanceof Date || (typeof value === "string" && dayjs(value).isValid());
}
```

6. Drag a *SetValue* action into the Global Script and place it under the *JavaScript* action
   1. Target: = ~.Parameters.Output.Return
   2. Value: = ~.Javascript

## Usage
1. Drag the script called "GetChangedListItems" into a script or event handler
2. Enter values for the script input parameters
   1. IDColumn: The name of an object that uniquely identifies an item in the two lists (e.g. ID)
   2. OriginalList: The original List of items
   3. UpdatedList: The updated List of items
4. Result: The script returns four Lists of items
   1. changed: a list of items where some values were changed
   2. added: a list of items that are in the updated list but not in the original list
   3. removed: a list of items that are in the original list but not in the updated list
   4. allDifferences: a list that contains all items that were added, changed or removed
5. To access the returned lists, use the following syntax (see sample application):
   1. = ~.GetChangedListItems.Return.changed
   2. = ~.GetChangedListItems.Return.added
   3. = ~.GetChangedListItems.Return.removed
   4. = ~.GetChangedListItems.Return.allDifferences

## Notes

The format of dates is changed when it is assigned to a DatePicker control and then read back into a list. The script returns all dates in Lists in "YYYY-MM-DDTHH:mm:ss.SSSZ" format.