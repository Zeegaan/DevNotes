# 3 different language folders
In Umbraco we have some wierd conventions for language files, as in they can actually be placed in 3 different folders and still work just fine.

The placements are:
- /App_Plugins/Test/lang/en-us.xml
- /config/lang/en-us.user.xml
- /wwwroot/app_plugins/text/lang/en-us.xml

Here is Kevin Jumps PR where he mentions all 3: 
https://github.com/umbraco/Umbraco-CMS/pull/12206