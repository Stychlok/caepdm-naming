# Naming generator for openTD converter in CAE-PDM

## General Purpose
This document documents how the naming generator can be used to modify the default file names for export converters in CAE-PDM.

## Where is it used?
The naming generator is part of the CAE-PDM specific batch worker which is deployed at PAG. It is used if a configuration is provided.

## How it works?
A configuration needs to be provided. The configuration will be stored in the .config-file of batch worker (`converter-cli.exe.config`).
It contains a string with a mix of configurable place holders and fix text. A file extension can be configured per converter.

If no configuration is provided the openTD default file names will be used.

### Example Configuration

```
<converter>
    <naming-generator>
        <pattern>{Id}_{Project:TopMost}_{Project}_{Category:Porsche.CTs.Deviat:1}_PAG_{Category:Porsche.CTs.VehicleStructure:n}Info_{Timestamp}</pattern>
        <extension converterName="GFCP-Xml">.gfcp</extension>
        <extension converterName="GFCP-Excel">.xlsx</extension>
    </naming-generator>
</converter>
```

## Limitations

The maximum length of generated names is 256 chars including file extension. Generated names longer than that will be cut off after 256 chars minus length of file extension.
Therefore it's recommened to use the `{Id}` placeholder as early as possible to ensure unique names if needed.

## Placeholders

A defined set of placeholders can be used to define the pattern for naming generator.
A placeholder is defined by curly brakets and it's name e.g. `{Id}` or `{Timestamp}`.

Some placeholders can have parameters defined as described below. It's necessary to use the parameters in the correct order which is defined by it's index. Optional parameters cannot be omitted. So you can't use only the second parameter without the first one e.g. you can't set a format for `{Timestamp}` without defining the
Timestamp type parameter.

Invalid file name characters are replaced with `_`.

| Placeholder  | Description | Parameter (with Index) | Optional | Possible Values | Default value |
|-|-|-|-|-|-|
| Id           | Id of the data set. | - | - | - | - | - |
| Name           | Name of the data set. | - | - | - | - | - |
| Timestamp | A timestamp.<br>Invalid file name characters are replaced by `_` | Type (0) | yes | CreatedOn, ChangedOn, Current | CreatedOn
|  |  | Format (1) | yes | see below | d
| Project | Connected project | Level (0) | yes | Default, TopMost | Default |
| Category | Connected category(ies) | Category type (0) | no | Name of category type | - |
| | | Number (1) | yes | number > 0; all | 1

### Id

Id of the data set.

### Name

Name of the data set.

### Timestamp

A timestamp.

#### Parameter: Type

The type of timestamp

| Value | Description |
|-|-|
| CreatedOn | Creation timestamp |
| ChangedOn | Last changed timestamp |
| Current | Curent timestamp which is the timestamp the converter runs the conversion |

#### Parameter: Format

Format of the timestamp. See following links for possible values:
* https://learn.microsoft.com/de-de/dotnet/standard/base-types/standard-date-and-time-format-strings
* https://learn.microsoft.com/de-de/dotnet/standard/base-types/custom-date-and-time-format-strings

### Project

Project connected to the dataset.

#### Limitations

There is the technical possibility to connect more than one project to a data set. If more than one project is connected the first one will be picked. Which is - from technical perspective - a random one.

#### Parameter: Level

Define the level of project used.

| Value | Description |
|-|-|
| Default | Use connected project |
| TopMost | Go up in project hirarchy from the connection project to top-level parent and use this

### Category

Category or categories connected to the dataset. The display name of categories will used.

#### Parameter: Category type

Name of the category type to pick (hint: not the display name).

Mandatory.

#### Parameter: Number

As there can be connected a unlimmited number of categories of a category type to the data set you can specify how many categories you want to use.

| Value | Description |
|-|-|
| Number > 0 | Exactly how many categories should be picked. Default is 1. |
| all | all categories will be used

If there's more than one category used for the placeholder, the categories will be separated by `_`.

## Extensions

You can provide a file extension (including `.`) for each export converter. See configuration example above.
The extension length counts to the maximum length of the file name.

## Examples

### Example 1

Data set:
* Id: 123
* Name: My-DataSet 1
* CreatedOn: 2024-02-29T15:17:41.6870000Z
* Project:
    * Development<br>
      Which is a sub-project of Aurora
*  Categories
    * Category type: PDTec.PEP.Category.CustomType.VehicleStructureCategory
        * Series A
        * SeriesB

Pattern: `{Id}_{Name}_{Project:TopMost}_{Project}_{Category:PDTec.PEP.Category.CustomType.VehicleStructureCategory:all}_{Timestamp:CreatedOn:MM/dd/yy H:mm:ss}`

Result: `123_My-DataSet_1_Aurora_Development_Series_A_SeriesB_02_29_24_15_17_41`