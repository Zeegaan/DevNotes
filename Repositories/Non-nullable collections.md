# Returning lists as null or empty?
I had an interesting discussion with Andy Butland about this, and we both agreed that when retrieving a list from a method, do we really expect that to ever be null? 
The thought it, that even if we call the method and there was no elements, the list would just be empty, never null. 

By extension, we return of IEnumerable<T> where we really should be returning IReadOnlyList<T> instead, in this discussion interesting article on the subject is mentioned: https://github.com/umbraco/Umbraco-CMS/pull/12071