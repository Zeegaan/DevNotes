# Custom indexes

## What is needed for creating custom indexes?
You need some data, for training purposes, we just use a json feed, but this could be from anywhere, a PIM, DAM, etc.. https://jsonplaceholder.typicode.com/todos/

Then what we need to create is:
- A `UmbracoExamineIndex` - This will be our actual index, the class has no logic other than deriving from UmbracoExamineIndex
- A `ConfigureNamedOptions` - We have created a blank index, now we need to actually configure it with fields and types, we do that in this class
- An `IndexPopulator` - We have created a index with fields etc. now we actually need to fill out the index with actual data, we do that with an index populator
- A `ValueSetBuilder` - This builds all the value sets for the index, this will be called by the index populator to fill out the actual index with value sets.
- A model with our data

(Remember to register these as services in a composer)
## The model
For the model,  we've created a timople `ToDoModel`:
```csharp
public class ToDoModel  
{  
    public int UserId { get; set; }  
    public int Id { get; set; }  
    public string Title { get; set; }  
    public bool Completed { get; set; }  
}
```

## ValueSetBuilder
```csharp
public class TodoValueSetBuilder : IValueSetBuilder<ToDoModel>  
{  
    public IEnumerable<ValueSet> GetValueSets(params ToDoModel[] data)  
    {   foreach (var todo in data)  
        {        
	        var indexValues = new Dictionary<string, object>  
            {                
            ["userId"] = todo.UserId,  
                ["id"] = todo.Id,  
                ["title"] = todo.Title,  
                ["completed"] = todo.Completed  
            };  
            var valueSet = new ValueSet(todo.Id.ToString(), "todo", indexValues);  
            yield return valueSet;  
        }    
    }
}
```
## IndexPopulator
```csharp
public class TodoIndexPopulator : IndexPopulator  
{  
    private readonly TodoValueSetBuilder _todoValueSetBuilder;  
  
    public TodoIndexPopulator(TodoValueSetBuilder productValueSetBuilder)  
    {        _todoValueSetBuilder = productValueSetBuilder;  
        //We're telling this populator that it's responsible for populating only our index  
        RegisterIndex("TodoIndex");  
    }  
    protected override void PopulateIndexes(IReadOnlyList<IIndex> indexes)  
    {        using (WebClient httpClient = new WebClient())  
        {            var jsonData = httpClient.DownloadString("https://jsonplaceholder.typicode.com/todos/");  
            var data = JsonConvert.DeserializeObject<IEnumerable<ToDoModel>>(jsonData);  
            if (data != null)  
            {
                foreach (var item in indexes)  
                {                    
	                item.IndexItems(_todoValueSetBuilder.GetValueSets(data.ToArray()));  
                }            
            }        
        }    
    }
}
```

## ConfigureNamedOptions

```csharp
public class ConfigureCustomIndexOptions : IConfigureNamedOptions<LuceneDirectoryIndexOptions>  
{  
    private readonly IOptions<IndexCreatorSettings> _settings;  
  
    public ConfigureCustomIndexOptions(IOptions<IndexCreatorSettings> settings)  
    {        _settings = settings;  
    }  
    public void Configure(string name, LuceneDirectoryIndexOptions options)  
    {        
	    if (name.Equals("TodoIndex"))  
        {           
	        options.Analyzer = new StandardAnalyzer(LuceneVersion.LUCENE_48);  
            options.FieldDefinitions = new FieldDefinitionCollection(  
                new FieldDefinition("userID", FieldDefinitionTypes.Integer),  
                new FieldDefinition("id", FieldDefinitionTypes.Integer),  
                new FieldDefinition("title", FieldDefinitionTypes.FullTextSortable),  
                new FieldDefinition("completed", FieldDefinitionTypes.FullTextSortable  
                ));  
            options.UnlockIndex = true;  
            if (_settings.Value.LuceneDirectoryFactory == LuceneDirectoryFactory.SyncedTempFileSystemDirectoryFactory)  
            {                // if this directory factory is enabled then a snapshot deletion policy is required  
                options.IndexDeletionPolicy = new SnapshotDeletionPolicy(new KeepOnlyLastCommitDeletionPolicy());  
            }        
        }    
    }  
    public void Configure(LuceneDirectoryIndexOptions options)  
    {        
	    throw new System.NotImplementedException("This is never called and is just part of the interface");  
    }
}
```

## UmbracoExamineIndex

```csharp
public class CustomToDoIndex : UmbracoExamineIndex  
{  
    public CustomToDoIndex(  
        ILoggerFactory loggerFactory,  
        string name,  
        IOptionsMonitor<LuceneDirectoryIndexOptions> indexOptions,  
        Umbraco.Cms.Core.Hosting.IHostingEnvironment hostingEnvironment,  
        IRuntimeState runtimeState)  
        : base(loggerFactory, name, indexOptions, hostingEnvironment, runtimeState)  
    {    
    }
}
```