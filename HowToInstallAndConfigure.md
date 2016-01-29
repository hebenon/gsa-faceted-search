# How does it work #

All you need to add is a JSON object that defines the headings, facets and filter criteria. Take a look at [this screen shot](http://gsa-faceted-search.googlecode.com/files/screen-shot-2.gif) and the JSON code, which should be self explanatory. This JSON code is part of code-snippet-1 and needs to be modified according to your requirements.

The rest of the Java Script code will then automatically create a faceted navigation with the respective search queries for the user. You can also get the number of documents that are behind a specific facet if you enable this feature. Please consider the section on performance and statistics in that case.

**Note**: this module does not work in MSIE 6 or earlier.

# How to install #

In order to integrate this module into a plain front end on your GSA create a new front end. You only need to add code on three different places in the XSLT code of a front end:

### Step one ###
Find this code
```
<!-- **********************************************************************
 Search results (do not customize)
     ********************************************************************** -->
<xsl:template name="search_results">
<html>

  <!-- *** HTML header and style *** -->
  <xsl:call-template name="langHeadStart"/>
    <xsl:call-template name="redirect_if_few_results"/>
    <title><xsl:value-of select="$result_page_title"/>:
      <xsl:value-of select="$space_normalized_query"/>
    </title>
    <xsl:call-template name="style"/>
```
Then add [code-snippet-1](http://gsa-faceted-search.googlecode.com/files/code-snippet-1.1.txt) right beneath the last line of this code.

### Step 2 ###
Find this code
```
function resetForms() {
    for (var i = 0; i &lt; document.forms.length; i++ ) {
        document.forms[i].reset();
    }
```
Then add [code-snippet-2](http://gsa-faceted-search.googlecode.com/files/code-snippet-2.txt) right beneath the last line of this code (yes, within the resetForms() function).
### Step 3 ###
Find this code
```
<!-- *** Output results details *** -->
```
Then add [code-snippet-3](http://gsa-faceted-search.googlecode.com/files/code-snippet-3.txt) right beneath this line of code.

# How to configure #
You have two options for defining a facet, which is in fact nothing more than a filter expression:
### 1. Use a filter that is reflected through a [search parameter](http://code.google.com/apis/searchappliance/documentation/60/xml_reference.html#request_parameters) ###
In that case add a line that looks like this:

`{"name":"My Facet", "select":"site=ak_reports"},`

As select statement you can contain any of the query parameters that the GSA supports. Here are some examples:

`{"name":"My Facet", "select":"site=ak_reports"},`  // selects all results that are part of the collection "ak\_reports"; if you use collections as facets you should add the name of an an "overall collection" so that the user can "reset" this filter.

`{"name":"My Facet", "select":"lr=lang_fr"},`  // selects all results that are in French; consider the [documentation](http://code.google.com/apis/searchappliance/documentation/60/xml_reference.html#request_subcollections) for valid language codes

`{"name":"My Facet", "select":"requiredfields=author"},`  // selects all results that contain a meta tag named "author"

`{"name":"My Facet", "select":"requiredfields=author:john%2520smith"},`  // selects all results that contain a meta tag named "author" where the value is exactly "John Smith"; according to the [documentation](http://code.google.com/apis/searchappliance/documentation/60/xml_reference.html#request_meta_filter) the given value must be double URL-encoded

`{"name":"My Facet", "select":"partialfields=author:smith"},`  // selects all results that contain a meta tag named "author" where the value contains "smith"

Except for the site parameter a reset button will appear besides the facet after the user has selected a facet. Please note that only one selection per query term can be made at once. This is e.g. after the user has selected the facet that filters only French results, any previous language filters will be canceled - makes sense, doesn't it.

### 2. Use a [special query term](http://code.google.com/apis/searchappliance/documentation/60/xml_reference.html#SpecialQueryTerms) ###
You can use a special query term as documented by Google. If you choose this option you must use the as\_q parameter followed by the special query term and its desired value. Example:

`{"name":"My Facet", "select":"as_q=filetype:pdf"},`  // selects all results that are PDF files

Note that this module will remove the query term from the search input field in order to avoid that users are confused.

# About performance, precision and search statistics (search reports) #
This module will not add any additional load to you GSA by default. In case you need to display the number of search results per facet you can enable the **display doc count feature** by removing the two slashes from the line

`//          docCount = getDocCount(facetURL); // remove the two slashes at the beginning of this line if you want to display the number of documents for each facet`

The code will then submit an additional query per facet and obviously multiply the load to the GSA by the number of facets you have defined. So please be careful with this option in case you expect more than one user query per second. Please note that the displayed doc count number is just an approximate value and not a 100% reliable.

If you enable the display doc count feature, you might also notice that your **search reports are distorted**. This is also a side effect of the module sending multiple requests per user query to the GSA. You can solve this by utilizing the the **collection alias feature**. Do so by modifying the line and changing the name of the variable to whatever suffix you prefer.

`var collectionAliasSuffix = "_alias"; // set this variable to "" if you do not want to use this feature;`

You must also add new collections within you GSA where the name of these collection is just like the collections in use but with the collection alias suffix you have chosen.

Example:
You use the collection named `default_collection` and for more facets the collections named `reports`, `wiki` and `customers`. The value of the variable collectionAliasSuffix is `_alias`. Hence you have to add the collections `default_collection_alias`, `reports_alias`, `wiki_alias` and `customers_alias`. Then your search reports won't be distorted any more since the module will only send queries to the alias collections.