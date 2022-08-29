# What even is an index? 
And index works alot like a dictionary, where we have a Key and some value attached to that field.

## Removing/Adding fields to the index

See index for more information about hooking into the IndexProviderTransformingValues event.

When hooking into the even, we can get out the valueset, and then add/remove entries in the dictionary, by using `updatedValues.Add()`
Remember to use `e.SetValues` to


```csharp
var updatedValues = e.ValueSet.Values.ToDictionary(x => x.Key, x => x.Value.ToList());  
string[] alias = new string[1] {"blog"};  
string blogTypeId = _contentTypeService.GetAllContentTypeIds(alias).FirstOrDefault().ToString();  
updatedValues.TryGetValue("nodeType", out var nType);  
updatedValues.TryGetValue("publishingDate", out var publishingDate);  
updatedValues.TryGetValue("createDate", out var fallbackDate);  
//good to know - dates are saved in DateTime.Ticks  
//we only do this on Blog doctype  
if (nType.FirstOrDefault().ToString() == blogTypeId)  
{  
    if (publishingDate == null)  
    {        updatedValues.Remove("publishingDate");  
        //cleared the publishingDate value, let's add one that is filled out  
        updatedValues.Add("publishingDate", fallbackDate);  
    }    else  
    {  
        updatedValues.Remove("publishingDate");  
        updatedValues.Add("publishingDate", publishingDate);  
    }  
    e.SetValues(updatedValues.ToDictionary(x => x.Key, x => (IEnumerable<object>) x.Value));  
}
```


