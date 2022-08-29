**Value Sets are immutable in v10

# What are value sets
Value sets are the value sets of the given index.

## Categories
Value sets also have categories, the different categories can be found under the `IndexTypes` constant, the different IndexTypes are:
- `IndexTypes.Content`
- `IndexTypes.Media`
- `IndexTypes.Member`

# Handling variant content with value sets
To handle variant & invariant content at the same time, we first have to know, if the content can vary by culture at all. We can do this by running a check like this:
```csharp
string? variesByCulture = null;
if (e.ValueSet.Values.TryGetValue("__VariesByCulture", out var result))  
{  
    variesByCulture = (string) result[0];  
}
```
So our string `variesByCulture` will have a value if the content varies by culture, and will be `null` otherwise.
But what value will the string have if it actually does vary by culture? Is it safe to assume that if its not null, it varies by culture? Well probably, but if it varies by culture, we actually have a specific value `y` , so we can run a check like:
```csharp
if (variesByCulture is "y")  
{
	// Handle if it varies by culture
}
else
{
	// Handle if it does not vary
}
```

# What is a value set?
A value set is a dictionary that holds the different things for an object, lets take the `ToDoModel` from the Custom indexing section and see how we'd create a value set using that model:
```csharp
var indexValues = new Dictionary<string, object>  
{  
    ["userId"] = todo.UserId,  
    ["id"] = todo.Id,  
    ["title"] = todo.Title,  
    ["completed"] = todo.Completed  
};  
var valueSet = new ValueSet(todo.Id.ToString(), "todo", indexValues);
```