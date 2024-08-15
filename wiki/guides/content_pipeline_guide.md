---
title: "Guide to Content Pipeline"
---

import Video from "../../components/Video";

Often games engage players with new content in a way that does not require new features to be created. Games usually do this by giving game designers an easy interface with which they create content (e.g. like google spreadsheets). The content is then exported and usually loaded at run-time. Because of the dynamic nature of all of this, bad data inputted into the content creation tool is often the cause of bugs. Sometimes one off validations are added to the content creation tool to mitigate this, but because this is also a time-consuming process and most teams don't add nearly enough validations.

The GPF has three significant advantages over the vast majority of content systems:

1. Instead of using one-off validations, our validation script knows how to validate every type of value that we support based on its type.

2. Instead of loading an external file at run-time, we generate compiled code so dependence on content is strongly typed.

3. Content is automatically placed in our `Data Store` making it accessible for our UI / `ViewBinding` system to leverage without.

<center>
<Video videoTitle="content_guide" youtubeID="ORHyJppXnCw" />
</center>

## Content Phases

Our content pipeline can be broken down into the following phases:

```mermaid
graph LR
    id1([1. Setup Spreadsheet]) --> id2([2. Add To Spreadsheet]) --> id3([3. Export/Import]) --> id4([4. Integrate Code])
```

### Setup Spreadsheet

Open the [Google Spreadsheet](https://docs.google.com/spreadsheets/d/1_WN5JmYfGNXZdaOOcFDlRSi4PHXiJ2UTQRsqslL3NXQ/edit?usp=sharing) for reference.

#### Notice:

- the sheets: `config`, `characters`, `strings`, `images`
- that the `id` column is always the first column.
- that the `config` sheet defines parameters for a single object (config) with values defined in the second column and types in the third column.
- that every other sheet defines parameters for multiple objects, with the type defined in the second row.
- that the `GPF` menu item has two scripts: `Export` and `Validate`.

#### dollar sign

By default, only the `Client` has access to content, but adding a dollar sign infront of the type makes that conent avaialble to server object code.

## strong vs week

The `A2` cell for each sheet can be defined as strong or weak depedning on how you want to reference that sheet's data in code.

`weak` creates a dictionary for that sheet, and an entry will be made for each item using the `id` as the key.  
e.g.

```csharp
SC.characters.ContainsKey(message.id);
```

`strong` will create a custom class for that sheet, with each id becoming a field which contains the item data.

```csharp
SC.configs.default_char;
```

These rules only apply to the Cleint Content (`CC`) and Server Content (`SC`) objects. Inside the datastore sheets are always represented as a list of strings (the ids).

#### config

If the `B2` cell of any sheet has type "config" then that sheet will only
define a single object, with values in the `B` column and types in the `C` column. Usually this will only be done for the "config" sheet, which is generally the first sheet. When a sheet is not a "config" sheet, then it will define an object per row, with fields defined in the first row of the sheet, types on the second, and values throughout the other rows.

#### C#

The following Basic C# datatypes are supported: string, int, float and bool.

#### ignore

Fields with type "ignore" will not be exported.

#### @

Sheets can be used as types when the sheet name is prepended with "@". To link to another sheet use the id of the other sheet as the value of the field that links to that sheet.

#### [,] and [;]

Every Type can be used as an array. You can use a comma or a semi-colon as the deliminator within the value cell.

#### Validation

As you setup your spreadsheet, setup validations by running GPF -> Validate. You will need to re-validate everytime you add a new column.

You can add notes to field names if you want to better explain them.

### Enter Data

Self explanatory.

### Export/Import

1. Export data using `GPF -> Export`. This will create a `content.txt` file. Add it to `Assets/Data`.

2. In Unity, If you don't see a `GPFContentLoader` in `Resources` then create one by right-clicking on `Resources` and going to `Create -> Game Plumbing Framework -> Content Loader`.

3. Select the `Content Loader` and set `Content` to the content file you just added.

4. Click `Generate Types`.

At this point, `GPF` will:

1. Create the `CC` (Client Config) class contiaining everything in the spreadsheet.
2. Create the `SC` (Server Config) class only contianing fields that begin with a dollar sign.
3. Add the content into the `DataStore`.

### Integrate Code

Content is automatically added to the `DataStore` under the "content" path.
Use `DataStore Explorer` to see how the `DataStore` path to your content Data.

Content is also accessable to ServerObjects using SC class, for example:

```csharp
public CharactersSC character = SC.configs.default_char;
```

Here we are setting character to the automatically generated `CharacterSC` specified by the `default_char` field in the `configs` sheet. Because the code is autogenerated it's best to simply explore the `SC` (which stands for "Serverobject Content") class using autocomplete to find the data you need.

You can do the same thing on the client using `CC`(which stands for "Client Content") instead of `SC`.
