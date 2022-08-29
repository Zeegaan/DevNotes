# Examine Component Code
```csharp
if (variesByCulture is "y")  
{  
    // Loop through each language, look at all fields and aggregate them in the contents_ fields  
    foreach (var language in languages)  
    {        var languageIsoCode = language.IsoCode.ToLower();  
        var cultureAndInvariantFields = GetCultureAndInvariantFields(updatedValues, languageIsoCode);  
        var combinedFieldsLang = new StringBuilder();  
        foreach (var field in cultureAndInvariantFields.Where(x => !x.StartsWith("contents") && !x.StartsWith("__Raw")))  
        {            updatedValues.TryGetValue(field, out List<object>? values);  
            if (values != null)  
            {                foreach (var value in values)  
                {                    combinedFieldsLang.AppendLine(value.ToString());  
                }            }        }  
        updatedValues.Add("contents_" + languageIsoCode, new List<object> {combinedFieldsLang.ToString()});  
        e.SetValues(updatedValues.ToDictionary(x => x.Key, x => (IEnumerable<object>) x.Value));  
    }}
```