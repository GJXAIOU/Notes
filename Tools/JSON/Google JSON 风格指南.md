## Important Note

### Display Hidden Details in this Guide

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=Display_Hidden_Details_in_this_Guide#Display_Hidden_Details_in_this_Guide)▽

This style guide contains many details that are initially hidden from view. They are marked by the triangle icon, which you see here on your left. Click it now. You should see "Hooray" appear below.

Hooray! Now you know you can expand points to get more details. Alternatively, there's an "expand all" at the top of this document.

## Introduction

This style guide documents guidelines and recommendations for building JSON APIs at Google. In general, JSON APIs should follow the spec found at [JSON.org](http://www.json.org/). This style guide clarifies and standardizes specific cases so that JSON APIs from Google have a standard look and feel. These guidelines are applicable to JSON requests and responses in both RPC-based and REST-based APIs.

## Definitions

For the purposes of this style guide, we define the following terms:

- property - a name/value pair inside a JSON object.
- property name - the name (or key) portion of the property.
- property value - the value portion of the property.

```
{
  // The name/value pair together is a "property".
  "propertyName": "propertyValue"
}
```

Javascript's `number` type encompasses all floating-point numbers, which is a broad designation. In this guide, `number` will refer to JavaScript's `number` type, while `integer` will refer to integers.

## General Guidelines

### Comments

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=Comments#Comments)▽

No comments in JSON objects.

Comments should not be included in JSON objects. Some of the examples in this style guide include comments. However this is only to clarify the examples.

```
{
  // You may see comments in the examples below,
  // But don't include comments in your JSON.
  "propertyName": "propertyValue"
}
```

### Double Quotes

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=Double_Quotes#Double_Quotes)▽

Use double quotes.

If a property requires quotes, double quotes must be used. All property names must be surrounded by double quotes. Property values of type string must be surrounded by double quotes. Other value types (like boolean or number) should not be surrounded by double quotes.

### Flattened data vs Structured Hierarchy

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=Flattened_data_vs_Structured_Hierarchy#Flattened_data_vs_Structured_Hierarchy)▽

Data should not be arbitrarily grouped for convenience.

Data elements should be "flattened" in the JSON representation. Data should not be arbitrarily grouped for convenience.

In some cases, such as a collection of properties that represents a single structure, it may make sense to keep the structured hierarchy. These cases should be carefully considered, and only used if it makes semantic sense. For example, an address could be represented two ways, but the structured way probably makes more sense for developers:

Flattened Address:

```
{
  "company": "Google",
  "website": "https://www.google.com/",
  "addressLine1": "111 8th Ave",
  "addressLine2": "4th Floor",
  "state": "NY",
  "city": "New York",
  "zip": "10011"
}
```

Structured Address:

```
{
  "company": "Google",
  "website": "https://www.google.com/",
  "address": {
    "line1": "111 8th Ave",
    "line2": "4th Floor",
    "state": "NY",
    "city": "New York",
    "zip": "10011"
  }
}
```

## Property Name Guidelines

### Property Name Format

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=Property_Name_Format#Property_Name_Format)▽

Choose meaningful property names.

Property names must conform to the following guidelines:

- Property names should be meaningful names with defined semantics.
- Property names must be camel-cased, ascii strings.
- The first character must be a letter, an underscore (_) or a dollar sign ($).
- Subsequent characters can be a letter, a digit, an underscore, or a dollar sign.
- Reserved JavaScript keywords should be avoided (A list of reserved JavaScript keywords can be found below).

These guidelines mirror the guidelines for naming JavaScript identifiers. This allows JavaScript clients to access properties using dot notation. (for example, `result.thisIsAnInstanceVariable`). Here's an example of an object with one property:

```
{
  "thisPropertyIsAnIdentifier": "identifier value"
}
```

### Key Names in JSON Maps

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=Key_Names_in_JSON_Maps#Key_Names_in_JSON_Maps)▽

JSON maps can use any Unicode character in key names.

The property name naming rules do not apply when a JSON object is used as a map. A map (also referred to as an associative array) is a data type with arbitrary key/value pairs that use the keys to access the corresponding values. JSON objects and JSON maps look the same at runtime; this distinction is relevant to the design of the API. The API documentation should indicate when JSON objects are used as maps.

The keys of a map do not have to obey the naming guidelines for property names. Map keys may contain any Unicode characters. Clients can access these properties using the square bracket notation familiar for maps (for example, `result.thumbnails["72"]`).

```
{
  // The "address" property is a sub-object
  // holding the parts of an address.
  "address": {
    "addressLine1": "123 Anystreet",
    "city": "Anytown",
    "state": "XX",
    "zip": "00000"
  },
  // The "thumbnails" property is a map that maps
  // a pixel size to the thumbnail url of that size.
  "thumbnails": {
    "72": "http://url.to.72px.thumbnail",
    "144": "http://url.to.144px.thumbnail"
  }
}
```

### Reserved Property Names

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=Reserved_Property_Names#Reserved_Property_Names)▽

Certain property names are reserved for consistent use across services.

Details about reserved property names, along with the full list, can be found later on in this guide. Services should avoid using these property names for anything other than their defined semantics.

### Singular vs Plural Property Names

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=Singular_vs_Plural_Property_Names#Singular_vs_Plural_Property_Names)▽

Array types should have plural property names. All other property names should be singular.

Arrays usually contain multiple items, and a plural property name reflects this. An example of this can be seen in the reserved names below. The `items` property name is plural because it represents an array of item objects. Most of the other fields are singular.

There may be exceptions to this, especially when referring to numeric property values. For example, in the reserved names, `totalItems` makes more sense than `totalItem`. However, technically, this is not violating the style guide, since `totalItems` can be viewed as `totalOfItems`, where `total` is singular (as per the style guide), and `OfItems` serves to qualify the total. The field name could also be changed to `itemCount` to look singular.

```
{
  // Singular
  "author": "lisa",
  // An array of siblings, plural
  "siblings": [ "bart", "maggie"],
  // "totalItem" doesn't sound right
  "totalItems": 10,
  // But maybe "itemCount" is better
  "itemCount": 10,
}
```

### Naming Conflicts

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=Naming_Conflicts#Naming_Conflicts)▽

Avoid naming conflicts by choosing a new property name or versioning the API.

New properties may be added to the reserved list in the future. There is no concept of JSON namespacing. If there is a naming conflict, these can usually be resolved by choosing a new property name or by versioning. For example, suppose we start with the following JSON object:

```
{
  "apiVersion": "1.0",
  "data": {
    "recipeName": "pizza",
    "ingredients": ["tomatoes", "cheese", "sausage"]
  }
}
```

If in the future we wish to make `ingredients` a reserved word, we can do one of two things:

\1) Choose a different name:

```
{
  "apiVersion": "1.0",
  "data": {
    "recipeName": "pizza",
    "ingredientsData": "Some new property",
    "ingredients": ["tomatoes", "cheese", "sausage"]
  }
}
```

\2) Rename the property on a major version boundary:

```
{
  "apiVersion": "2.0",
  "data": {
    "recipeName": "pizza",
    "ingredients": "Some new property",
    "recipeIngredients": ["tomatos", "cheese", "sausage"]
  }
}
```

## Property Value Guidelines

### Property Value Format

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=Property_Value_Format#Property_Value_Format)▽

Property values must be Unicode booleans, numbers, strings, objects, arrays, or `null`.

The spec at [JSON.org](http://www.json.org/) specifies exactly what type of data is allowed in a property value. This includes Unicode booleans, numbers, strings, objects, arrays, and `null`. JavaScript expressions are not allowed. APIs should support that spec for all values, and should choose the data type most appropriate for a particular property (numbers to represent numbers, etc.).

Good:

```
{
  "canPigsFly": null,     // null
  "areWeThereYet": false, // boolean
  "answerToLife": 42,     // number
  "name": "Bart",         // string
  "moreData": {},         // object
  "things": []            // array
}
```

Bad:

```
{
  "aVariableName": aVariableName,         // Bad - JavaScript identifier
  "functionFoo": function() { return 1; } // Bad - JavaScript function
}
```

### Empty/Null Property Values

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=Empty/Null_Property_Values#Empty/Null_Property_Values)▽

Consider removing empty or `null` values.

If a property is optional or has an empty or `null` value, consider dropping the property from the JSON, unless there's a strong semantic reason for its existence.

```
{
  "volume": 10,

  // Even though the "balance" property's value is zero, it should be left in,
  // since "0" signifies "even balance" (the value could be "-1" for left
  // balance and "+1" for right balance.
  "balance": 0,

  // The "currentlyPlaying" property can be left out since it is null.
  // "currentlyPlaying": null
}
```

### Enum Values

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=Enum_Values#Enum_Values)▽

Enum values should be represented as strings.

As APIs grow, enum values may be added, removed or changed. Using strings as enum values ensures that downstream clients can gracefully handle changes to enum values.

Java code:

```
public enum Color {
  WHITE,
  BLACK,
  RED,
  YELLOW,
  BLUE
}
```

JSON object:

```
{
  "color": "WHITE"
}
```

## Property Value Data Types

As mentioned above, property value types must be booleans, numbers, strings, objects, arrays, or `null`. However, it is useful define a set of standard data types when dealing with certain values. These data types will always be strings, but they will be formatted in a specific manner so that they can be easily parsed.

### Date Property Values

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=Date_Property_Values#Date_Property_Values)▽

Dates should be formatted as recommended by RFC 3339.

Dates should be strings formatted as recommended by [RFC 3339](https://www.ietf.org/rfc/rfc3339.txt)

```
{
  "lastUpdate": "2007-11-06T16:34:41.000Z"
}
```

### Time Duration Property Values

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=Time_Duration_Property_Values#Time_Duration_Property_Values)▽

Time durations should be formatted as recommended by ISO 8601.

Time duration values should be strings formatted as recommended by [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Durations).

```
{
  // three years, six months, four days, twelve hours,
  // thirty minutes, and five seconds
  "duration": "P3Y6M4DT12H30M5S"
}
```

### Latitude/Longitude Property Values

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=Latitude/Longitude_Property_Values#Latitude/Longitude_Property_Values)▽

Latitudes/Longitudes should be formatted as recommended by ISO 6709.

Latitude/Longitude should be strings formatted as recommended by [ISO 6709](https://en.wikipedia.org/wiki/ISO_6709). Furthermore, they should favor the ±DD.DDDD±DDD.DDDD degrees format.

```
{
  // The latitude/longitude location of the statue of liberty.
  "statueOfLiberty": "+40.6894-074.0447"
}
```

## JSON Structure & Reserved Property Names

In order to maintain a consistent interface across APIs, JSON objects should follow the structure outlined below. This structure applies to both requests and responses made with JSON. Within this structure, there are certain property names that are reserved for specific uses. These properties are NOT required; in other words, each reserved property may appear zero or one times. But if a service needs these properties, this naming convention is recommend. Here is a schema of the JSON structure, represented in [Orderly](https://www.google.com/url?sa=D&q=http%3A%2F%2Forderly-json.org%2F) format (which in turn can be compiled into a [JSONSchema](https://www.google.com/url?sa=D&q=http%3A%2F%2Fjson-schema.org%2F)). You can few examples of the JSON structure at the end of this guide.

```
object {
  string apiVersion?;
  string context?;
  string id?;
  string method?;
  object {
    string id?
  }* params?;
  object {
    string kind?;
    string fields?;
    string etag?;
    string id?;
    string lang?;
    string updated?; # date formatted RFC 3339
    boolean deleted?;
    integer currentItemCount?;
    integer itemsPerPage?;
    integer startIndex?;
    integer totalItems?;
    integer pageIndex?;
    integer totalPages?;
    string pageLinkTemplate /^https?:/ ?;
    object {}* next?;
    string nextLink?;
    object {}* previous?;
    string previousLink?;
    object {}* self?;
    string selfLink?;
    object {}* edit?;
    string editLink?;
    array [
      object {}*;
    ] items?;
  }* data?;
  object {
    integer code?;
    string message?;
    array [
      object {
        string domain?;
        string reason?;
        string message?;
        string location?;
        string locationType?;
        string extendedHelp?;
        string sendReport?;
      }*;
    ] errors?;
  }* error?;
}*;
```

The JSON object has a few top-level properties, followed by either a `data` object or an `error` object, but not both. An explanation of each of these properties can be found below.

## Top-Level Reserved Property Names

The top-level of the JSON object may contain the following properties.

### apiVersion

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=apiVersion#apiVersion)▽

Property Value Type: string
Parent: -

Represents the desired version of the service API in a request, and the version of the service API that's served in the response. `apiVersion` should always be present. This is not related to the version of the data. Versioning of data should be handled through some other mechanism such as etags.

Example:

```
{ "apiVersion": "2.1" }
```

### context

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=context#context)▽

Property Value Type: string
Parent: -

Client sets this value and server echos data in the response. This is useful in JSON-P and batch situations , where the user can use the `context` to correlate responses with requests. This property is a top-level property because the `context` should present regardless of whether the response was successful or an error. `context` differs from `id` in that `context` is specified by the user while `id` is assigned by the service.

Example:

Request #1:

```
https://www.google.com/myapi?context=bart
```

Request #2:

```
https://www.google.com/myapi?context=lisa
```

Response #1:

```
{
  "context": "bart",
  "data": {
    "items": []
  }
}
```

Response #2:

```
{
  "context": "lisa",
  "data": {
    "items": []
  }
}
```

Common JavaScript handler code to process both responses:

```
function handleResponse(response) {
  if (response.result.context == "bart") {
    // Update the "Bart" section of the page.
  } else if (response.result.context == "lisa") {
    // Update the "Lisa" section of the page.
  }
}
```

### id

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=id#id)▽

Property Value Type: string
Parent: -

A server supplied identifier for the response (regardless of whether the response is a success or an error). This is useful for correlating server logs with individual responses received at a client.

Example:

```
{ "id": "1" }
```

### method

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=method#method)▽

Property Value Type: string
Parent: -

Represents the operation to perform, or that was performed, on the data. In the case of a JSON request, the `method` property can be used to indicate which operation to perform on the data. In the case of a JSON response, the `method` property can indicate the operation performed on the data.

One example of this is in JSON-RPC requests, where `method` indicates the operation to perform on the `params` property:

```
{
  "method": "people.get",
  "params": {
    "userId": "@me",
    "groupId": "@self"
  }
}
```

### params

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=params#params)▽

Property Value Type: object
Parent: -

This object serves as a map of input parameters to send to an RPC request. It can be used in conjunction with the `method` property to execute an RPC function. If an RPC function does not need parameters, this property can be omitted.

Example:

```
{
  "method": "people.get",
  "params": {
    "userId": "@me",
    "groupId": "@self"
  }
}
```

### data

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=data#data)▽

Property Value Type: object
Parent: -

Container for all the data from a response. This property itself has many reserved property names, which are described below. Services are free to add their own data to this object. A JSON response should contain either a `data` object or an `error` object, but not both. If both `data` and `error` are present, the `error` object takes precedence.

### error

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=error#error)▽

Property Value Type: object
Parent: -

Indicates that an error has occurred, with details about the error. The error format supports either one or more errors returned from the service. A JSON response should contain either a `data` object or an `error` object, but not both. If both `data` and `error` are present, the `error` object takes precedence.

Example:

```
{
  "apiVersion": "2.0",
  "error": {
    "code": 404,
    "message": "File Not Found",
    "errors": [{
      "domain": "Calendar",
      "reason": "ResourceNotFoundException",
      "message": "File Not Found
    }]
  }
}
```

## Reserved Property Names in the data object

The `data` property of the JSON object may contain the following properties.

### data.kind

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=data.kind#data.kind)▽

Property Value Type: string
Parent: `data`

The `kind` property serves as a guide to what type of information this particular object stores. It can be present at the `data` level, or at the `items` level, or in any object where its helpful to distinguish between various types of objects. If the `kind` object is present, it should be the first property in the object (See the "Property Ordering" section below for more details).

Example:

```
// "Kind" indicates an "album" in the Picasa API.
{"data": {"kind": "album"}}
```

### data.fields

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=data.fields#data.fields)▽

Property Value Type: string
Parent: `data`

Represents the fields present in the response when doing a partial GET, or the fields present in a request when doing a partial PATCH. This property should only exist during a partial GET/PATCH, and should not be empty.

Example:

```
{
  "data": {
    "kind": "user",
    "fields": "author,id",
    "id": "bart",
    "author": "Bart"
  }
}
```

### data.etag

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=data.etag#data.etag)▽

Property Value Type: string
Parent: `data`

Represents the etag for the response. Details about ETags in the GData APIs can be found here: https://code.google.com/apis/gdata/docs/2.0/reference.html#ResourceVersioning

Example:

```
{"data": {"etag": "W/"C0QBRXcycSp7ImA9WxRVFUk.""}}
```

### data.id

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=data.id#data.id)▽

Property Value Type: string
Parent: `data`

A globally unique string used to reference the object. The specific details of the `id` property are left up to the service.

Example:

```
{"data": {"id": "12345"}}
```

### data.lang

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=data.lang#data.lang)▽

Property Value Type: string (formatted as specified in BCP 47)
Parent: `data (or any child element)`

Indicates the language of the rest of the properties in this object. This property mimics HTML's `lang` property and XML's `xml:lang` properties. The value should be a language value as defined in [BCP 47](https://www.rfc-editor.org/rfc/bcp/bcp47.txt). If a single JSON object contains data in multiple languages, the service is responsible for developing and documenting an appropriate location for the `lang` property.

Example:

```
{"data": {
  "items": [
    { "lang": "en",
      "title": "Hello world!" },
    { "lang": "fr",
      "title": "Bonjour monde!" }
  ]}
}
```

### data.updated

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=data.updated#data.updated)▽

Property Value Type: string (formatted as specified in RFC 3339)
Parent: `data`

Indicates the last date/time ([RFC 3339](https://www.ietf.org/rfc/rfc3339.txt)) the item was updated, as defined by the service.

Example:

```
{"data": {"updated": "2007-11-06T16:34:41.000Z"}}
```

### data.deleted

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=data.deleted#data.deleted)▽

Property Value Type: boolean
Parent: `data (or any child element)`

A marker element, that, when present, indicates the containing entry is deleted. If deleted is present, its value must be `true`; a value of `false` can cause confusion and should be avoided.

Example:

```
{"data": {
  "items": [
    { "title": "A deleted entry",
      "deleted": true
    }
  ]}
}
```

### data.items

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=data.items#data.items)▽

Property Value Type: array
Parent: `data`

The property name `items` is reserved to represent an array of items (for example, photos in Picasa, videos in YouTube). This construct is intended to provide a standard location for collections related to the current result. For example, the JSON output could be plugged into a generic pagination system that knows to page on the `items` array. If `items` exists, it should be the last property in the `data` object (See the "Property Ordering" section below for more details).

Example:

```
{
  "data": {
    "items": [
      { /* Object #1 */ },
      { /* Object #2 */ },
      ...
    ]
  }
}
```

## Reserved Property Names for Paging

The following properties are located in the `data` object, and help page through a list of items. Some of the language and concepts are borrowed from the [OpenSearch specification](http://www.opensearch.org/Home).

The paging properties below allow for various styles of paging, including:

- Previous/Next paging - Allows user's to move forward and backward through a list, one page at a time. The `nextLink` and `previousLink` properties (described in the "Reserved Property Names for Links" section below) are used for this style of paging.
- Index-based paging - Allows user's to jump directly to a specific item position within a list of items. For example, to load 10 items starting at item 200, the developer may point the user to a url with the query string `?startIndex=200`.
- Page-based paging - Allows user's to jump directly to a specific page within the items. This is similar to index-based paging, but saves the developer the extra step of having to calculate the item index for a new page of items. For example, rather than jump to item number 200, the developer could jump to page 20. The urls during page-based paging could use the query string `?page=1` or `?page=20`. The `pageIndex` and `totalPages` properties are used for this style of paging.

An example of how to use these properties to implement paging can be found at the end of this guide.

### data.currentItemCount

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=data.currentItemCount#data.currentItemCount)▽

Property Value Type: integer
Parent: `data`

The number of items in this result set. Should be equivalent to items.length, and is provided as a convenience property. For example, suppose a developer requests a set of search items, and asks for 10 items per page. The total set of that search has 14 total items. The first page of items will have 10 items in it, so both `itemsPerPage` and `currentItemCount` will equal "10". The next page of items will have the remaining 4 items; `itemsPerPage` will still be "10", but `currentItemCount` will be "4".

Example:

```
{
  "data": {
    // "itemsPerPage" does not necessarily match "currentItemCount"
    "itemsPerPage": 10,
    "currentItemCount": 4
  }
}
```

### data.itemsPerPage

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=data.itemsPerPage#data.itemsPerPage)▽

Property Value Type: integer
Parent: `data`

The number of items in the result. This is not necessarily the size of the data.items array; if we are viewing the last page of items, the size of data.items may be less than `itemsPerPage`. However the size of data.items should not exceed `itemsPerPage`.

Example:

```
{
  "data": {
    "itemsPerPage": 10
  }
}
```

### data.startIndex

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=data.startIndex#data.startIndex)▽

Property Value Type: integer
Parent: `data`

The index of the first item in data.items. For consistency, `startIndex` should be 1-based. For example, the first item in the first set of items should have a `startIndex` of 1. If the user requests the next set of data, the `startIndex` may be 10.

Example:

```
{
  "data": {
    "startIndex": 1
  }
}
```

### data.totalItems

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=data.totalItems#data.totalItems)▽

Property Value Type: integer
Parent: `data`

The total number of items available in this set. For example, if a user has 100 blog posts, the response may only contain 10 items, but the `totalItems` would be 100.

Example:

```
{
  "data": {
    "totalItems": 100
  }
}
```

### data.pagingLinkTemplate

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=data.pagingLinkTemplate#data.pagingLinkTemplate)▽

Property Value Type: string
Parent: `data`

A URI template indicating how users can calculate subsequent paging links. The URI template also has some reserved variable names: `{index}` representing the item number to load, and `{pageIndex}`, representing the page number to load.

Example:

```
{
  "data": {
    "pagingLinkTemplate": "https://www.google.com/search/hl=en&q=chicago+style+pizza&start={index}&sa=N"
  }
}
```

### data.pageIndex

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=data.pageIndex#data.pageIndex)▽

Property Value Type: integer
Parent: `data`

The index of the current page of items. For consistency, `pageIndex` should be 1-based. For example, the first page of items has a `pageIndex` of 1. `pageIndex` can also be calculated from the item-based paging properties: `pageIndex = floor(startIndex / itemsPerPage) + 1`.

Example:

```
{
  "data": {
    "pageIndex": 1
  }
}
```

### data.totalPages

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=data.totalPages#data.totalPages)▽

Property Value Type: integer
Parent: `data`

The total number of pages in the result set. `totalPages` can also be calculated from the item-based paging properties above: `totalPages = ceiling(totalItems / itemsPerPage)`.

Example:

```
{
  "data": {
    "totalPages": 50
  }
}
```

## Reserved Property Names for Links

The following properties are located in the `data` object, and represent references to other resources. There are two forms of link properties: 1) objects, which can contain any sort of reference (such as a JSON-RPC object), and 2) URI strings, which represent URIs to resources (and will always be suffixed with "Link").

### data.self / data.selfLink

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=data.self_/_data.selfLink#data.self_/_data.selfLink)▽

Property Value Type: object / string
Parent: `data`

The self link can be used to retrieve the item's data. For example, in a list of a user's Picasa album, each album object in the `items` array could contain a `selfLink` that can be used to retrieve data related to that particular album.

Example:

```
{
  "data": {
    "self": { },
    "selfLink": "https://www.google.com/feeds/album/1234"
  }
}
```

### data.edit / data.editLink

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=data.edit_/_data.editLink#data.edit_/_data.editLink)▽

Property Value Type: object / string
Parent: `data`

The edit link indicates where a user can send update or delete requests. This is useful for REST-based APIs. This link need only be present if the user can update/delete this item.

Example:

```
{
  "data": {
    "edit": { },
    "editLink": "https://www.google.com/feeds/album/1234/edit"
  }
}
```

### data.next / data.nextLink

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=data.next_/_data.nextLink#data.next_/_data.nextLink)▽

Property Value Type: object / string
Parent: `data`

The next link indicates how more data can be retrieved. It points to the location to load the next set of data. It can be used in conjunction with the `itemsPerPage`, `startIndex` and `totalItems` properties in order to page through data.

Example:

```
{
  "data": {
    "next": { },
    "nextLink": "https://www.google.com/feeds/album/1234/next"
  }
}
```

### data.previous / data.previousLink

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=data.previous_/_data.previousLink#data.previous_/_data.previousLink)▽

Property Value Type: object / string
Parent: `data`

The previous link indicates how more data can be retrieved. It points to the location to load the previous set of data. It can be used in conjunction with the `itemsPerPage`, `startIndex` and `totalItems` properties in order to page through data.

Example:

```
{
  "data": {
    "previous": { },
    "previousLink": "https://www.google.com/feeds/album/1234/next"
  }
}
```

## Reserved Property Names in the error object

The `error` property of the JSON object may contain the following properties.

### error.code

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=error.code#error.code)▽

Property Value Type: integer
Parent: `error`

Represents the code for this error. This property value will usually represent the HTTP response code. If there are multiple errors, `code` will be the error code for the first error.

Example:

```
{
  "error":{
    "code": 404
  }
}
```

### error.message

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=error.message#error.message)▽

Property Value Type: string
Parent: `error`

A human readable message providing more details about the error. If there are multiple errors, `message` will be the message for the first error.

Example:

```
{
  "error":{
    "message": "File Not Found"
  }
}
```

### error.errors

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=error.errors#error.errors)▽

Property Value Type: array
Parent: `error`

Container for any additional information regarding the error. If the service returns multiple errors, each element in the `errors` array represents a different error.

Example:

```
{ "error": { "errors": [] } }
```

### error.errors[].domain

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=error.errors[].domain#error.errors[].domain)▽

Property Value Type: string
Parent: `error.errors`

Unique identifier for the service raising this error. This helps distinguish service-specific errors (i.e. error inserting an event in a calendar) from general protocol errors (i.e. file not found).

Example:

```
{
  "error":{
    "errors": [{"domain": "Calendar"}]
  }
}
```

### error.errors[].reason

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=error.errors[].reason#error.errors[].reason)▽

Property Value Type: string
Parent: `error.errors`

Unique identifier for this error. Different from the `error.code` property in that this is not an http response code.

Example:

```
{
  "error":{
    "errors": [{"reason": "ResourceNotFoundException"}]
  }
}
```

### error.errors[].message

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=error.errors[].message#error.errors[].message)▽

Property Value Type: string
Parent: `error.errors`

A human readable message providing more details about the error. If there is only one error, this field will match `error.message`.

Example:

```
{
  "error":{
    "code": 404
    "message": "File Not Found",
    "errors": [{"message": "File Not Found"}]
  }
}
```

### error.errors[].location

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=error.errors[].location#error.errors[].location)▽

Property Value Type: string
Parent: `error.errors`

The location of the error (the interpretation of its value depends on `locationType`).

Example:

```
{
  "error":{
    "errors": [{"location": ""}]
  }
}
```

### error.errors[].locationType

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=error.errors[].locationType#error.errors[].locationType)▽

Property Value Type: string
Parent: `error.errors`

Indicates how the `location` property should be interpreted.

Example:

```
{
  "error":{
    "errors": [{"locationType": ""}]
  }
}
```

### error.errors[].extendedHelp

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=error.errors[].extendedHelp#error.errors[].extendedHelp)▽

Property Value Type: string
Parent: `error.errors`

A URI for a help text that might shed some more light on the error.

Example:

```
{
  "error":{
    "errors": [{"extendedHelper": "http://url.to.more.details.example.com/"}]
  }
}
```

### error.errors[].sendReport

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=error.errors[].sendReport#error.errors[].sendReport)▽

Property Value Type: string
Parent: `error.errors`

A URI for a report form used by the service to collect data about the error condition. This URI should be preloaded with parameters describing the request.

Example:

```
{
  "error":{
    "errors": [{"sendReport": "https://report.example.com/"}]
  }
}
```

## Property Ordering

Properties can be in any order within the JSON object. However, in some cases the ordering of properties can help parsers quickly interpret data and lead to better performance. One example is a pull parser in a mobile environment, where performance and memory are critical, and unnecessary parsing should be avoided.

### Kind Property

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=Kind_Property#Kind_Property)▽

`kind` should be the first property

Suppose a parser is responsible for parsing a raw JSON stream into a specific object. The `kind` property guides the parser to instantiate the appropriate object. Therefore it should be the first property in the JSON object. This only applies when objects have a `kind` property (usually found in the `data` and `items` properties).

### Items Property

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=Items_Property#Items_Property)▽

`items` should be the last property in the `data` object

This allows all of the collection's properties to be read before reading each individual item. In cases where there are a lot of items, this avoids unnecessarily parsing those items when the developer only needs fields from the data.

### Property Ordering Example

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=Property_Ordering_Example#Property_Ordering_Example)▽

```
// The "kind" property distinguishes between an "album" and a "photo".
// "Kind" is always the first property in its parent object.
// The "items" property is the last property in the "data" object.
{
  "data": {
    "kind": "album",
    "title": "My Photo Album",
    "description": "An album in the user's account",
    "items": [
      {
        "kind": "photo",
        "title": "My First Photo"
      }
    ]
  }
}
```

## Examples

### YouTube JSON API

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=YouTube_JSON_API#YouTube_JSON_API)▽

Here's an example of the YouTube JSON API's response object. You can learn more about YouTube's JSON API here: https://code.google.com/apis/youtube/2.0/developers_guide_jsonc.html.

```
{
  "apiVersion": "2.0",
  "data": {
    "updated": "2010-02-04T19:29:54.001Z",
    "totalItems": 6741,
    "startIndex": 1,
    "itemsPerPage": 1,
    "items": [
      {
        "id": "BGODurRfVv4",
        "uploaded": "2009-11-17T20:10:06.000Z",
        "updated": "2010-02-04T06:25:57.000Z",
        "uploader": "docchat",
        "category": "Animals",
        "title": "From service dog to SURFice dog",
        "description": "Surf dog Ricochets inspirational video ...",
        "tags": [
          "Surf dog",
          "dog surfing",
          "dog",
          "golden retriever",
        ],
        "thumbnail": {
          "default": "https://i.ytimg.com/vi/BGODurRfVv4/default.jpg",
          "hqDefault": "https://i.ytimg.com/vi/BGODurRfVv4/hqdefault.jpg"
        },
        "player": {
          "default": "https://www.youtube.com/watch?v=BGODurRfVv4&feature=youtube_gdata",
          "mobile": "https://m.youtube.com/details?v=BGODurRfVv4"
        },
        "content": {
          "1": "rtsp://v5.cache6.c.youtube.com/CiILENy73wIaGQn-Vl-0uoNjBBMYDSANFEgGUgZ2aWRlb3MM/0/0/0/video.3gp",
          "5": "https://www.youtube.com/v/BGODurRfVv4?f=videos&app=youtube_gdata",
          "6": "rtsp://v7.cache7.c.youtube.com/CiILENy73wIaGQn-Vl-0uoNjBBMYESARFEgGUgZ2aWRlb3MM/0/0/0/video.3gp"
        },
        "duration": 315,
        "rating": 4.96,
        "ratingCount": 2043,
        "viewCount": 1781691,
        "favoriteCount": 3363,
        "commentCount": 1007,
        "commentsAllowed": true
      }
    ]
  }
}
```

### Paging Example

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=Paging_Example#Paging_Example)▽

This example demonstrates how the Google search items could be represented as a JSON object, with special attention to the paging variables.

This sample is for illustrative purposes only. The API below does not actually exist.

Here's a sample Google search results page:
![img](https://google.github.io/styleguide/jsoncstyleguide_example_01.png)
![img](https://google.github.io/styleguide/jsoncstyleguide_example_02.png)

Here's a sample JSON representation of this page:

```
{
  "apiVersion": "2.1",
  "id": "1",
  "data": {
    "query": "chicago style pizza",
    "time": "0.1",
    "currentItemCount": 10,
    "itemsPerPage": 10,
    "startIndex": 11,
    "totalItems": 2700000,
    "nextLink": "https://www.google.com/search?hl=en&q=chicago+style+pizza&start=20&sa=N"
    "previousLink": "https://www.google.com/search?hl=en&q=chicago+style+pizza&start=0&sa=N",
    "pagingLinkTemplate": "https://www.google.com/search/hl=en&q=chicago+style+pizza&start={index}&sa=N",
    "items": [
      {
        "title": "Pizz'a Chicago Home Page"
        // More fields for the search results
      }
      // More search results
    ]
  }
}
```

Here's how each of the colored boxes from the screenshot would be represented (the background colors correspond to the colors in the images above):

- Results 11 - 20 of about 2,700,000 = startIndex

- Results 11 - 20 of about 2,700,000 = startIndex + currentItemCount - 1

- Results 11 - 20 of about 2,700,000 = totalItems

- Search results = items (formatted appropriately)

- Previous/Next = previousLink / nextLink

- Numbered links in "Gooooooooooogle"

     

    = Derived from "pageLinkTemplate". The developer is responsible for calculating the values for {index} and substituting those values into the "pageLinkTemplate". The pageLinkTemplate's {index} variable is calculated as follows:

    - Index #1 = 0 * itemsPerPage = 0
    - Index #2 = 2 * itemsPerPage = 10
    - Index #3 = 3 * itemsPerPage = 20
    - Index #N = N * itemsPerPage

## Appendix

### Appendix A: Reserved JavaScript Words

[link](https://google.github.io/styleguide/jsoncstyleguide.xml?showone=Appendix_A:_Reserved_JavaScript_Words#Appendix_A:_Reserved_JavaScript_Words)▽

A list of reserved JavaScript words that should be avoided in property names.

The words below are reserved by the JavaScript language and cannot be referred to using dot notation. The list represents best knowledge of keywords at this time; the list may change or vary based on your specific execution environment.

From the [ECMAScript Language Specification 5th Edition](https://www.google.com/url?sa=D&q=http%3A%2F%2Fwww.ecma-international.org%2Fpublications%2Fstandards%2FEcma-262.htm)

```
abstract
boolean break byte
case catch char class const continue
debugger default delete do double
else enum export extends
false final finally float for function
goto
if implements import in instanceof int interface
let long
native new null
package private protected public
return
short static super switch synchronized
this throw throws transient true try typeof
var volatile void
while with
yield
```





# JSON风格指南

版本：0.9

英文版：https://google.github.io/styleguide/jsoncstyleguide.xml

翻译：Darcy Liu

译文状态：草稿

## 简介

该风格指南是对在Google创建JSON APIs而提供的指导性准则和建议。总体来讲，JSON APIs应遵循JSON.org上的规范。这份风格指南澄清和标准化了特定情况，从而使Google的JSON APIs有一种标准的外观和感觉。这些指南适用于基于RPC和基于REST风格的API的JSON请求和响应。

## 定义

为了更好地实现这份风格指南的目的，下面几项需要说明：

- 属性(property) - JSON对象内的键值对(name/value pair)
- 属性名(property name) - 属性的名称(或键)
- 属性值(property value) - 分配给属性的值

示例：

```
{
  // 一组键值对称作一个 "属性".
  "propertyName": "propertyValue"
}
```

Javascript的数字(*number*)包含所有的浮点数,这是一个宽泛的指定。在这份指南中，数字(*number*)指代Javascript中的数字(*number*)类型，而整型(*integer*)则指代整型。

## 一般准则

### 注释

**JSON对象中不包含注释。**

JSON对象中不应该包含注释。该指南中的某些示例含有注释。但这仅仅是为了说明示例。

```
{
  // 你可能在下面的示例中看到注释,
  // 但不要在你的JSON数据中加入注释.
  "propertyName": "propertyValue"
}
```

### 双引号

**使用双引号**

如果（某个）属性需要引号，则必须使用双引号。所有的属性名必须在双引号内。字符类型的属性值必须使用双引号。其它类型值（如布尔或数字）不应该使用双引号。

### 扁平化数据 VS 结构层次

**不能为了方便而将数据任意分组**

JSON中的数据元素应以*扁平化*方式呈现。不能为了方便而将数据任意分组。

在某些情况下，比如描述单一结构的一批属性，因为它被用来保持结构层次，因而是有意义的。但是遇到这些情况还是应当慎重考虑，记住只有语义上有意义的时候才使用它。例如，一个地址可以有表示两种方式，但结构化的方式对开发人员来讲可能更有意义：

扁平化地址:

```
{
  "company": "Google",
  "website": "http://www.google.com/",
  "addressLine1": "111 8th Ave",
  "addressLine2": "4th Floor",
  "state": "NY",
  "city": "New York",
  "zip": "10011"
}
```

结构化地址：

```
{
  "company": "Google",
  "website": "http://www.google.com/",
  "address": {
    "line1": "111 8th Ave",
    "line2": "4th Floor",
    "state": "NY",
    "city": "New York",
    "zip": "10011"
  }
}
```

## 属性名准则

### 属性名格式

**选择有意义的属性名**

属性名必须遵循以下准则:

- 属性名应该是具有定义语义的有意义的名称。
- 属性名必须是驼峰式的，ASCII码字符串。
- 首字符必须是字母，下划线(*_*)或美元符号(*$*)。
- 随后的其他字符可以是字母，数字，下划线(*_*)或美元符号(*$*)。
- 应该避免使用Javascript中的保留关键字(下文附有Javascript保留字清单)

这些准则反映JavaScript标识符命名的指导方针。使JavaScript的客户端可以使用点符号来访问属性。(例如, `result.thisIsAnInstanceVariable`).

下面是一个对象的一个属性的例子：

```
{
  "thisPropertyIsAnIdentifier": "identifier value"
}
```

### JSON Map中的键名

**在JSON Map中键名可以使用任意Unicode字符**

当JSON对象作为Map(映射)使用时，属性的名称命名规则并不适用。Map（也称作关联数组）是一个具有任意键/值对的数据类型，这些键/值对通过特定的键来访问相应的值。JSON对象和JSON Map在运行时看起来是一样的；这个特性与API设计相关。当JSON对象被当作map使用时，API文件应当做出说明。

Map的键名不一定要遵循属性名称的命名准则。键名可以包含任意的Unicode字符。客户端可使用maps熟悉的方括号来访问这些属性。（例如`result.thumbnails["72"]`）

```
{
  // "address" 属性是一个子对象
  // 包含地址的各部分.
  "address": {
    "addressLine1": "123 Anystreet",
    "city": "Anytown",
    "state": "XX",
    "zip": "00000"
  },
  // "address" 是一个映射
  // 含有响应规格所对应的URL，用来映射thumbnail url的像素规格
  "thumbnails": {
    "72": "http://url.to.72px.thumbnail",
    "144": "http://url.to.144px.thumbnail"
  }
}
```

### 保留的属性名称

**某些属性名称会被保留以便能在多个服务间相容使用**

保留属性名称的详细信息，连同完整的列表，可在本指南后面的内容中找到。服务应按照被定义的语义来使用属性名称。

### 单数属性名 VS 复数属性名

**数组类型应该是复数属性名。其它属性名都应该是单数。**

数组通常包含多个条目，复数属性名就反映了这点。在下面这个保留名称中可以看到例子。属性名*items*是复数因为它描述的是一组对象。大多数的其它字段是单数。

当然也有例外，尤其是涉及到数字的属性值的时候。例如，在保留属性名中，*totalItems* 比 *totalItem*更合理。然后，从技术上讲，这并不违反风格指南，因为 *totalItems* 可以被看作 *totalOfItems*, 其中 *total* 是单数（依照风格指南），*OfItems* 用来限定总数。字段名也可被改为 *itemCount*，这样看起来更象单数.

```
{
  // 单数
  "author": "lisa",
  // 一组同胞, 复数
  "siblings": [ "bart", "maggie"],
  // "totalItem" 看起来并不对
  "totalItems": 10,
  // 但 "itemCount" 要好些
  "itemCount": 10
}
```

### 命名冲突

**通过选择新的属性名或将API版本化来避免命名冲突**

新的属性可在将来被添加进保留列表中。JSON中不存在命名空间。如果存在命名冲突，可通过选择新的属性名或者版本化来解决这个问题。例如，假设我们由下面的JSON对象开始：

```
{
  "apiVersion": "1.0",
  "data": {
    "recipeName": "pizza",
    "ingredients": ["tomatoes", "cheese", "sausage"]
  }
}
```

如果我们希望将来把*ingredients*列为保留字，我们可以通过下面两件事情来达成:

1.选一个不同的名字

```
{
  "apiVersion": "1.0",
  "data": {
    "recipeName": "pizza",
    "ingredientsData": "Some new property",
    "ingredients": ["tomatoes", "cheese", "sausage"]
  }
}
```

2.在主版本上重新命名属性

```
{
  "apiVersion": "2.0",
  "data": {
    "recipeName": "pizza",
    "ingredients": "Some new property",
    "recipeIngredients": ["tomatos", "cheese", "sausage"]
  }
}
```

## 属性值准则

### 属性值格式

**属性值必须是Unicode 的 booleans（布尔）, 数字(numbers), 字符串(strings), 对象(objects), 数组(arrays), 或 null.**

JSON.org上的标准准确地说明了哪些类型的数据可以作为属性值。这包含Unicode的布尔(booleans), 数字(numbers), 字符串(strings), 对象(objects), 数组(arrays), 或 null。JavaScript表达式是不被接受的。APIs应该支持该准则，并为某个特定的属性选择最合适的数据类型（比如，用numbers代表numbers等）。

好的例子：

```
{
  "canPigsFly": null,     // null
  "areWeThereYet": false, // boolean
  "answerToLife": 42,     // number
  "name": "Bart",         // string
  "moreData": {},         // object
  "things": []            // array
}
```

不好的例子：

```
{
  "aVariableName": aVariableName,         // Bad - JavaScript 标识符
  "functionFoo": function() { return 1; } // Bad - JavaScript 函数
}
```

### 空或Null 属性值

**考虑移除空或null值**

如果一个属性是可选的或者包含空值或*null*值，考虑从JSON中去掉该属性，除非它的存在有很强的语义原因。

```
{
  "volume": 10,

  // 即使 "balance" 属性值是零, 它也应当被保留,
  // 因为 "0" 表示 "均衡" 
  // "-1" 表示左倾斜和"＋1" 表示右倾斜
  "balance": 0,

  // "currentlyPlaying" 是null的时候可被移除
  // "currentlyPlaying": null
}
```

### 枚举值

**枚举值应当以字符串的形式呈现**

随着APIs的发展，枚举值可能被添加，移除或者改变。将枚举值当作字符串可以使下游用户优雅地处理枚举值的变更。

Java代码：

```
public enum Color {
  WHITE,
  BLACK,
  RED,
  YELLOW,
  BLUE
}
```

JSON对象：

```
{
  "color": "WHITE"
}
```

## 属性值数据类型

上面提到，属性值必须是布尔(booleans), 数字(numbers), 字符串(strings), 对象(objects), 数组(arrays), 或 null. 然而在处理某些值时，定义一组标准的数据类型是非常有用的。这些数据类型必须始终是字符串，但是为了便于解析，它们也会以特定的方式被格式化。

### 日期属性值

**日期应该使用RFC3339建议的格式**

日期应该是RFC 3339所建议的字符串格式。

```
{
  "lastUpdate": "2007-11-06T16:34:41.000Z"
}
```

### 时间间隔属性值

**时间间隔应该使用ISO 8601建议的格式**

时间间隔应该是ISO 8601所建议的字符串格式。

```
{
  // 三年, 6个月, 4天, 12小时,
  // 三十分钟, 5秒
  "duration": "P3Y6M4DT12H30M5S"
}
```

### 纬度/经度属性值

**纬度/经度应该使用ISO 6709建议的格式**

纬度/经度应该是ISO 6709所建议的字符串格式。 而且, 它应该更偏好使用 e Â±DD.DDDDÂ±DDD.DDDD 角度格式.

```
{
  // 自由女神像的纬度/经度位置.
  "statueOfLiberty": "+40.6894-074.0447"
}
```

## JSON结构和保留属性名

为了使APIs保持一致的接口，JSON对象应当使用以下的结构。该结构适用于JSON的请求和响应。在这个结构中，某些属性名将被保留用作特殊用途。这些属性并不是必需的，也就是说，每个保留的属性可能出现零次或一次。但是如果服务需要这些属性，建议遵循该命名条约。下面是一份JSON结构语义表，以Orderly格式呈现(现在已经被纳入 JSONSchema)。你可以在该指南的最后找到关于JSON结构的例子。

```
object {
  string apiVersion?;
  string context?;
  string id?;
  string method?;
  object {
    string id?
  }* params?;
  object {
    string kind?;
    string fields?;
    string etag?;
    string id?;
    string lang?;
    string updated?; # date formatted RFC 3339
    boolean deleted?;
    integer currentItemCount?;
    integer itemsPerPage?;
    integer startIndex?;
    integer totalItems?;
    integer pageIndex?;
    integer totalPages?;
    string pageLinkTemplate /^https?:/ ?;
    object {}* next?;
    string nextLink?;
    object {}* previous?;
    string previousLink?;
    object {}* self?;
    string selfLink?;
    object {}* edit?;
    string editLink?;
    array [
      object {}*;
    ] items?;
  }* data?;
  object {
    integer code?;
    string message?;
    array [
      object {
        string domain?;
        string reason?;
        string message?;
        string location?;
        string locationType?;
        string extendedHelp?;
        string sendReport?;
      }*;
    ] errors?;
  }* error?;
}*;
```

JSON对象有一些顶级属性，然后是*data*对象或*error*对象，这两者不会同时出现。下面是这些属性的解释。

## 顶级保留属性名称

**顶级的JSON对象可能包含下面这些属性**

### apiVersion

```
属性值类型: 字符串(string)
父节点: -
```

呈现请求中服务API期望的版本，以及在响应中保存的服务API版本。应随时提供*apiVersion*。这与数据的版本无关。将数据版本化应该通过其他的机制来处理，如etag。

示例：

```
{ "apiVersion": "2.1" }
```

### context

```
属性值类型: 字符串(string)
父节点: -
```

客户端设置这个值，服务器通过数据作出回应。这在JSON-P和批处理中很有用，用户可以使用*context*将响应与请求关联起来。该属性是顶级属性，因为不管响应是成功还是有错误，*context*总应当被呈现出来。*context*不同于*id*在于*context*由用户提供而*id*由服务分配。

示例：

请求 #1:

```
http://www.google.com/myapi?context=bart
```

请求 #2:

```
http://www.google.com/myapi?context=lisa
```

响应 #1:

```
{
  "context": "bart",
  "data": {
    "items": []
  }
}
```

响应 #2:

```
{
  "context": "lisa",
  "data": {
    "items": []
  }
}
```

公共的JavaScript处理器通过编码同时处理以下两个响应：

```
function handleResponse(response) {
  if (response.result.context == "bart") {
    // 更新页面中的 "Bart" 部分。
  } else if (response.result.context == "lisa") {
    // 更新页面中的 "Lisa" 部分。
  }
}
```

### id

```
属性值类型: 字符串(string)
父节点: -
```

服务提供用于识别响应的标识(无论请求是成功还是有错误)。这对于将服务日志和单独收到的响应对应起来很有用。

示例：

```
{ "id": "1" }
```

### method

```
属性值类型: 字符串(string)
父节点: -
```

表示对数据即将执行，或已被执行的操作。在JSON请求的情况下，*method*属性可以用来指明对数据进行何种操作。在JSON响应的情况下，*method*属性表明对数据进行了何种操作。

一个JSON-RPC请求的例子，其中*method*属性表示要在*params*上执行的操作：

```
{
  "method": "people.get",
  "params": {
    "userId": "@me",
    "groupId": "@self"
  }
}
```

### params

```
属性值类型: 对象(object)
父节点: -
```

这个对象作为输入参数的映射发送给RPC请求。它可以和*method*属性一起用来执行RPC功能。若RPC方法不需要参数，则可以省略该属性。

示例：

```
{
  "method": "people.get",
  "params": {
    "userId": "@me",
    "groupId": "@self"
  }
}
```

### data

```
属性值类型: 对象(object)
父节点: -
```

包含响应的所有数据。该属性本身拥有许多保留属性名，下面会有相应的说明。服务可以自由地将自己的数据添加到这个对象。一个JSON响应要么应当包含一个*data*对象，要么应当包含*error*对象，但不能两者都包含。如果*data*和*error*同时出现，则*error*对象优先。

### error

```
属性值类型: 对象(object)
父节点: -
```

表明错误发生，提供错误的详细信息。错误的格式支持从服务返回一个或多个错误。一个JSON响应可以有一个*data*对象或者一个*error*对象，但不能两者都包含。如果*data*和*error*都出现，*error*对象优先。

示例：

```
{
  "apiVersion": "2.0",
  "error": {
    "code": 404,
    "message": "File Not Found",
    "errors": [{
      "domain": "Calendar",
      "reason": "ResourceNotFoundException",
      "message": "File Not Found
    }]
  }
}
```

## data对象的保留属性名

JSON对象的*data*属性可能包含以下属性。

### data.kind

```
属性值类型: 字符串(sting)
父节点: data
```

*kind*属性是对某个特定的对象存储何种类型的信息的指南。可以把它放在*data*层次，或*items*的层次，或其它任何有助于区分各类对象的对象中。如果*kind*对象被提供，它应该是对象的第一个属性（详见下面的_属性顺序_部分）。

示例：

```
// "Kind" indicates an "album" in the Picasa API.
{"data": {"kind": "album"}}
```

### data.fields

```
属性值类型: 字符串(string)
父节点: data
```

表示做了部分GET之后响应中出现的字段，或做了部分PATCH之后出现在请求中的字段。该属性仅在做了部分GET请求/批处理时存在，且不能为空。

示例：

```
{
  "data": {
    "kind": "user",
    "fields": "author,id",
    "id": "bart",
    "author": "Bart"
  }
}	
```

### data.etag

```
属性值类型: 字符串(string)
父节点: data
```

响应时提供etag。关于GData APIs中的ETags详情可以在这里找到：http://code.google.com/apis/gdata/docs/2.0/reference.html#ResourceVersioning

示例：

```
{"data": {"etag": "W/"C0QBRXcycSp7ImA9WxRVFUk.""}}
```

### data.id

```
属性值类型: 字符串(string)
父节点: data
```

一个全局唯一标识符用于引用该对象。*id*属性的具体细节都留给了服务。

示例：

```
{"data": {"id": "12345"}}
```

### data.lang

```
属性值类型: 字符串(string)(格式由BCP 47指定)
父节点: data (或任何子元素)
```

表示该对象内其他属性的语言。该属性模拟HTML的*lang*属性和XML的*xml:lang*属性。值应该是[BCP 47](http://www.rfc-editor.org/rfc/bcp/bcp47.txt)中定义的一种语言值。如果一个单一的JSON对象包含的数据有多种语言，服务负责制定和标明lang属性的适当位置。

示例：

```
{"data": {
  "items": [
    { "lang": "en",
      "title": "Hello world!" },
    { "lang": "fr",
      "title": "Bonjour monde!" }
  ]}
}
```

### data.updated

```
属性值类型: 字符串(string)(格式由RFC 3339指定)
父节点: data
```

指明条目更新的最后日期/时间([RFC 3339](http://www.ietf.org/rfc/rfc3339.txt))，由服务规定。

示例：

```
{"data": {"updated": "2007-11-06T16:34:41.000Z"}}
```

### data.deleted

```
属性值类型: 布尔(boolean)
父节点: data (或任何子元素)
```

一个标记元素，当出现时，表示包含的条目已被删除。如果提供了删除属性，它的值必须为*true*;为*false*会导致混乱，应该避免。

示例：

```
{"data": {
  "items": [
    { "title": "A deleted entry",
      "deleted": true
    }
  ]}
}
```

### data.items

```
属性值类型: 数组(array)
父节点: data
```

属性名*items*被保留用作表示一组条目(例如,Picasa中的图片，YouTube中的视频)。这种结构的目的是给与当前结果相关的集合提供一个标准位置。例如，知道页面上的*items*是数组，JSON输出便可能插入一个通用的分页系统。如果*items*存在，它应该是*data*对象的最后一个属性。（详见下面的_属性顺序_部分）。

示例：

```
{
  "data": {
    "items": [
      { /* Object #1 */ },
      { /* Object #2 */ },
      ...
    ]
  }
}
```

## 用于分页的保留属性名

下面的属性位于*data*对象中，用来给一列数据分页。一些语言和概念是从OpenSearch规范中借鉴过来的。

下面的分页数据允许各种风格的分页，包括：

- 上一页/下一页 - 允许用户在列表中前进和后退，一次一页。*nextLink* 和*previousLink*属性 (下面的"链接保留属性名"部分有描述) 用于这种风格的分页。
- 基于索引的分页 - 允许用户直接跳到条目列表的某个条目位置。例如，要从第200个条目开始载入10个新的条目，开发者可以给用户提供一个URL的查询字符串*?startIndex=200*。
- 基于页面的分页 - 允许用户直接跳到条目内的具体页。这跟基于索引的分页很类似,但节省了开发者额外的步骤，不需再为新一页的条目计算条目索引。例如，开发人员可以直接跳到第20页，而不是跳到第200条条目。基于页面分页的网址，可以使用查询字符串*?page=1*或*?page=20*。*pageIndex*和 *totalPages* 属性用作这种风格的分页.

在这份指南的最后可以找到如何使用这些属性来实现分页的例子。

### data.currentItemCount

```
属性值类型: 整数(integer)
父节点: data
```

结果集中的条目数目。应该与items.length相等，并作为一个便利属性提供。例如，假设开发者请求一组搜索条目，并且要求每页10条。查询集共有14条。第一个条目页将会有10个条目，因此*itemsPerPage*和*currentItemCount*都应该等于10。下一页的条目还剩下4条；*itemsPerPage*仍然是10,但是*currentItemCount*是4.

示例：

```
{
  "data": {
    // "itemsPerPage" 不需要与 "currentItemCount" 匹配
    "itemsPerPage": 10,
    "currentItemCount": 4
  }
}
```

### data.itemsPerPage

```
属性值类型: 整数(integer)
父节点: data
```

items结果的数目。未必是data.items数组的大小；如果我们查看的是最后一页，data.items的大小可能小于*itemsPerPage*。但是，data.items的大小不应超过*itemsPerPage*。

示例：

```
{
  "data": {
    "itemsPerPage": 10
  }
}
```

### data.startIndex

```
属性值类型: 整数(integer)
父节点: data
```

data.items中第一个条目的索引。为了一致，*startIndex*应从1开始。例如，第一组items中第一条的*startIndex*应该是1。如果用户请求下一组数据，*startIndex*可能是10。

示例：

```
{
  "data": {
    "startIndex": 1
  }
}
```

### data.totalItems

```
属性值类型: 整数(integer)
父节点: data
```

当前集合中可用的总条目数。例如，如果用户有100篇博客文章，响应可能只包含10篇，但是*totalItems*应该是100。

示例：

```
{
  "data": {
    "totalItems": 100
  }
}
```

### data.pagingLinkTemplate

```
属性值类型: 字符串(string)
父节点: data
```

URL模板指出用户可以如何计算随后的分页链接。URL模板中也包含一些保留变量名：表示要载入的条目的*{index}*，和要载入的页面的*{pageIndex}*。

示例：

```
{
  "data": {
    "pagingLinkTemplate": "http://www.google.com/search/hl=en&q=chicago+style+pizza&start={index}&sa=N"
  }
}
```

### data.pageIndex

```
属性值类型: 整数(integer)
父节点: data
```

条目的当前页索引。为了一致，*pageIndex*应从1开始。例如，第一页的*pageIndex*是1。*pageIndex*也可以通过基于条目的分页而计算出来*pageIndex = floor(startIndex / itemsPerPage) + 1*。

示例：

```
{
  "data": {
    "pageIndex": 1
  }
}
```

### data.totalPages

```
属性值类型: 整数(integer)
父节点: data
```

当前结果集中的总页数。*totalPages*也可以通过上面基于条目的分页属性计算出来: *totalPages = ceiling(totalItems / itemsPerPage).*。

示例：

```
{
  "data": {
    "totalPages": 50
  }
}
```

## 用于链接的保留属性名

下面的属性位于*data*对象中，用来表示对其他资源的引用。有两种形式的链接属性：1）对象，它可以包含任何种类的引用（比如JSON-RPC对象），2)URL字符串，表示资源的URIs(后缀总为'Link')。

### data.self / data.selfLink

```
属性值类型: 对象(object)/字符串(string)
父节点: data
```

自身链接可以用于取回条目数据。比如，在用户的Picasa相册中，条目中的每个相册对象都会包含一个*selfLink*用于检索这个相册的相关数据。

示例：

```
{
  "data": {
    "self": { },
    "selfLink": "http://www.google.com/feeds/album/1234"
  }
}
```

### data.edit / data.editLink

```
属性值类型: 对象(object)/字符串(string)
父节点: data
```

编辑链接表明用户可以发送更新或删除请求。这对于REST风格的APIs很有用。该链接仅在用户能够更新和删除该条目时提供。

示例：

```
{
  "data": {
    "edit": { },
    "editLink": "http://www.google.com/feeds/album/1234/edit"
  }
}
```

### data.next / data.nextLink

```
属性值类型: 对象(object)/字符串(string)
父节点: data
```

该下一页链接标明如何取得更多数据。它指明载入下一组数据的位置。它可以同*itemsPerPage*，*startIndex* 和 *totalItems* 属性一起使用用于分页数据。

示例：

```
{
  "data": {
    "next": { },
    "nextLink": "http://www.google.com/feeds/album/1234/next"
  }
}
```

### data.previous / data.previousLink

```
属性值类型: 对象(object)/字符串(string)
父节点: data
```

该上一页链接标明如何取得更多数据。它指明载入上一组数据的位置。它可以连同*itemsPerPage*，*startIndex* 和 *totalItems* 属性用于分页数据。

示例：

```
{
  "data": {
    "previous": { },
    "previousLink": "http://www.google.com/feeds/album/1234/next"
  }
}
```

\##错误对象中的保留属性名## JSON对象的*error*属性应包含以下属性。

### error.code

```
属性值类型: 整数(integer)
父节点: error
```

表示该错误的编号。这个属性通常表示HTTP响应码。如果存在多个错误，*code*应为第一个出错的错误码。

示例：

```
{
  "error":{
    "code": 404
  }
}
```

### error.message

```
属性值类型: 字符串(string)
父节点: error
```

一个人类可读的信息，提供有关错误的详细信息。如果存在多个错误，*message*应为第一个错误的错误信息。

示例：

```
{
  "error":{
    "message": "File Not Found"
  }
}	
```

### error.errors

```
属性值类型: 数组(array)
父节点: error
```

包含关于错误的附加信息。如果服务返回多个错误。*errors*数组中的每个元素表示一个不同的错误。

示例：

```
{ "error": { "errors": [] } }	
```

### error.errors[].domain

```
属性值类型: 字符串(string)
父节点: error.errors
```

服务抛出该错误的唯一识别符。它帮助区分服务的从普通协议错误(如,找不到文件)中区分出具体错误(例如，给日历插入事件的错误)。

示例：

```
{
  "error":{
    "errors": [{"domain": "Calendar"}]
  }
}
```

### error.errors[].reason

```
属性值类型: 字符串(string)
父节点: error.errors
```

该错误的唯一识别符。不同于*error.code*属性，它不是HTTP响应码。

示例：

```
{
  "error":{
    "errors": [{"reason": "ResourceNotFoundException"}]
  }
}
```

### error.errors[].message

```
属性值类型: 字符串(string)
父节点: error.errors
```

一个人类可读的信息，提供有关错误的更多细节。如果只有一个错误，该字段应该与*error.message*匹配。

示例：

```
{
  "error":{
    "code": 404
    "message": "File Not Found",
    "errors": [{"message": "File Not Found"}]
  }
}		
```

### error.errors[].location

```
属性值类型: 字符串(string)
父节点: error.errors
```

错误发生的位置（根据*locationType*字段解释该值）。

示例：

```
{
  "error":{
    "errors": [{"location": ""}]
  }
}
```

### error.errors[].locationType

```
属性值类型: 字符串(string)
父节点: error.errors
```

标明如何解释*location*属性。

示例：

```
{
  "error":{
    "errors": [{"locationType": ""}]
  }
}
```

### error.errors[].extendedHelp

```
属性值类型: 字符串(string)
父节点: error.errors
```

help text的URI，使错误更易于理解。

示例：（注：原示例这里有笔误，中文版这里做了校正）

```
{
  "error":{
    "errors": [{"extendedHelp": "http://url.to.more.details.example.com/"}]
  }
}
```

### error.errors[].sendReport

```
属性值类型: 字符串(string)
父节点: error.errors
```

report form的URI，服务用它来收集错误状态的数据。该URL会预先载入描述请求的参数

示例：

```
{
  "error":{
    "errors": [{"sendReport": "http://report.example.com/"}]
  }
}
```

## 属性顺序

在JSON对象中属性可有任意顺序。然而，在某些情况下，有序的属性可以帮助分析器快速解释数据，并带来更好的性能。在移动环境下的解析器就是个例子，在这种情况下，性能和内存是至关重要的，不必要的解析也应尽量避免。

### Kind属性

**Kind属性应为第一属性**

假设一个解析器负责将一个原始JSON流解析成一个特定的对象。*kind*属性会引导解析器将适合的对象实例化。因而它应该是JSON对象的第一个属性。这仅适用于对象有一个kind属性的情况(通常可以在*data*和*items*属性中找到)。

### Items属性

**_items_应该是_data_对象的最后一个属性**

这使得阅读每一个具体条目前前已读所有的集合属性。在有很多条目的情况下，这样就避免了开发人员只需要从数据的字段时不必要的解析这些条目。

这让阅读所有集合属性先于阅读单个条目。如遇多个条目的情况，当开发者仅需要数据中的字段时，这就可避免解析不必要的条目。

属性顺序示例：

```
// "kind" 属性区分 "album" 和 "photo".
// "Kind" 始终是它父对象的第一个属性.
// "items" 属性是 "data" 对象的最后一个属性.
{
  "data": {
    "kind": "album",
    "title": "My Photo Album",
    "description": "An album in the user's account",
    "items": [
      {
        "kind": "photo",
        "title": "My First Photo"
      }
    ]
  }
}
```

## 示例

### YouTube JSON API

这是YouTube JSON API响应对象的示例。你可以从中学到更多关于YouTube JSON API的内容：http://code.google.com/apis/youtube/2.0/developers_guide_jsonc.html

```
{
  "apiVersion": "2.0",
  "data": {
    "updated": "2010-02-04T19:29:54.001Z",
    "totalItems": 6741,
    "startIndex": 1,
    "itemsPerPage": 1,
    "items": [
      {
        "id": "BGODurRfVv4",
        "uploaded": "2009-11-17T20:10:06.000Z",
        "updated": "2010-02-04T06:25:57.000Z",
        "uploader": "docchat",
        "category": "Animals",
        "title": "From service dog to SURFice dog",
        "description": "Surf dog Ricochets inspirational video ...",
        "tags": [
          "Surf dog",
          "dog surfing",
          "dog",
          "golden retriever",
        ],
        "thumbnail": {
          "default": "http://i.ytimg.com/vi/BGODurRfVv4/default.jpg",
          "hqDefault": "http://i.ytimg.com/vi/BGODurRfVv4/hqdefault.jpg"
        },
        "player": {
          "default": "http://www.youtube.com/watch?v=BGODurRfVv4&feature=youtube_gdata",
          "mobile": "http://m.youtube.com/details?v=BGODurRfVv4"
        },
        "content": {
          "1": "rtsp://v5.cache6.c.youtube.com/CiILENy73wIaGQn-Vl-0uoNjBBMYDSANFEgGUgZ2aWRlb3MM/0/0/0/video.3gp",
          "5": "http://www.youtube.com/v/BGODurRfVv4?f=videos&app=youtube_gdata",
          "6": "rtsp://v7.cache7.c.youtube.com/CiILENy73wIaGQn-Vl-0uoNjBBMYESARFEgGUgZ2aWRlb3MM/0/0/0/video.3gp"
        },
        "duration": 315,
        "rating": 4.96,
        "ratingCount": 2043,
        "viewCount": 1781691,
        "favoriteCount": 3363,
        "commentCount": 1007,
        "commentsAllowed": true
      }
    ]
  }
}
```

### 分页示例

如何将Google搜索条目作为JSON对象展现出来，对分页变量也有特别关注。

这个示例仅用作说明。下面的API实际上并不存在。

这是Google搜索结果页面的示例： [![image](https://camo.githubusercontent.com/ab72a2779f414bbf3761f0e3e1e708a18ae22fd19c5ec7c3c81f8b7eb2c72f24/687474703a2f2f676f6f676c652d7374796c6567756964652e676f6f676c65636f64652e636f6d2f73766e2f7472756e6b2f6a736f6e637374796c6567756964655f6578616d706c655f30312e706e67)](https://camo.githubusercontent.com/ab72a2779f414bbf3761f0e3e1e708a18ae22fd19c5ec7c3c81f8b7eb2c72f24/687474703a2f2f676f6f676c652d7374796c6567756964652e676f6f676c65636f64652e636f6d2f73766e2f7472756e6b2f6a736f6e637374796c6567756964655f6578616d706c655f30312e706e67)

[![image](https://camo.githubusercontent.com/2c240bd507d07583b54a8188a69e68346ac999871e13eac4c734068036169ebf/687474703a2f2f676f6f676c652d7374796c6567756964652e676f6f676c65636f64652e636f6d2f73766e2f7472756e6b2f6a736f6e637374796c6567756964655f6578616d706c655f30322e706e67)](https://camo.githubusercontent.com/2c240bd507d07583b54a8188a69e68346ac999871e13eac4c734068036169ebf/687474703a2f2f676f6f676c652d7374796c6567756964652e676f6f676c65636f64652e636f6d2f73766e2f7472756e6b2f6a736f6e637374796c6567756964655f6578616d706c655f30322e706e67)

这是该页面JSON形式的呈现：

```
{
  "apiVersion": "2.1",
  "id": "1",
  "data": {
    "query": "chicago style pizza",
    "time": "0.1",
    "currentItemCount": 10,
    "itemsPerPage": 10,
    "startIndex": 11,
    "totalItems": 2700000,
    "nextLink": "http://www.google.com/search?hl=en&q=chicago+style+pizza&start=20&sa=N"
    "previousLink": "http://www.google.com/search?hl=en&q=chicago+style+pizza&start=0&sa=N",
    "pagingLinkTemplate": "http://www.google.com/search/hl=en&q=chicago+style+pizza&start={index}&sa=N",
    "items": [
      {
        "title": "Pizz'a Chicago Home Page"
        // More fields for the search results
      }
      // More search results
    ]
  }
}
```

这是如何展现屏幕截图中的色块的例子（背景颜色对应下图中的颜色）

- Results 11 - 20 of about 2,700,000 = startIndex

- Results 11 - 20 of about 2,700,000 = startIndex + currentItemCount - 1

- Results 11 - 20 of about 2,700,000 = totalItems

- Search results = items (formatted appropriately)

- Previous/Next = previousLink / nextLink

- Numbered links in "Gooooooooooogle"

     

    = Derived from "pageLinkTemplate". The developer is responsible for calculating the values for {index} and substituting those values into the "pageLinkTemplate". The pageLinkTemplate's {index} variable is calculated as follows:

    - Index #1 = 0 * itemsPerPage = 0
    - Index #2 = 2 * itemsPerPage = 10
    - Index #3 = 3 * itemsPerPage = 20
    - Index #N = N * itemsPerPage

## 附录

### 附录A:JavaScript中的保留字

**下列JavaScript保留字应该避免在属性名中使用**

下面的清单是JavaScript中的保留字，并不能通过点访问符访问。这份清单集合了当前最新的关键字，该清单可能会根据具体的执行环境而有所变更。

来自[ECMAScript 语言规范第五版](http://www.ecma-international.org/publications/standards/Ecma-262.htm)

```
abstract
boolean break byte
case catch char class const continue
debugger default delete do double
else enum export extends
false final finally float for function
goto
if implements import in instanceof int interface
let long
native new null
package private protected public
return
short static super switch synchronized
this throw throws transient true try typeof
var volatile void
while with
yield
```

除了特别[说明](http://code.google.com/policies.html)，该页面的内容均由[共同创作协议](http://creativecommons.org/licenses/by/3.0/)(CC BY 3.0)授权许可，示例代码均由[Apache 2.0](http://www.apache.org/licenses/LICENSE-2.0)许可证授权许可）

－EOF-