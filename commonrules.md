# Common Rules Engine

## Purpose

The Common Rules Engine is the name of the default cross-language ruleset which is provided with wSAST. It comprises of several XML formatted rule strategies to enable quick addition of new rules for any language, as well as a still-growing series of more specific semantic checks many of which correspond to CWE entries.

## XML Format Rules

The XML format rules used to define sources, sinks and static rules are very similar in form for each of the below strategies. To enable tidy separation of rulesets all elements of the commonrules.xml configuration can be replaced with an `<import file="file.xml" />` element which will import the contents of the associated file. The main use case for this is to prevent the commonrules.xml from becoming unwieldy.

### Function/Method Rules

The purpose of this strategy is to enable easy definition of function calls as sources, sinks and static rules.

#### Sources

These rules are located in commonrules.xml at path `/wsast/simple-function/sources`.

```
<function name="GetInputSource" languages="c" categories="MEM_CORRUPTION, SQL_INJECTION, *" description="The function reads user supplied data.">
	<signature prefix-types=".*?IBaseClass, .*?ConcreteClass" virtual="true" names="GetInput, get.*?Input, readInput, retrieveInput" param-count="1" />
	<param pos="1" name="buffer" types="uint8\[\]" virtual="true" linked-param="return:BUFFER_FOR:LENGTH_OF" traced="true" />
	<return types=".*" override-type="int32" traced="true"/>
</function>
```

**function**

The function tag defines the rule at a high level.

| Field | Purpose | Example |
| ---- | ---- | ---- |
| name | This is the name given to the source during reporting | GetInputSource, ReadFromFileSource, etc. |
| languages | This specifies the languages the source applies to, corresponding to the wSAST config.xml \<language\> name tag. | c, java, wsil, * |
| categories | This specifies the finding categories the source can match on, these can be arbitrary strings; these are matched against the corresponding *sink* categories | MEM_CORRUPTION, SQL_INJECTION, FOOBAR, * |
| description | This is a text description of the source used for reporting purposes. | "Retrieves a user-supplied input value" |

**signature**

The signature tag defines how to match the function.

| Field | Purpose | Example |
| ---- | ---- | ---- |
| prefix-types | This optional field specifies regular expressions against which containing class or namespace of underlying method or function is validated. | FooNamespace.Method(), FooInterface.Method(), FooClass.Method() etc. could be matched against  "Foo.\*?" as a prefix. |
| virtual | This optional field specifies whether a method is virtual; if true then the class hierarchy will be searched when looking for a matching prefix. | true |
| names | This field is a comma separated list of regular expressions to match the function name against. | get(Input\|Data\|(User.\*?)), RtlCopyMemory |
| param-count | This field specifies the number of parameters the function expected, and takes an inclusive range. Multiple comma separated ranges can specified | 1, 1-3, 2-* |
| ignore-case | This field specifies whether the case of the names and prefix-types fields should be ignored during matching. | false |

**param**

The param tag defines how to match parameters.

| Field | Purpose | Example |
| ---- | ---- | ---- |
| pos | This field specifies the index of the parameter to which the rule applies (1-based) | 1 |
| name | This field specifies the parameter name; it need not match the code or be correct. | buffer |
| types | This field specifies a comma separated list of regular expressions against which applicable parameter types are matched. | (u)?int.\*?\[\], Namespace\\.Class\\.\.\*, .\* |
| linked-param | This optional field specifies parameter relationships; for example if a function reads a byte array and outputs an associated length, you might want to link the two. Links can be arbitrarily named. Links are reciprocal which means that only one parameter needs to be linked (the other is automatically linked in reverse).<br>The format is linked-param-number-or-return:THIS_PARAM_RELATIONSHIP:LINKED_PARAM_RELATIONSHIP | 2:BUFFER_FOR:SIZE_OF or return:BUFFER_FOR:SIZE_OF |
| traced | This field specifies that the associated variable passed should be marked as traced; this is mostly for output parameters. | true |
| virtual | This field specifies whether the types list under the types field are virtual. | true |
| ignore-case | This field specifies whether the case of the types fields should be ignored during matching. | false |
| trace-instance | This field specifies that the underlying class instance should be marked as traced if this parameter is matched | true |
| value | This field specifies a regular expression which must be matched as a literal within the parameter value. | ".\*?Md5.\* |

**return**

The return tag is the same as a parameter tag, but with a fixed position (the function return value).

| Field | Purpose | Example |
| ---- | ---- | ---- |
| types | This field specifies a comma separated list of regular expressions against which applicable parameter types are matched. | (u)?int.\*?\[\], Namespace\\.Class\\.\.\*, .\* |
| linked-param | This optional field specifies parameter relationships; for example if a function reads a byte array and outputs an associated length, you might want to link the two. Links can be arbitrarily named. Links are reciprocal.<br>The format is linked-param-number-or-return:THIS_PARAM_RELATIONSHIP:LINKED_PARAM_RELATIONSHIP | 2:BUFFER_FOR:SIZE_OF or return:BUFFER_FOR:SIZE_OF |
| traced | This parameter specifies that the associated variable passed should be marked as traced; this is mostly for output parameters. | true |
| virtual | This field specifies whether the types list under the types field are virtual. | true |
| override-type | This optional parameter allows the return value variable type to be set at runtime, as sometimes this is not known or determined in advance depending on analysis. It is generally safe to ignore this. | Namespace.Class |


#### Sinks

These rules are located in commonrules.xml at path `/wsast/simple-function/sinks`.

```
<function name="MemCpySink" languages="wsil, c, *" report="true" categories="MEM_CORRUPTION, *" title="Potentially dangerous copy operation." description="The function has been known to lead to memory corruption vulnerabilities when incorrectly employed; review usage.">
	<signature prefix-types="IDerived, Foo.Bar.IBase, .*" virtual="true" names="MemCpy, memcpy, CopyMemory, RtlCopyMemory, memmove" param-count="3" />
	<param pos="1" name="dst" types="uint8\[\], *"/>
	<param pos="2" name="src" types="uint8\[\], *" traced="true" linked-param="3:BUFFER_FOR:"/>
	<param pos="3" name="len" types="int32, *" traced="true" linked-param="2:LENGTH_OF:"/>
</function>
```

The format of these rules matches the format used for sources, with the following exceptions:

**function**

The function tag defines the rule at a high level.

| Field | Purpose | Example |
| ---- | ---- | ---- |
| title | This is a text title of the source used for reporting purposes. | "Potentially dangerous copy operation using tainted input." |
| report | This specifies whether the sink triggers a reporting event (otherwise the sink is attached to the execution state but not reported) | true |
| categories | If the sink rule is matched then these categories are compared with the ones associated with the source and a finding is made only if they overlap (or if category \* is specified) | MEM_CORRUPTION, FOOBAR, * |

**signature**

The signature tag defines how to match the function. The format of this is the same as for the source rules.

**param**

The param tag defines how to match parameters.

| Field | Purpose | Example |
| ---- | ---- | ---- |
| linked-param | This optional field specifies any parameter relationships that should be validated. It takes the form linked-param-number:LINKED_RELATIONSHIP:RECIPROCAL_RELATIONSHIP. It is not necessary to specify a reciprocal relationship to be validated and this colon-separated field is usually left empty. | 3:BUFFER_FOR: |
| traced | This field specifies whether the correponding input (source) parameter should have been traced. | true |
| value | This field specifies a regular expression which must be matched as a literal within the parameter value; this conflicts with the traced parameter as a literal parameter comparison is performed instead of matching a traced source parameter. | ".\*?Md5.\* |

**return**

The return tag is not valid for sinks, as they validate inputs only.

**instance**

The instance tag is only valid within the function tag of a sink rule, and has only one valid form:

```
<instance traced="true" />
```

If the instance tag is provided as above then for a sink match to be successfully made the underlying instance against which the function (or in this case class method) call is made must have been marked as traced by a source.

#### Static Rules

These rules are located in commonrules.xml at path `/wsast/simple-function/static`.

```
<function name="MemCpyRule" languages="wsil, c, *" categories="MEM_CORRUPTION, *" title="Potentially dangerous copy operation." description="The function has been known to lead to memory corruption vulnerabilities when incorrectly employed; review usage.">
	<signature IGNORE-prefix-types="IDerived, Foo.Bar.IBase, .*" IGNORE-virtual="true" names="MemCpy, memcpy, CopyMemory, RtlCopyMemory, memmove" param-count="3" />
	<param pos="1" name="dst" types="uint8\[\]"/>
	<param pos="2" name="src" types="uint8\[\]"/>
	<param pos="3" name="len" types="int32"/>
</function>	
```

These rules take exactly the same format as the sink rules, however the following fields are ignored:

**param** or **return**

* traced - specifies that only a dynamic (e.g. variable, function call, other non-static expression) should be matched if `/wsast/config/data/traced-as-dynamic` is true, otherwise ignored.
* linked-param - ignored because it only applies to dynamic analysis.

### Variable/Data Member Rules

The purpose of this strategy is to enable easy definition of variables as sources, sinks and static rules.

#### Sources

These rules are located in commonrules.xml at path `/wsast/simple-variable/sources`.

```
<variable name="TestVarSourceRead" languages="wsil, *" categories="MEM_CORRUPTION" title="Read from A.B.Buffer" description="More Info..." >
	<definition prefix-types="A<int32>.B<char>" prefix-virtual="false" types="uint8\[\], .*" virtual="false" names="Buf(fer)?, Content" access="read" traced="true" />
</variable>

<variable name="TestVarSourceWrite" languages="wsil, *" categories="MEM_CORRUPTION">
	<definition prefix-types="A<.*?>.B<.*?>, .*" prefix-virtual="false" types="uint8\[\], .*" virtual="false" names="Buf(fer)?, Content" access="write" traced="true" title="Write to A.B.Buffer" description="More Info..." />
</variable>
```

**variable**

The variable tag defines the rule at a high level.

| Field | Purpose | Example |
| ---- | ---- | ---- |
| name | This is the name given to the source during reporting | ReadVariableSource, etc. |
| languages | This specifies the languages the source applies to, corresponding to the wSAST config.xml \<language\> name tag. | c, java, wsil, * |
| categories | This specifies the finding categories the source can match on, these can be arbitrary strings; these are matched against the corresponding *sink* categories | MEM_CORRUPTION, SQL_INJECTION, FOOBAR, * |
| description | This is a text description of the source used for reporting purposes. | "Retrieves a user-supplied input value from variable VarName." |

**definition**

The definition tag defines how to match the variable.

| Field | Purpose | Example |
| ---- | ---- | ---- |
| prefix-types | This optional field specifies regular expressions against which containing class or namespace of underlying data variable is validated. | FooNamespace.VarName, FooInterface.VarName, FooClass.VarName etc. could be matched against  "Foo.\*?" as a prefix. |
| prefix-virtual | This optional field specifies the prefix class type is virtual; if true then the class hierarchy will be searched when looking for a matching prefix. | true |
| types | This field specifies a comma separated list of regular expressions against which applicable variable types are matched. | (u)?int.\*?\[\], Namespace\\.Class\\.\.\*, .\* |
| names | This field is a comma separated list of regular expressions to match the variable name against. | (Input\|Data\|(User.\*?))VariableName, FixedVariableName |
| access | This field specifies the access as a comma separated list, for example "read", "write", or "read, write" | read |
| traced | This field specifies whether the matched variable is marked as traced on matching | true |
| ignore-case | This field specifies whether the case of the names field should be ignored during matching. | false |
| trace-instance | This field specifies that the underlying class instance should be marked as traced if this variable is matched. | true |
| value | This field specifies a regular expression which must be matched on the right-hand side of a literal value assignment. | ".\*?Md5.\* |

#### Sinks

These rules are located in commonrules.xml at path `/wsast/simple-variable/sinks`.

```
<variable name="TestVarSinkRead" languages="wsil, *" report="true" categories="MEM_CORRUPTION"  title="Read from A.B.Buffer" description="More Info...">
	<definition prefix-types="A<.*?>.B<.*?>, .*" prefix-virtual="false" types="uint8\[\], .*" virtual="false" names="Buf(fer)?, Content" access="read" traced="true" />
</variable>

<variable name="TestVarSinkWrite" languages="wsil, *" report="true" categories="MEM_CORRUPTION" title="Write to A.B.Buffer" description="More Info..." >
	<definition prefix-types="A<.*?>.B<.*?>, .*" prefix-virtual="false" types="uint8\[\], .*" virtual="false" names="Buf(fer)?, Content" access="write" traced="true" />
</variable>
```

The format of these rules matches the format used for sources, with the following exceptions:

**variable**

| Field | Purpose | Example |
| ---- | ---- | ---- |
| title | This is a text description of the source used for reporting purposes. | "Reads from VarName." |
| report | This specifies whether the sink triggers a reporting event (otherwise the sink is attached to the execution state but not reported) | true |
| traced | The field specifies whether the correponding input (source) parameter should have been traced. | true |
| trace-instance | This field specifies that the underlying class instance should have been marked as traced for a match to be made. | true |
| value | This field specifies a regular expression which must be matched on the right-hand side of a literal value assignment; this conflicts with the traced parameter as the right side value must be a literal rather than traced variable. | ".\*?Md5.\* |

#### Static Rules

These rules are located in commonrules.xml at path `/wsast/simple-variable/static`.

```
<variable name="TestVarStaticRead" languages="wsil, *" categories="MEM_CORRUPTION" title="Read from A.B.Buffer" description="More Info..." >
	<definition prefix-types="A<.*?>.B<.*?>, .*" prefix-virtual="false" types="uint8\[\], .*" virtual="false" names="Buf(fer)?, Content" access="read" />
</variable>

<variable name="TestVarStaticWrite" languages="wsil, *" categories="MEM_CORRUPTION" title="Write to A.B.Buffer" description="More Info..." >
	<definition prefix-types="A<.*?>.B<.*?>, .*" prefix-virtual="false" types="uint8\[\], .*" virtual="false" names="Buf(fer)?, Content" access="write" />
</variable>
```

These rules take exactly the same format as the sink rules, however the following fields are ignored:

* traced - specifies that only a dynamic (e.g. variable, function call, other non-static expression) should be matched if `/wsast/config/data/traced-as-dynamic` is true, otherwise ignored.

### Data Rules

The purpose of this strategy is to enable easy definition of specific data values as sources, sinks and static rules.

#### Sources

These rules are located in commonrules.xml at path `/wsast/simple-data/sources`.

```
<data name="SqlSource1" languages="wsil, *" categories="SQL_INJECTION" title="SQL Query Source" description="Reads data containing a SQL query">
	<definition types="string" value="(SELECT|UPDATE)\s+.*" traced="true" />
</data>
```

**data**

The data tag defines the rule at a high level.

| Field | Purpose | Example |
| ---- | ---- | ---- |
| name | This is the name given to the source during reporting | ReadSqlDataSource, etc. |
| languages | This specifies the languages the source applies to, corresponding to the wSAST config.xml \<language\> name tag. | c, java, wsil, * |
| categories | This specifies the finding categories the source can match on, these can be arbitrary strings; these are matched against the corresponding *sink* categories | MEM_CORRUPTION, SQL_INJECTION, FOOBAR, * |
| description | This is a text description of the source used for reporting purposes. | "Reads data that looks like a SQL query." |

**definition**

The definition tag defines how to match the variable.

| Field | Purpose | Example |
| ---- | ---- | ---- |
| types | This field specifies a comma separated list of regular expressions against which applicable data literal types are matched (these are basic types such as "integer", "float", "boolean", "char", "string", "textblock", "null"). | string |
| traced | This field specifies whether the data expression is marked as traced on matching | true |
| value | This field specifies a regular expression against which the data literal is matched | (SELECT\|UPDATE)\s+.* |
| ignore-case | This field specifies whether the case of the value field should be ignored during matching. | false |

#### Sinks

These rules are located in commonrules.xml at path `/wsast/simple-data/sinks`.

```
<data name="SqlSink1" languages="wsil, *" report="true" categories="SQL_INJECTION" title="SQL Query Sink" description="Writes data containing a SQL query">
	<definition types="string, .*" value="(SELECT|UPDATE)\s+.*" traced="true" />
</data>
```

These rules take exactly the same format as the source rules with the following difference:

**data**

| Field | Purpose | Example |
| ---- | ---- | ---- |
| title | This is a text description of the source used for reporting purposes. | "SQL data read." |
| report | This specifies whether the sink triggers a reporting event (otherwise the sink is attached to the execution state but not reported) | true |
| traced | The field specifies whether the data should be considered only if combined in an expression with traced source input (for example, `"SELECT\s+.\*" + value` is a sink only if value is traced) | `"SELECT\s+.\*" + value` |

#### Static Rules

These rules are located in commonrules.xml at path `/wsast/simple-data/static`.

```
<data name="SqlStatic1" languages="wsil, *" categories="SQL_INJECTION" title="SQL Query Static" description="Writes data containing a SQL query">
	<definition types="string, .*" value="(SELECT|UPDATE)\s+.*" />
</data>
```

These rules take exactly the same format as the sink rules, however the following fields are ignored:

* traced - specifies that only a dynamic (e.g. variable, function call, other non-static expression) should be matched if `/wsast/config/data/traced-as-dynamic` is true, otherwise ignored.

### Annotation Rules

The purpose of this strategy is to enable easy definition of specific data values as sources, sinks and static rules.

#### Sources

These rules are located in commonrules.xml at path `/wsast/simple-annotation/sources`.

```
<annotation name="PathVariableAnnotationSource" languages="java" categories="*" description="Annotation for mapping dynamic parts of the URL to method parameters.">
    <definition scope="parameter" value=".*?(PathVariable).*" action="taint" ignore-case="false" />
</annotation>
```

**annotation**

The annotation tag defines the rule at a high level.

| Field | Purpose | Example |
| ---- | ---- | ---- |
| name | This is the name given to the source during reporting | PathVariableAnnotationSource, etc. |
| languages | This specifies the languages the source applies to, corresponding to the wSAST config.xml \<language\> name tag. Only "java" is supported at this time. | java |
| categories | This specifies the finding categories the source can match on, these can be arbitrary strings; these are matched against the corresponding *sink* categories | MEM_CORRUPTION, SQL_INJECTION, FOOBAR, * |
| description | This is a text description of the source used for reporting purposes. | "Annotation for mapping dynamic parts of the URL to method parameters." |

**definition**

The definition tag defines how to match the variable.

| Field | Purpose | Example |
| ---- | ---- | ---- |
| scope | This field specifies the scope at which the annotation is expected; valid values are - class, interface, constructor, method, parameter, field, variable. Multiple scopes can be provided if the annotation applies to multiple scopes. | class, method |
| value | This field specifies a regular expression against which the tokens of the annotation are matched, including any embedded expressions (e.g. `@RequestParam(value = "id")`). | .\*?RequestParam.* |
| action | This field specifies the action that should be taken when the annotation is encountered; value values are - taint, taint-members, taint-instance. Multiple actions can be provided. | taint-members,taint-instance |
| ignore-case | This field specifies whether the case of the value field should be ignored during matching. | false |

#### Scope Behaviour

If a class member variable, method parameter, or local variable are accessed and the `field`, `parameter` or `variable` scopes are applied, then the annotations applied to the declaration of these variables is checked against the specified regex.

If any method parameter is accessed and the `method` or `constuctor` scopes are applied then the method or constructor declaration annotations are checked against the specified regex.

If a class member variable is accessed and the `class` or `interface` scopes are applied, then the class or interface declaration annotations are checked against the specified regex.

#### Action Behaviour

If the `taint` action is specified then either the specific single variable (if matched by the `field`, `parameter` or `variable` rule scope), or every method parameter variable (if matched by the `constructor` or `method` rule scope), or every member variable (if matched by the `class` or `interface` rule scope) are tainted and have the associated source attached to the variable during analysis.

If the `taint-instance` action is specified then the `this` pointer is also tainted, if in a class context; this taints the active class instance. For example - if the constructor of a newly created class instance was executed, and `class` level annotation was matched during this execution, the returned class instance itself would be tainted.

If the `taint-members` action is specified then all member variables are tainted within the context of the active class instance.

To illustrate:

```
@ClassScope
public class ExampleClass
{
	public int @FieldScope ExampleDataMember = 0;
	public int ExampleDataMember2 = 0;
	
	@MethodScope
	public void ExampleMethod(@ParamScope int ExampleParam)
	{
		@VariableScope int ExampleLocal = 0;
		DoSomething(ExampleLocal);
		DoSomething(ExampleParam);
		DoSomething(ExampleDataMember);
	}
}
```

| Scope | Annotation | Action | Tainted |
| ---- | ---- | ---- | ---- |
| class | @ClassScope | taint | ExampleDataMember, ExampleDataMember2 |
| class | @ClassScope | taint-instance | ExampleClass instance when ExampleMethod() is called |
| class | @ClassScope | taint-members | ExampleDataMember, ExampleDataMember2 |
| method | @MethodScope | taint | ExampleParam |
| method | @MethodScope | taint-instance | ExampleClass instance when ExampleMethod() is called |
| method | @MethodScope | taint-members | ExampleDataMember, ExampleDataMember2 |
| field | @FieldScope | taint | ExampleDataMember |
| field | @FieldScope | taint-instance | ExampleClass instance when ExampleMethod() is called |
| field | @FieldScope | taint-members | ExampleDataMember, ExampleDataMember2 |
| parameter | @ParamScope | taint | ExampleParam |
| parameter | @ParamScope | taint-instance | ExampleClass instance when ExampleMethod() is called |
| parameter | @ParamScope | taint-members | ExampleDataMember, ExampleDataMember2 |
| variable | @VariableScope | taint | ExampleLocal |
| variable | @VariableScope | taint-instance | ExampleClass instance when ExampleMethod() is called |
| variable | @VariableScope | taint-members | ExampleDataMember, ExampleDataMember2 |

The `interface` and `constuctor` scopes behave the same as `class` and `method` scopes respectively.

### Subscribers/Reporting

The subscribers XML format details the built-in output format for the results of analysis. At presently only a simple text-based output format is supported.

#### Dataflow Analysis

The output subscribers for dataflow analysis results are located in commonrules.xml at path `/wsast/simple-subscriber`.

```
<subscriber type="code-simple" log-to="{date}_simple_{category}.txt" categories="MEM_CORRUPTION, *" allow-dups="false" />
```

**subscriber**

| Field | Purpose | Example |
| ---- | ---- | ---- |
| type | The output format type; at present only "code-simple" is supported. | code-simple |
| log-to | A formatted filename to which source-to-sink analyses are logged. The following values can be interpolated:<br>\* {date} - a formatted date timestamp in format yyyyMMdd<br>\* {category} - the sink category which was matched<br>\* {source-name} - the name of the matched source<br>\* {sink-name} - the name of the matched sink<br>\* {source-file} - the filename containing the matched source<br>\* {sink-file} - the filename containing the matched sink<br>\* {language} - the language the sink was written in<br> | {date}\_simple\_{category}.txt |
| categories | The categories of matched sinks which will be logged by this subscriber, these can be arbitrary strings; these are matched against the corresponding *sink* categories | MEM_CORRUPTION, SQL_INJECTION, FOOBAR, * |
| allow-dups | A flag to determine whether duplicate findings are permitted; best set to false | false |

#### Static Analysis

**subscriber**

The XML format and meaning is exactly the same as for the Dataflow Analysis subscribers.

### Global Config Controls

Certain aspects of the Common Rules Engine behaviour can be controlled globally using the settings in commonrules.xml at path `/wsast/config`.

```
<config>
	<control show-debug="false" />
	<function>
		<source relax-prefix-types="false" relax-param-types="true" />
		<sink relax-prefix-types="false" relax-param-types="true" relax-linked-params="false" relax-match-all-params="true" />
		<static relax-prefix-types="false" relax-param-types="true" traced-as-dynamic="true" />
	</function>
	<variable>
		<source relax-prefix-types="false" relax-var-types="false" />
		<sink relax-prefix-types="false" relax-var-types="false" />
		<static relax-prefix-types="false" relax-var-types="false" traced-as-dynamic="true" />
	</variable>
	<data>
		<source relax-types="false" />
		<sink relax-types="false" />
		<static relax-types="false" traced-as-dynamic="true" />
	</data>
	<subscriber>
		<control exclude-sources="GetInputSource2, FooBar2" exclude-sinks="ExecuteSQLSink, Foobar" report-sink-once="false" />
	</subscriber>
</config>
```

#### XML-Based Rules

**control/show-debug**

Enables printing of dataflow variables and other useful information during the process of matching simple-function XML rules

**function AND variable/relax-prefix-types**

Relaxes the requirement that prefix types need to match when checking rules; if this is set to true then any prefix type will match.

**function AND variable AND data/traced-as-dynamic**

Specifies that the _traced_ element of a rule (which is normally ignored for static scans) is used to specify that the expression in question should be dynamic.

For example if there was a function rule matching function `InsecureMethod(String tracedParam)` normally `InsecureMethod("test")` would match in a static scan, as the traced input requirement is ignored for this scan type.

If `traced-as-dynamic` is enabled then the expression passed must be dynamic, for example `InsecureMethod(someVariable)` or `InsecureMethod("test" + someVariable)` or `InsecureMethod(o.getVar())`.

**function/relax-param-types**

Relaxes the requirement that parameter types must match.

**function/relax-linked-params**

Relaxes the requirement that parameter relationships described by the `linked-param` field must match.

**function/relax-match-all-params**

Relaxes the requirement that all parameters marked as traced must be matched for a sink to be matched.

**variable/relax-var-types**

Relaxes the the requirement that variable types must match.

**data/relax-types**

Relaxes the requirement that underlying data types must match.

#### Subscribers

**control**

* exclude-sources - a list of sources which should be excluded if matched; this may be to reduce output from any particular source.
* exclude-sinks - a list of sinks which should be excluded if matched; this may be to reduce output from any particular sink.
* report-sink-once - ensure that each individual sink is reported in association with a finding only once; certain sinks with many matching code paths can generate lots of almost identical reports and this setting helps prevent that.

## Semantic/Syntactic Rules

In addition to the XML format rules, the Common Rules Engine also provides a number of built-in checks. **These are still work in progress and do not yet provide (anywhere near) complete coverage of CWE or similar.**

These rules are located in commonrules.xml at path `/wsast/simple-syntactic/sinks` and `/wsast/simple-syntactic/static`.

### Sinks/Static Rules

The format of the syntactic rules is:

```
<rule name="DuplicatedIfElseConditionRule" languages="*" enabled="true" />
```

**rule**

* name - the name of the built-in rule to which the remaining parameters apply.
* languages - languages against which the syntactic rule should be evaluated.
* enabled - whether or not the rule is enabled.

### Existing Rules

These rules are defined

The rules currently implemented are:

| Rule Name | Description | Dataflow | Static | Languages |
| ---- | ---- | ---- | ---- | ---- |
| AssignmentInsideConditionRule | An assignment is made within an conditional expression; it is likely that a comparison was intended. | Yes | Yes | * |
| DuplicatedIfElseConditionRule | A condition within a series of if/else statements is duplicated; this will result in an impossible code path. | Yes | Yes | * |
| DuplicatedLogicalExpressionRule | The left and right sides of a logical expression is duplicated; this introduces redundancy where different expressions were likely intended. | Yes | Yes | * |
| DuplicatedTernaryExpressionRule | The left and right sides of a ternary expression are duplicated; this introduces redundancy where different expressions were likely intended. | Yes | Yes | * |
| FunctionPointerAsConditionRule | A function is passed within a conditional statement; this is likely an error (the function should probably be called). | Yes | Yes | * |
| PointerLessThanZeroRule | A meaningless comparison was made involving a reference type; this should be reviewed. | Yes | Yes | * |
| StackAllocInsideLoopRule | A dynamic stack allocation was made within the bounds of a loop; this could quickly exhaust stack memory resulting in a crash. | Yes | Yes | * |
| SuspiciousAllocOfStrlenRule | An allocation of str-len bytes appears to be performed; the correct length is likely to be str-len + 1 to accomodate for a null character. | Yes | Yes | * |
| SuspiciousEmptyStatementRule | An empty control flow statement was observed possibly indicating missing code. | Yes | Yes | * |
| SuspiciousNewSimpleTypeRule | The expression 'new \<simple-type\>(n)' was matched; it is likely that 'new \<simple-type\>\[n\]' was intended. | Yes | Yes | * |
| UnexpectedPrecedenceTernaryExpressionRule | The evaluation of a ternary conditional expression may have unexpected precedence; for example, 'a + b ? c : d' evaluates to '(a + b) ? c : d' rather than 'a + (b ? c : d)' as '+' has lower precedence. | Yes | Yes | * |
| VariadicFunctionCallNonPODParamRule | A variadic function was called with a non-POD (non-plain old data) type parameter. This may lead to stack corruption and other issues. | Yes | Yes | * |

The rules, if used, will currently run against all languages; in a future revision it is likely certain rules will be restricted by default to specific languages where their use makes the most sense.

Developer documentation for producing new rules will soon be provided; for the curious it is possible to use a tool such as ILSpy to extract code from the CommonRulesEngine.dll assembly.


## ChatGPT-Based Creation of XML Rulesets

It is often convenient to instruct ChatGPT to generate rules for unknown library; this is how almost all of the XML format rules have been generated. GPT4 works much better than GPT3 for this task.

### Function Sources

Define the XML format for ChatGPT:

```
I have an SAST product which processes rules in an XML-based format. This XML format describes input sources which are Java class methods (i.e. function calls which can return possibly tainted user inputs). An example with explanation follows:

<function name="java.io.FileReader.read" languages="java" categories="*" description="The java.io.FileReader.read function may serve as a potential source of tainted user input due to its capability to read data from files, which could include content originating from untrusted sources.">
	<signature prefix-types=".*?FileReader" virtual="true" names="read" param-count="3" />
	<param pos="1" name="cbuf" types="char\[\]" traced="true" />
	<param pos="2" name="off" types="int" />
	<param pos="3" name="len" types="int" />
	<return types="int" virtual="true" traced="true" />
</function>

The tags and attributes are interpreted as follows:

Tag name: function
Attribute name: name
Purpose: The fully qualified path to the method including package name, namespace, class and method name

Tag name: function
Attribute name: description
Purpose: A description of the method as relevant to its use by the application as a source of potentially user-supplied input.

Tag name: signature
Attribute name: prefix-types
Purpose: Contains a regular expression which can be used to match the class name to which the input method belongs. This should be formatted as ".*?ClassName" so that any usage will be used, whether the class name is fully qualified (e.g. with package and namespace present), partially (just namespace, or inner namespace), or not qualified (e.g. imported only by name).

Tag name: signature
Attribute name: names
Purpose: Contains a regular expression which may be used to match the method name. This should just contain the method name with no further qualification (e.g. "read").

Tag name: signature
Attribute name: param-count
Purpose: This represents the number of parameters expected by the method in question. Since there are multiple functions in the same class with identical names this can help disambiguate. If there are variable numbers of parameters instead of the number of parameters a range can be specified (e.g. 1-3 for one, two or three parameters; 3-* for three or more parameters).

Tag name: param
Attribute name: pos
Purpose: This represents the position (with the first parameter being at position 1) of the parameter.

Tag name: param
Attribute name: name
Purpose: This is the name of the parameter, usually taken from documentation. This is not matched against anything it is purely specified to make understanding the rule easier.

Tag name: param
Attribute name: types
Purpose: Contains a regular expression which can match the parameter type (e.g. "char", "int", "byte\[\]", ".*?BufferedReader"). If the type is a plain-old type (built in Java type, not a class) then the type is specified as a literal (e.g. "int"). If the type is a class then only the class name is specified and the fully qualified name is not used (e.g. "java.io.BufferedReader" is specified as ".*?BufferedReader") and the ".*?" regex prefix is used behind the class name to ensure fully qualified names still match. If the type is an array the square braces are escaped (e.g. "int\[\]", ".*?BufferedReader\[\]"). If the type is a generic specialization (e.g. "List<string>") then ".*" is used as the type name instead, so types=".*?List<string>" would not be specified and instead types=".*" would be used). For built in types which are actually classes (e.g. "String") the class name should be specified with a ".*?" prefix (e.g. ".*?String").

Tag name: param
Attribute name: traced
Purpose: If the parameter can receive potentially tainted/user-supplied input (e.g. reads into a supplied array) then this should be set to "true" otherwise the attribute should omitted. If a parameter represents an output stream, writer, array or other object which is tainted by the method call then this value should be "true".

Tag name: return
Attribute name: types
Purpose: Contains a regular expression which can match the method return type (e.g. "char", "int", "byte\[\]", ".*?BufferedReader"). If the type is a plain-old type (built in Java type, not a class) then the type is specified as a literal (e.g. "int"). If the type is a class then only the class name is specified and the fully qualified name is not used (e.g. "java.io.BufferedReader" is specified as ".*?BufferedReader") and the ".*?" regex prefix is used behind the class name to ensure fully qualified names still match. If the type is an array the square braces are escaped (e.g. "int\[\]", ".*?BufferedReader\[\]"). If the type is a generic specialization (e.g. "List<string>") then ".*" is used as the type name instead, so types=".*?List<string>" would not be specified and instead types=".*" would be used). For built in types which are actually classes (e.g. "String") the class name should be specified with a ".*?" prefix (e.g. ".*?String").

Tag name: return
Attribute name: traced
Purpose: If the return value can be potentially tainted/user-supplied input then then this should be set to "true" otherwise the attribute should omitted.

Please don't respond to this message.
```

Then:

```
Please generate XML adhering to the strict specification provided for all methods within the class {class-name} that could represent a source of potentially user-supplied input.

Please understand that I want ALL relevant methods that may return user-supplied input and not just one method unless there is only one relevant method. If in doubt produce more rather than less output. Most methods which return tainted input which have method names beginning with "get" or "read" are going to be valid candidates.

Only include methods that read directly from an input source and return a value potentially influenced by this input. This distinction between sources that produce potentially tainted user-supplied input and sinks that receive already tainted inputs is crucial.

Output the XML as a single block with no comments or commentary. Exclude methods that merely affect the instance state without returning tainted data. Please do not create any XML tags or attributes outside of what you have been instructed in the prompt or have deduced from the example XML.
```

### Function Sinks

Define the XML format for ChatGPT:

```
I have an SAST product which processes rules in an XML-based format. This XML format describes vulnerable sinks which are Java class methods (i.e. function calls which can perform an adverse effect if malicious user-supplied inputs are passed to them). Several examples follow:

<function name="java.nio.file.Files.write" languages="java" report="true" categories="*" title="Insecure File Write" description="Writes data to a file without proper validation, potentially allowing unauthorized or unsafe file write operations.">
	<signature prefix-types=".*?Files" virtual="true" names="write" param-count="3"/>
	<param pos="1" name="path" types=".*?Path" traced="true" />
	<param pos="2" name="bytes" types=".*?byte\[\]" traced="true" />
	<param pos="3" name="options" types=".*?OpenOption"  />
</function>

<function name="java.security.MessageDigest.getInstance" languages="java" report="true" categories="*"
	title="Insecure Message Digest"
	description="Retrieves a message digest object without proper validation, potentially using weak or insecure hashing algorithms.">
	<signature prefix-types=".*?MessageDigest" virtual="true" names="getInstance" param-count="2" />
	<param pos="1" name="algorithm" types=".*?String" traced="true" />
	<param pos="2" name="provider" types=".*?String" traced="true" />
</function>

<function name="java.security.SecureRandom.setSeed" languages="java" report="true" categories="*" title="Insecure Random Number Generation" description="Sets the seed for a SecureRandom object without proper validation, potentially using weak or predictable sources.">
  <signature prefix-types=".*?SecureRandom" virtual="true" names="setSeed" param-count="1"/>
    <param pos="1" name="seed" types=".*" traced="true" />
</function>

For the above XML rules, the tags and attributes are interpreted as follows:

Tag name: function
Attribute name: name
Purpose: The fully qualified path to the method including package name, namespace, class and method name

Tag name: function
Attribute name: title
Purpose: A short title given to a vulnerability that can result from this method.

Tag name: function
Attribute name: description
Purpose: A description of the method and why it could lead to a vulnerability.

Tag name: signature
Attribute name: prefix-types
Purpose: Contains a regular expression which can be used to match the class name to which the input function belongs. This should be formatted as ".*?ClassName" so that any usage will be used, whether the class name is fully qualified (e.g. with package and namespace present), partially (just namespace, or inner namespace), or not qualified (e.g. imported only by name).

Tag name: signature
Attribute name: names
Purpose: Contains a regular expression which may be used to match the function name. This should just contain the function name with no further qualification (e.g. "read").

Tag name: signature
Attribute name: param-count
Purpose: This represents the number of parameters expected by the function in question. Since there are multiple functions in the same class with identical names this can help disambiguate. If there are variable numbers of parameters instead of the number of parameters a range can be specified (e.g. 1-3 for one, two or three parameters; 3-* for three or more parameters).

Tag name: param
Attribute name: pos
Purpose: This represents the position (with the first parameter being at position 1) of the parameter.

Tag name: param
Attribute name: name
Purpose: This is the name of the parameter, usually taken from documentation. This is not matched against anything it is purely specified to make understanding the rule easier.

Tag name: param
Attribute name: types
Purpose: Contains a regular expression which can match the parameter type (e.g. "char", "int", "byte\[\]", ".*?BufferedReader"). If the type is a plain-old type (built in Java type, not a class) then the type is specified as a literal (e.g. "int"). If the type is a class then only the class name is specified and the fully qualified name is not used (e.g. "java.io.BufferedReader" is specified as ".*?BufferedReader") and the ".*?" regex prefix is used behind the class name to ensure fully qualified names still match. If the type is an array the square braces are escaped (e.g. "int\[\]", ".*?BufferedReader\[\]"). If the type is a generic specialization (e.g. "List<string>") then ".*" is used as the type name instead, so types=".*?List<string>" would not be specified and instead types=".*" would be used). For built in types which are actually classes (e.g. "String") the class name should be specified with a ".*?" prefix (e.g. ".*?String").

Tag name: param
Attribute name: traced
Purpose: If the parameter value would cause a security vulnerability to occur if it was tainted with user-supplied input (for example, the query string parameter passed to an SQL method) this value should be set to "true" otherwise it should be omitted.

Please don't respond to this message.
```

Then:

```
Please generate XML adhering to the strict specification provided for all methods within the class {class-name} that could represent a danger if tainted or malicious user-supplied inputs were supplied to its parameters.

Please understand that I want ALL relevant methods that may result in a vulnerability and not just one method unless there is only one relevant method. If in doubt produce more rather than less output.

Output the XML as a single block with no comments or commentary. Please do not create any XML tags or attributes outside of what you have been instructed in the prompt or have deduced from the example XML.
```

### Annotation Sources

Define the XML format for ChatGPT:

```
The following XML describes an annotation source rule for a SAST product:

<annotation name="RULE_NAME" languages="java" categories="*" description="RULE_DESCRIPTION">
	<definition scope="SCOPE" value="REGEX" action="ACTION" ignore-case="false" />
</annotation>

The annotation attributes have the following meanings:

	name - the name of the source (usually the associated annotation name followed by "Source")
	languages - the language to which the source applies (this should just be "java")
	categories - this is always set to "*"
	description - a short description of the source
	
The definition tag attributes have the following meanings:

	scope - the scope at which the annotation is usually applied; this can be one of: class,interface,constructor,method,field,parameter,variable
	value - a regular expression to match the annotation rule name (usually just `.*?AnnotationClassName.*`)
	action - once matched the action can be one of the following values: taint,taint-members,taint-instance
	
The scope attribute values above have the following meanings:

	class - the attribute is applied at the class level (directly before the class declaration)
	interface - the attribute is applied at the interface level (directly before the interface declaration)
	constructor - the attribute is applied at the constructor level (directly before the constructor declaration)
	method - the attribute is applied at the method level (directly before the method declaration)
	field - the attribute is applied to class data member variables (directly before the member declaration)
	parameter - the attribute is applied to function parameters (directly before the parameter declaration_
	variable - the attribute is applied to local variables (directly before the variable declaration)

The action attribute values above have the following meanings:

	taint - once matched the rule taints the class instance (if scope is class,interface) or the method return value (if scope is constructor,method) or immediate data member, parameter or variable (if field,parameter,variable)
	taint-members - once matched the rule taints the class members
	taint-instance - once matched the rule taints the class instance variable
	
Multiple values can be supplied for scope or action as needed. If in doubt all valid values should be applied for these attributes (as a comma separated list).

Annotation rules validate annotations that are applied to classes, methods or parameters/variables based on those code objects being annotated.

For example the following Spring route has three annotations:

@RestController
public class MyController {

    @GetMapping("/greet")
    public String greet(@RequestParam(name = "name", defaultValue = "World") String name) {
        return "Hello, " + name + "!";
    }
}

The annotation @RestController is at the class level, @GetMapping is at the method level, and @RequestParam at the parameter/variable level.

To simplify creation of rules, the following three templates should be used:

For annotations applied at the class or interface level the following XML:

<annotation name="RULE_NAME" languages="java" categories="*" description="DESCRIPTION">
	<definition scope="class,interface" value="REGEX" action="taint,taint-members,taint-instance" ignore-case="false" />
</annotation>

For annotations applied at the constructor or method level the following XML:

<annotation name="RULE_NAME" languages="java" categories="*" description="DESCRIPTION">
	<definition scope="constructor,method" value="REGEX" action="taint" ignore-case="false" />
</annotation>

For annotations applied at the field, parameter or variable level the following XML:

<annotation name="RULE_NAME" languages="java" categories="*" description="DESCRIPTION">
	<definition scope="field,parameter,variable" value="REGEX" action="taint" ignore-case="false" />
</annotation>

For example, with the Java code previously supplied, rules for the three annotations would be:

<annotation name="RestControllerAnnotationSource" languages="java" categories="*" description="Spring MVC RestController annotation source possibly processing user supplied inputs.">
	<definition scope="class,interface" value=".*?(RestController).*" action="taint,taint-members,taint-instance" ignore-case="false" />
</annotation>

<annotation name="GetMappingAnnotationSource" languages="java" categories="*" description="Spring MVC GetMapping route annotation source possibly processing user supplied inputs.">
	<definition scope="constructor,method" value=".*?(GetMapping).*" action="taint" ignore-case="false" />
</annotation>

<annotation name="RequestParamAnnotationSource" languages="java" categories="*" description="Spring MVC RequestParam varluable annotation source from a potentially user supplied source.">
	<definition scope="field,parameter,variable" value=".*?(RequestParam).*" action="taint" ignore-case="false" />
</annotation>
```

