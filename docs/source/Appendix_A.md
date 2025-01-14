# A. Schema format

HED schema developers generally do initial development of the schema using `.mediawiki` format.
The tools to convert schema between `.mediawiki` and `.xml` format are located in the 
`hed.schema` module of the 
[hedtools](https://github.com/hed-standard/hed-python/tree/master/hedtools) 
project of the hed-python repository located at  
[https://github.com/hed-standard/hed-python](https://github.com/hed-standard/hed-python). 
All conversions are performed by converting the schema to a `HedSchema` object. 
Then modules `wiki2xml.py` and `xml2wiki.py` provide top-level functions to perform these
conversions. This appendix presents the rules for standard HED schema and library schema in `.mediawiki` 
and `.xml` formats.

## A.1. Mediawiki file format

The rules for creating a valid `.mediawiki` specification of a HED schema are given below. 
The format is line-oriented, meaning that all information about an individual entity should be on a 
single line. Empty lines and lines containing only blanks are ignored.

### A.1.1. Overall file layout

````{admonition} Overall layout of a HED MEDIAWIKI schema file.

```moin
header-line
prologue
             . . .
!# start schema
schema-specification
!# end schema
unit-class-specification
unit-modifier-specification
value-class-specification
schema-attribute-specification
property-specification
!# end hed
epilogue
```
````



### A.1.2. The *header-line*

The first line of the `.mediawiki` file should be a _header-line_ that starts with the 
keyword `HED` followed by a blank-separated list of name-value pairs.

````{eval-rst}
.. list-table:: Allowed HED schema header parameters
   :header-rows: 1
   :widths: 15 15 50

   * - Name
     - Level
     - Description
   * - library
     - optional
     - Name of library used in XML file names.  
     
       The value should only have alphabetic characters.
   * - version
     - required
     - A valid semantic version number of the schema.  
   * - xmlns
     - optional
     - xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance".
   * - xsi
     - optional
     - xsi:noNamespaceSchemaLocation points to XSD file.     
````

The `xsi` attribute is required if `xmlns:xsi` is given.
The `library` and `version` values are used to form the official xml file name and appear as attributes
in the `<HED>` root of the `.xml` file`.` The versions of the schema that use XSD validation to 
verify the format (versions 8.0.0 and above) have `xmlns:xsi` and `xsi:noNamespaceSchemaLocation` attributes.

````{admonition} **Example:** Version 8.0.0 of the HED MEDIAWIKI schema.

```moin
HED version="8.0.0"
```

````

The version line must be the first line of the `.mediawiki` file. The schema 
`.mediawiki` file is `HED-schema-8.0.0.mediawiki` found in 
[https://github.com/hed-standard/hed-specification/tree/master/hedwiki](https://github.com/hed-standard/hed-specification/tree/master/hedwiki).

````{admonition} **Example:** Version 8.0.0 of the standard HED XML schema.

```xml
<HED version="8.0.0">
```
````

The file name in `hedxml` in `hed-specification` is `HED8.0.0.xml`.

````{admonition} **Example:** Version 1.0.2 of HED test library in MEDIAWIKI format.

```moin
HED library="test" version="1.0.2"
```
````

The resulting XML root is:

````{admonition} **Example:** Version 1.0.2 of HED test library schema in XML format.
```xml
<HED library="test" version="1.0.2">
```
````

The file name in `hedxml` in the HED schema library `test` is `HED_test_1.0.2.xml`.

Unknown header-line attributes are translated as attributes of the `HED` root node of the 
`.xml` version, but a warning is used when the `.mediawiki` file is validated.

### A.1.3. Schema section

The beginning of the HED specification is marked by the *start-line*:

```moin
!# start schema
```

An arbitrary number of lines of informational text can be placed between the header-line 
and the start-line. Older versions of HED have a CHANGE_LOG as well as information about 
the syntax and rules. New versions of HED generate a separate change log file for released 
versions. 

The end of the main HED-specification is marked by the end-line:

```moin
!# end schema
```

The section separator lines (`!# start schema`, `!# end schema`, `!# end hed`) must only 
appear once in the file and must appear in that order within the file. A section separator 
is a line starting with `!#`.

The body of the HED specification consists of two types of lines: top-level node-specification
specifications and other node specifications. Each specification is a single line in the
`.mediawiki` file. Empty lines or lines containing only blanks are ignored. The basic format
for a node-specification is:

```moin
node-name  <nowiki>{attributes}[description]</nowiki>
```

Top-level node names are enclosed in triple single quotes (e.g., `'''Event'''`), while other
node names have at least one preceding asterisk (*) followed by a blank and then the name. 
The number of asterisks indicates the level of the node in the subtree. HED-3G node names 
can only contain alphanumeric characters, hyphens, and under-bars. They cannot contain blanks 
and must be unique. HED (2G) and earlier versions allow blanks.   Everything after the node
name must be contained within `<nowiki></nowiki>` tags. Placeholder nodes have an empty node 
name, but are followed by a `#` enclosed in  `<nowiki></nowiki>` tags.

````{admonition} **Example:** Different types of HED node specifications.

**Top-level:**

```moin
'''Property''' <nowiki>{extensionAllowed} [Subtree of properties.]</nowiki>
```

**Normal-level:**

```moin
***** Duration <nowiki>{requireChild} [Time extent of something.]</nowiki>
```

**Placeholder-level:**

```moin
****** <nowiki># {takesValue, unitClass=time,valueClass=numericClass}</nowiki>
```
````

The *Duration* tag of this example is at the fifth level below the root of its subtree. 
The tag: *Property/Data-property/Data-value/Spatiotemporal-value/Temporal-value/Duration* 
is the long form. The placeholder in the example is the node directly below *Duration* 
in the hierarchy.

### A.1.4. Other sections

After the line marking the end of the schema (`!# end schema`), the `.mediawiki` file contains 
the unit class specifications, unit modifier specifications, value class specification, 
the schema attribute specifications, and property specifications. All of these sections are
required starting with HED version 8.0.0 and must be given in this order.

Unit classes specify the kind of units are allowed to be used with a value that is provided
for a `#` value. The unit class specification section starts with `'''Unit classes'''` and 
lists the type of unit at the first level and the specific units at the second level.

````{admonition} **Example:** Part of the HED unit class specification for time.

```moin
'''Unit classes''' 
* time <nowiki>{defaultUnits=s}</nowiki> 
** second <nowiki>{SIUnit}</nowiki> 
** s <nowiki>{SIUnit, unitSymbol}</nowiki> 
```
````

The unit classes can be modified by SI (International System Units) sub-multiples 
and super-multiples. All unit modifiers are at level 1 of the `.mediawiki`
file. Unit modifiers have either the `SIUnitModifer` or the `SIUnitSymbolModifer`
to indicate whether they are regular modifiers or symbol modifiers.

````{admonition} **Example:** Part of the HED unit modifier specification.

```moin
'''Unit modifiers''' 
* deca <nowiki>{SIUnitModifier} [SI unit multiple for 10^1]</nowiki> 
* da <nowiki>{SIUnitSymbolModifier} [SI unit multiple for 10^1]</nowiki>
```
````

Units that have the `SIUnit` attribute can be modified by any unit modifier 
that has the `SIUnitModifier`. So for example, `second` and `decasecond` are
valid time units as are `seconds` and `decaseconds`. Similarly, units that
have the `SIUnit` and `unitSymbol` modifiers can be modified with unit modifiers
that have the `SIUnitSymbolModifier` attribute.

Value attributes give rules about what kind of value is allowed to be substituted
for `#` placeholder tags.

````{admonition} **Example:** Part of the HED value class specification.

```moin
'''Value classes'''
* posixPath <nowiki>{allowedCharacter=/,allowedCharacter=:}[Posix path specification.]</nowiki> 
```
````

The schema attributes specify other characteristics about how particular tags may be used in 
annotation. These attributes allow validators and other tools to process tag strings based
on the HED schema specification, thus avoiding hard-coding particular behavior.

````{admonition} **Example:** Part of the HED schema attribute specification.

```moin
'''Schema attributes'''
* allowedCharacter <nowiki>{valueClassProperty}[Attribute of value classes specifying a special character that is allowed in expressing the value of a placeholder.]</nowiki>
* defaultUnits <nowiki>{unitClassProperty}[Attribute of unit classes specifying the default units for a tag.]</nowiki> 
```
````
Notice that in the above example, the schema attributes, themselves have attributes referred to as
*HED schema properties*. These schema properties are listed in the `Properties` section of the
schema.

````{admonition} **Example:** Part of the HED schema property specification.

```moin
'''Properties''' 
* valueClassProperty <nowiki>[Attribute is meant to be applied to value classes.]</nowiki> 
```
````

## A.2. XML file format

The XML schema file format has a header, prologue, main schema, definitions, and epilogue
sections. The general layout is as follows:

````{admonition} XML layout of the HED schema.
```xml
<?xml version="1.0" ?>
<HED library="test" version="0.0.1" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="https://github.com/hed-standard/hed-specification/raw/master/hedxml/HED8.0.0-beta.3.xsd">
<prologue>unique optional text blob</prologue>
<schema>
         ...  schema specification  ... 
</schema>
<unitClassDefinitions>
   <unitClassDefinition> ... </unitClassDefinition>
                         ...
   <unitClassDefinition> ... </unitClassDefinition>
</unitClassDefinitions>
<unitModifierDefinitions>
   <unitModifierDefinition> ... </unitModifierDefinition>
                                ...
   <unitModifierDefinition> ... </unitModifierDefinition>
</unitModifierDefinitions>
    
<valueClassDefinitions>
    <valueClassDefinition> ... </valueClassDefinition>
                           ... 
    <valueClassDefinition> ... </valueClassDefinition>
</valueClassDefinitions>

<schemaAttributeDefinitions> 
   <schemaAttributeDefinition> ... </schemaAttributeDefinition>
                               ... 
   <schemaAttributeDefinition> ... </schemaAttributeDefinition>
</schemaAttributeDefinitions>

<propertyDefinitions>
    <propertyDefinition> ... </propertyDefinition>
                             ... 
    <propertyDefinition> ... </propertyDefinition>
</propertyDefinitions>

<epilogue>unique optional text blob</epilogue>
</HED>
```
````

The `<prologue>xxx</prologue>` and `<epilogue>xxx</epilogue>` elements are meant to be treated
as opaque as far as schema processing goes. In earlier versions of HED the prologue section
contained a Change Log for the schema as well as some basic documentation of syntax. 
The epilogue section contained additional metadata to be ignored during processing. 
The following subsections give a more detailed description of the format of these sections.

### A.2.1. The schema section

The schema section of the HED XML document consists of an arbitrary number of `<node></node>` 
elements enclosed in a single `<schema></schema>` element.

````{admonition} Top-level XML layout of the HED schema.
```xml
<schema>
    <node> ... </node>
           ...
    <node> ... </node>
</schema>
```
````

A `<node>` element contains a required `<name>` child element, an optional `<description>` 
child element, and an optional number of additional `<attribute>` child elements:

````{admonition} XML layout HED node element.
```xml
<node>
    <name>xxx</name>
    <description>yyy</description>
    <attribute> ... </attribute>
    <attribute> ... </attribute>
    <attribute> ... </attribute>   
    <node> ... <node>
</node>
```
````

The `<name>` element text must conform to the rules for naming HED schema nodes. 
It corresponds to the _node-name_ in the `mediawiki` specification and must not be empty. 
A `#` value is used to represent value place-holder elements.

The `<description>` element has the text contained in the square brackets `[]` in the 
`.mediawiki` node specification. If the `.mediawiki` specification is missing or has an 
empty `[]`, the `<description>` element is omitted.

The optional `<attribute>` elements are derived from the attribute list contained in curly 
braces `{}` of the `.mediawiki` specification. An `<attribute>` element has a single non-empty
`<name></name>` child element whose text value corresponds to the node-name of attribute in the
corresponding `.mediawiki` file. If the attribute does not have the `boolProperty`, 
then the `<attribute>` element should also have one or more child `<value></value>` elements
giving the value(s) of the attribute. **Example:** The `requireChild` attribute represents a boolean value. In the `.mediawiki` representation this attribute appears as `{requireChild}` if present and is omitted if absent.

````{admonition} The requireChild attribute represents a boolean value.

**Old xml if true:**     

```xml
<node requireChild="true"><name>xxx</name></node>
```

**New xml if true:**

```xml
<node>
    <name>xxx</name>
    <attribute>
       <name>requireChild</name>
    </attribute>
</node>
```
````

The `suggestedTag` attribute has a valid HED tag value. In the mediawiki representation 
this attribute is omitted if absent and appears when present as shown in this example.

````{admonition} The suggestedTag attribute has a valid HED tag value.

```moin
{suggestedTag=Sweet,suggestedTag=Gustatory-attribute/Salty}
```
````

The `suggestedTag` attribute is meant to be used by tagging tools to suggest additional tags
that a user might want to include. Notice that the `suggestedTag` values are  valid HED tags
in any form (short, long, or intermediate).

````{admonition} The suggestedTag old format.

**Old xml if present:**

```xml
<node suggestedTag="Sweet,Gustatory-attribute/Salty">
    <name>xxx</name>
</node>
```

**New xml if present:**

```xml
<node>
   <name>xxx</name>
   <attribute>
      <name>suggestedTag</name>
    	 <value>Sweet</value>
    	 <value>Gustatory-attribute/Salty</value>
   </attribute>
</node>
```
````

### A.2.2. Unit classes

The valid HED-3G unit classes are defined in the `<unitClassDefinitions>` section of the XML
schema file, and valid HED-3G unit modifiers are defined in the `<unitModifierDefinitions>` 
section. These sections follow a format similar to the `<node>` element in the `<schema>` 
section:

````{admonition} XML layout of the unit class definitions.
```xml
<unitClassDefinitions>
    <unitClassDefinition> ... </unitClassDefinition>
                          ... 
    <unitClassDefinition> ... </unitClassDefinition>
</unitClassDefinitions>
```
````

The `<unitClassDefinition>` elements have a required `<name>`, an optional `<description>` 
and an arbitrary number of additional `<attribute>` child elements. These `<attribute>` elements 
describe properties of the unit class rather than of individual unit types. In addition,
`<unitClassDefinition>` elements may have an arbitrary number of `<unit>` child elements.

````{admonition} XML layout of the unit class definitions.
```xml
<unitClassDefinition>
    <name>time</name>
    <description>Temporal values except date and time of day.</description>
    <attribute>
       <name>defaultUnits</name>
       <value>s</value>
    </attribute>
    <unit>
       <name>second</name>
       <description>SI unit second.</description>
       <attribute>
          <name>SIUnit</name>
       </attribute>
    </unit>
    <unit>
       <name>s</name>
       <description>SI unit second in abbreviated form.</description>
       <attribute>
          <name>SIUnit</name>
       </attribute>
       <attribute>
          <name>unitSymbol</name>
       </attribute>
    </unit>  
</unitClassDefinition>
```
````

### A.2.3. Value classes

Value classes are defined in the `<valueClassDefinitions>` section of the XML schema file. 
These sections follow a format similar to the `<node>` element in the `<schema>`:

````{admonition} XML layout of the unit class definitions.
```xml
<valueClassDefinitions>
    <valueClassDefinition> ... </valueClassDefinition>
                             ... 
    <valueClassDefinition> ... </valueClassDefinition>
</valueClassDefinitions>
```
````

### A.2.4. Schema attributes

The `<schemaAttributeDefinitions>` section specifies the allowed attributes of the other elements
including the `<node>`, `<unitClassDefinition>`, `<unitModifierDefinition>`, and
`<valueClassDefinition>` elements. The specifications of individual attributes are given in
`<schemaAttributeDefinition>` elements.

````{admonition} XML layout of the schema attribute definitions.
```xml
<schemaAttributeDefinitions>
    <schemaAttributeDefinition> ...</schemaAttributeDefinition>
                                   ... 
    <schemaAttributeDefinition> ... </schemaAttributeDefinition>
</schemaAttributeDefinitions>
```
````

The individual `<schemaAttributeDefinition>` elements have the following format:

````{admonition} XML layout of the schema attribute definitions.
```xml
<schemaAttributeDefinition>
    <name>allowedCharacter</name>
    <description>An attribute of value classes indicating a special character that is allowed in expressing the value of that placeholder.</description>
    <property>
        <name>valueClassProperty</name>
    </property>
</schemaAttributeDefinition>
```
````

## A.3. Schema sections

This section gives information about how the various auxiliary sections of the HED 
schema are used to specify the behavior of the schema elements.

### A.3.1. Schema properties

The `property` elements indicate where various schema attributes apply. 
Their meanings are hard-coded into the schema processors. The following is a list of schema
attribute properties. 

`````{list-table} Summary of unit classes and units in HED 8.0.0 (* indicates unit symbol).
:widths: 20 50
:header-rows: 1

* - Property
  - Description
* - boolProperty
  - Indicates schema attribute values are either true or false. 
* - unitClassProperty
  - Indicates schema attribute only applies to unit classes.
* - unitModifierProperty
  - Indicates schema attribute only applies to unit modifiers.
* - valueClassProperty
  - Indicates the schema attribute only applies to value classes.
* - textClass
  - Alphanumeric characters, blank, <br/>
    +, -, :, ;, ., /, (, ), ?, *, %, $, @, ^, _
``````


````{admonition} Notes on rules for allowed characters in the HED schema. 
:class: tip

1. Schema attributes with the `boolProperty`  have a `<name>` node but no 
`<value>` node in the XML.
Presence indicates true.
2. Schema attributes with the `boolProperty`  have both `<name>` and 
`<value>` nodes in the XML.

````


A given schema attribute can only apply to one type of element (`node`, `unitClassDefinition`, 
`unitModifierDefinition` or `unit`). Attributes that don’t have one of `unitClassProperty`,
`unitClassProperty` or `unitProperty` are assumed to apply to `node` elements.

### A.3.2. Schema attributes

As mentioned in the previous section schema attributes can only apply to one
type of element in the schema as indicated by their property values. 
Tools hardcode processing based on the schema attribute name. Only the schema 
attributes listed in the following table can be handled by current HED tools.


`````{list-table} Schema attributes (* indicates attribute has a value).
:widths: 20 15 45
:header-rows: 1

* - Attribute
  - Target
  - Description
* - allowedCharacter*
  - valueClass
  - Specifies a character used in values of this class.
* - defaultUnits*
  - unitClass
  - Specifies units to use if placeholder value has no units.   
* - extensionAllowed
  - node
  - A tag can have unlimited levels of child nodes added.
* - recommended
  - node
  - Event-level HED strings should include this tag.
* - relatedTag*
  - node
  - A HED tag closely related to this HED tag.
* - requireChild
  - node    
  - A child of this node must be included in the HED tag.
* - required
  - node      
  - Event-level HED string must include this tag.
* - SIUnit
  - unit   
  - This unit represents an SI unit and can be modified.
* - SIUnitModifier
  - unitModifier   
  - Modifier applies to base units.
* - SIUnitSymbolModifier
  - unitModifier    
  - Modifier applies to unit symbols.
* - suggestedTag*
  - node   
  - Tag could be included with this HED tag.
* - tagGroup
  - node   
  - Tag can only appear inside a tag group.
* - takesValue
  - node #   
  - Placeholder (#)should be replaced by a value.
* - topLevelTagGroup
  - node        
  - Tag (or its descendants) can be in a top-level tag group.
* - unique
  - node        
  - Tag or its descendants can only occur once in <br/>
    an event-level HED string.
* - unitClass*
  - node #        
  - Unit class this replacement value belongs to.
* - unitPrefix
  - unit        
  - Unit is a prefix (e.g., $ in the currency units).
* - unitSymbol
  - unit        
  - Tag is an abbreviation representing a unit.
* - valueClass*
  - node #        
  - Type of value this is.        
``````

Normally the allowed characters are listed individually as values of the `allowedCharacter`
attribute. However, the word `letters` designates upper and lower case alphabetic characters
are allowed. Further, the word `blank` indicates a space is an allowed character, and the 
word `digits` indicates the digits 0-9 may be used in the value.

If placeholder (`#`) has a `unitClass`, but the replacement value for the placeholder
does not have units, tools use the value of `defaultUnits` if the unit class has them.
For example, the `timeUnits` has the attribute `defaultUnits=s` in HED 8.0.0. The tag
`Duration/3` is assumed to be equivalent to `Duration/3 s` because `Duration` has
`defaultUnits` of `s`.

The `extensionAllowed` tag indicates that descendents of this node may be extended by
annotators. However, any tag that has a placeholder (`#`) child cannot be extended,
regardless of `extensionAllowed` since it single child is always interpreted as a
user-supplied value. 

Tags with the 'required' or 'unique' attributes cannot appear in definitions.

In addition to the attributes listed above, some schema attributes have been deprecated
and are no longer supported in HED, although they are still present in earlier versions of 
the schema. The following table lists these.

`````{list-table} Schema attributes (* indicates attribute has a value).
:widths: 20 15 45
:header-rows: 1

* - Schema attribute
  - Target
  - Description
* - default
  - node #
  - Indicates a default value used if no value is provided.
* - position
  - node    
  - Indicates where this tag should appear during display.
* - predicateType
  - node   
  - Indicates the relationship of the node to its parent. 
``````

The `default` attribute was not implemented in existing tools. 
The attribute is not used in HED-3G. Only the `defaultUnits` for the unit class 
will be implemented going forward. 

The `position` attribute was used to assist annotation tools, which sought to 
display required and recommend tags before others. The position attribute value
is an integer and the order can start at 0 or 1. Required or recommended 
tags without this attribute or with negative position will be shown after the 
others in canonical ordering. The tagging strategy of HED-3G using decomposition
and definitions. The `position` attribute is not used for HED-3G.

The `predicateType` attribute was introduced in HED-2G to facilitate mapping 
to OWL or RDF. It was needed because the HED-2G schema had a mixture of children
that were properties and subclasses. The possible values of `predicateType`
were `propertyOf`, `subclassOf`, or `passThrough` to indicate which role each
child node had with respect to its parent. The schema vocabulary redesign of 
HED-3G eliminated this issue. The attribute is ignored by tools.

### A.3.3. Value classes

HED has very strict rules about what characters are allowed in various elements of the HED
schema, HED tags and the substitutions made for `#` placeholders. These rules are encoded in 
the schema using value classes. When a node name or placeholder substitution is given a
particular value class, that name or substituted value can only contain the characters allowed
for that value class. 

The allowed characters for a value class are specified in the definition
of each value class. The HED validator and other HED tools may hardcode information about 
behavior of certain value classes (for example the `numericClass` value class). 
**HED does not allow commas or single quotes in any of its values.**

`````{list-table} Rules for value classes.
:widths: 20 50
:header-rows: 1

* - Value class
  - Allowed characters
* - dateTimeClass
  - digits, T, :, - 
* - nameClass
  - alphabetic characters, -
* - numericClass
  -  digits, ., -, +, E, e 
* - posixPath
  -  As yet unspecified
* - textClass
  - Alphanumeric characters, blank, <br/>
    +, -, :, ;, ., /, (, ), ?, *, %, $, @, ^, _
``````

````{admonition} Notes on rules for allowed characters in the HED schema. 
:class: tip

1. Commas are not allowed in any values.
2. Date-times should conform to ISO8601 date-time format YYYY-MM-DDThh:mm:ss.
3. Any variation on the full form of ISO8601 date-time is allowed.
4. The name class is for schema nodes and labels.
5. Values that have a value class of `numericClass` must be valid fixed point of floating point values.
6. Scientific notation is supported with the `numericClass`.
7. The text class is for descriptions, mainly for use with the *Description/* tag.
8. The posix path class is yet unspecified and currently allows any characters besides commas.

````

### A.3.4. HED unit classes

Unit classes allow annotators to express the units of values in a consistent way. 
The plurals of the various units are not explicitly listed, but are allowed as HED
tools uses standard pluralize functions to expand the list of allowed units. However,
Unit symbols represent abbreviated versions of units and cannot be pluralized. 

Nodes with the `SIUnit` modifier may be prefixed with multiple or sub-multiple modifiers.
If the SI unit does not also have the `unitSymbol` attribute, then multiples and sub-multiples
with the `SIUnitModifier` attribute are used for the expansion. On the other hand,
units with both `SIUnit` and `SIUnitModifier` attributes are expanded using
multiples and sub-multiples having the `SIUnitSymbolModifier` attribute.
  
Note that some units such as byte are designated as SI units, although they are not 
part of the standard.


`````{list-table} Unit classes and units in HED 8.0.0 (* indicates unit symbol).
:widths: 20 10 40
:header-rows: 1

* - Unit class
  - Default units
  - Units
* - accelerationUnits
  - m-per-s^2 
  - m-per-s^2*
* - angleUnits
  - rad 
  - radian, rad*, degree
* - areaUnits
  - m^2 
  - metre^2, m^2*
* - currencyUnits
  - $ 
  - dollar, $, point
* - frequencyUnits
  - Hz 
  - hertz, Hz*
* - intensityUnits
  - dB 
  - dB, candela, cd*
* - jerkUnits
  - m-per-s^3 
  - m-per-s^3*  
* - memorySizeUnits
  - B 
  - byte, B 
* - physicalLength
  - m 
  - metre, m*, inch, foot, mile   
* - speedUnits
  - m-per-s 
  - m-per-s*, mph, kph     
* - timeUnits
  - s 
  - second, s*, day, minute, hour
* - volumeUnits
  - m^3 
  - metre^3, m^3*  
* - weightUnits
  - g 
  - gram, g*, pound, lb   
``````

### A.3.5. HED unit modifiers

The unit modifiers are can be applied to SI base units to indicate multiples or
sub-multiples of the unit. Unit symbols are modified by unit symbol modifiers, whereas
non symbol SI units are modified by unit modifiers.

`````{list-table} Unit modifiers (* indicates unit symbol modifier).
:widths: 20 50
:header-rows: 1

* - Schema attribute
  - Description
* - deca, da*
  - SI unit multiple representing 10^1
* - hecto, h*
  - SI unit multiple representing 10^2
* - kilo, k*
  - SI unit multiple representing 10^3
* - mega, M*
  - SI unit multiple representing 10^6
* - giga, G*
  - SI unit multiple representing 10^9
* - tera, T*
  - SI unit multiple representing 10^12
* - peta, P*
  - SI unit multiple representing 10^15
* - exa, E*	
  - SI unit multiple representing 10^18
* - zetta, Z*
  - SI unit multiple representing 10^21
* - yotta, Y*
  - SI unit multiple representing 10^24
* - deci, d*
  - SI unit submultiple representing 10^−1
* - centi, c*
  - SI unit submultiple representing 10^−2
* - milli, m*
  - SI unit submultiple representing 10^−3
* - micro, u*
  - SI unit submultiple representing 10^−6
* - nano, n*
  - SI unit submultiple representing 10^−9
* - pico, p*
  - SI unit submultiple representing 10^−12
* - femto, f*
  - SI unit submultiple representing 10^−15
* - atto, a*
  - SI unit submultiple representing 10^−18
* - zepto, z*
  - SI unit submultiple representing 10^−21
* - yocto, y*
  - SI unit submultiple representing 10^−24
``````