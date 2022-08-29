# Searching
Searching is done by taking the index and using the Searcher to find some content based on criteria.

First, we have to inject the `IExamineManager` into our service, and we will then get the index like so:

```csharp
if (_examineManager.TryGetIndex(Constants.UmbracoIndexes.ExternalIndexName, out var index))  
{
	
}
```
In the example we now have the External index as the var `index` and we can now use that to the the searcher with `var searcher = index.Searcher`

An example of a serach:
```csharp
totalItemCount = 0;  
if (_examineManager.TryGetIndex(Constants.UmbracoIndexes.ExternalIndexName, out var index))  
{  
    var searcher = index.Searcher;  
    var fieldToSearchLang = "contents" + "_" + CultureInfo.CurrentCulture.ToString().ToLower();  
    var fieldToSearchInvariant = "contents";  
    var fieldsToSearch = new[] {fieldToSearchLang, fieldToSearchInvariant};  
    var hideFromNavigation = "umbracoNaviHide";  
    var criteria = searcher.CreateQuery(IndexTypes.Content);  
    var examineQuery = criteria.GroupedOr(fieldsToSearch, searchTerm.MultipleCharacterWildcard());  
    examineQuery.Not().Field(hideFromNavigation, 1.ToString());  
    var results = examineQuery.Execute();  
    totalItemCount = results.TotalItemCount;  
    if (results.Any())  
    {        return results;  
    }  
    Console.WriteLine("Error");  
}
```
Do note here that we are using the `serachTerm.MultipleCharacterWildcard` to add wildcard searching, IE. if we search for `hous`, we will still hit any entry with `house`

# Searching multiple indexes using a multi-searcher
You do not have to only search 1 index though, in most of the examples I've used we're actually only searching on the ExternalIndex, but we can actually search on multiple indexes at once!

We do this by adding a multi searcher, we just need to register it in our Startup.cs like so:
`services.AddExamineLuceneMultiSearcher("MultiSearcher", new[] { Constants.UmbracoIndexes.ExternalIndexName, PdfIndexConstants.PdfIndexName });`
What that line does is tell the runtime, what indexes this searcher will traverse. In the example, it will traverse both the `ExternalIndex` and the `PdfIndex`
Just remember if you are trying to fiddle with the indexesd via `ConfigureCustomFieldOptions` or doing searching on different fields via the `IndexProviderTransformingIndexValues`
That you have to add `[ComposeAfter(typeof(ExaminePdfComposer))]` to your composer!
(if using other external indexes from packages)

Finally instead of getting the index, and using its searcher, we now have created our own searcher with already defined indexes!


# Different kinds of searching
***For more information about searching, look at the examine docs: [Searching (shazwazza.github.io)](https://shazwazza.github.io/Examine/searching)

## Fuzzy
What is fuzzy searching? Fuzzy searching is the matching of a word even if its misspelled slightly, for example, maybe if someone searches for `Umbrako`, we probably still want them to find results that have `Umbraco`
Fuzzy searching can be done by using the extention method `searchTerm.Fuzzy(0.2f)`, here we apply a 0.2f float value, and this might seem wierd, what does this number even do?
This number determines how much a word can be "off", but do be careful! High numbers might lead to false searches. 
So for low values `Unbrako` probably won't generate any results, but if we tweak the values higher, we might start to hit results that contain `Umbraco`


## Wildcard
What is wildcard searching? Wildcard searching is for example if you search for `Umbrac`, we can add a wildcard and you will still get the `Umbraco`  results.

## Boosting
https://www.cludo.com/blog/boosting-important-websites-success/#:~:text=Boosting%20is%20the%20process%20of,certain%20areas%20of%20your%20website.

# DO NOT DO FIRTHER FILTERING OF RESULTS USING LINQ
The Examine search results Object implements its own take and skip. Also, the search method has overrides. One override allows you to set the max number of results to send back. Use these methods and not LINQ. Further to this DO NOT DO FURTHER FILTERING OF RESULTS USING LINQ for example:

```csharp
// Do not do this 
var results = examineQuery.Execute(); 
results
.Where(x=>x.Values["nodeName"]=="whatever")
.OrderBy("nodeName")
.Take(10);
```

Another way to view this, is that searching IS filtering, and therefore your search should contain all your filtering, and should not be applied afterwards.