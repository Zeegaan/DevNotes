# Index rebuilding
It is never the intention that a content-editor has to rebuild indexes manually, thereby if you update content (or even document types) we rebuild indexes for you.
If you want to customize your index, IE. hooking into the IndexProviderTransformingValues event though, you will have to rebuild the indexes manually (or of course, trigger a rebuild) so that your code actually runs.

# When working with TransformingIndexValues
When working with TransformingIndexValues you may need to get content from a node e.g you have multi node tree picker property and you want to get some values from those picked items to inject into your index. 
You can try and instantiate an Umbraco helper object in your TransformingIndexValues data class however you will get a null exception error. 
There is then a temptation to use ContentService to get these values. 
The problem here is that ContentService makes hits to the database. So imagine you have 30k blog posts each one has 3 or more tags. 
If you use ContentService you will potentially make 2*30k database hits during a full re-index. This will potentially bring your site down.
It will definitely give you index rebuild times of >20 minutes. If you are on Azure this can be a major problem because Azure web apps will swap file stores and this triggers index rebuilds. 
So always use IPublishedContent - the code sample below illustrates how to do this:

```csharp
private void IndexProviderTransformingIndexValues(object sender, IndexingItemEventArgs e) 
{
	using (var umbCtxRef = _umbracoContextFactory.EnsureUmbracoContext())
		{ 
			var contentCache = umbCtxRef.UmbracoContext?.ContentCache;
			if (contentCache == null) throw new InvalidOperationException("Could not acquire content cache"); 
			AddSearchFields(sender, e, umbCtxRef.UmbracoContext?.ContentCache);
		} 
} 
private void AddSearchFields(object sender, IndexingItemEventArgs args, IPublishedContentC ache contentCache) 
{ 
	if(contentCache == null) throw new ArgumentNullException(nameof(contentCache));
	var item = contentCache.GetById(1234);
}
```
