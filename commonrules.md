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

The format of these rules matches the format used for sinks, with the following exceptions:

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

**return**

The return tag is not valid for sinks, as they validate inputs only.

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

* traced - ignored because it only applies to dynamic (dataflow) analysis.
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
| title | This is a text description of the source used for reporting purposes. | "Reads from VarName." |
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

These rules take exactly the same format as the source rules with the following difference:

* traced - the field specifies whether the correponding input (source) parameter should have been traced.

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

* traced - ignored because it only applies to dynamic (dataflow) analysis.

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
| title | This is a text description of the source used for reporting purposes. | "SQL data read." |
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

* traced - the field specifies whether the data should be considered only if combined in an expression with traced source input (for example, `"SELECT\s+.\*" + value` is a sink only if value is traced)

#### Static Rules

These rules are located in commonrules.xml at path `/wsast/simple-data/static`.

```
<data name="SqlStatic1" languages="wsil, *" categories="SQL_INJECTION" title="SQL Query Static" description="Writes data containing a SQL query">
	<definition types="string, .*" value="(SELECT|UPDATE)\s+.*" />
</data>
```

These rules take exactly the same format as the sink rules, however the following fields are ignored:

* traced - ignored because it only applies to dynamic (dataflow) analysis.

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
		<sink relax-prefix-types="false" relax-param-types="true" relax-linked-params="false" />
		<static relax-prefix-types="false" relax-param-types="true" />
	</function>
	<variable>
		<source relax-prefix-types="false" />
		<sink relax-prefix-types="false" />
		<static relax-prefix-types="false" />
	</variable>
	<data>
		<source relax-types="false" />
		<sink relax-types="false" />
		<static relax-types="false" />
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

**function/relax-param-types**

Relaxes the requirement that parameter types must match.

**function/relax-linked-params**

Relaxes the requirement that parameter relationships described by the `linked-param` field must match.

**data/relax-types**

Relaxes the requirement that underlying data types must match.

#### Subscribers

**control**

* exclude-sources - a list of sources which should be excluded if matched; this may be to reduce output from any particular source.
* exclude-sinks - a list of sinks which should be excluded if matched; this may be to reduce output from any particular sink.
* report-sink-once - ensure that each individual sink is reported in association with a finding only once; certain sinks with many matching code paths can generate lots of almost identical reports and this setting helps prevent that.

## Semantic/Syntactic Rules

In addition to the XML format rules, the Common Rules Engine also provides a number of built-in checks. **These are still work in progress and do not yet provide (anywhere near) complete coverage of CWE or similar.**

### Existing Rules

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
The following XML describes a function/method source rule for a SAST product:

<function name="GetInputSource" languages="java" report="true" categories="SQL_INJECTION, *" description="The function reads untrusted user input.">
	<signature prefix-types=".*?InputClass" virtual="true" names="getInput" param-count="1" />
	<param pos="1" name="output" types="byte\[\]" traced="true" />
	<return types="int32" virtual="true" traced="true" />
</function>

The function described by this source has the following specification:

public class InputClass { public int32 getInput(byte[] output); }

The function tag attributes have the following meanings:

	name - the name of the source (usually the associated method name followed by "Source")
	languages - the language to which the source applies, this can be a comma separate list if it applies to multiple languages
	categories - this is always set to "*"
	description - a short description of the source

The signature tag attributes have the following meanings:

	prefix-types - a comma separated list of regular expressions matching classes to which the method belongs
	virtual - this is always set to "true"
	names - a comma separated list of regular expressions matching the names or aliases of the methods involved
	param-count - a comma separated list of the numbers of expected parameters for the method; this helps bundle overloaded methods in the name entry. The format can be ranges such as 1 if the function takes only parameter, 2-3 if the function takes exactly two or three parameters, or 3-* if the function takes a variable number of parameters, in this example 3 or more parameters.

The param tag attributes have the following meanings:

	pos - the index of the parameter being described, from 1
	name - the name of the parameter
	types - a comma separated list of regular expressions matching the type of the parameter
	virtual - this is always set to "true"
	traced - this is set to "true" if the parameter can be tainted by the source function

The return tag represents the function return value; its attributes have the following meanings:

	types - a comma separated list of regular expressions matching the type of the parameter
	virtual - this is always set to "true"
	traced - this is set to "true" if the parameter can be tainted by the source function

If the function tag prefix-types has a value then this should be only the class name prefixed by .*? to match any partially qualified name, for example java.sql.Statement and sql.Statement and Statement should match, so ".*?Statement" is the correct value to supply for the method java.sql.Statement.executeQuery. This rule for prefix-types should also be applied to parameter types, so for example a parameter type value matching "java.lang.String" should result in ".*?String" as the value stored in the param tag types value. When producing types, access modifiers such as private, public, const etc. should be omitted. Treat pointers as arrays (so char* becomes char[]). Escape array parenthesis [] as \[\]. Treat constructors as methods. Include inherited methods if they are public.
```

Then:

```
Generate me SAST source XML according to the specification provided, for methods and constructors within the class java.io.BufferedReader. Please output as a single block of XML, without comments. Include all relevant methods even if it takes some time, and all overloads. Only include methods which can return user supplied data, so ignore methods that just return other state and methods like Close() etc.
```



### Function Sinks

Define the XML format for ChatGPT:

```
The following XML describes a function/method sink rule for a SAST product:

<function name="ExecuteQuerySink" languages="java" report="true" categories="SQL_INJECTION, *" title="Possible SQL injection." description="The function has been known to result in SQL injection vulnerabilties when processing untrusted inputs.">
	<signature prefix-types=".*?Statement" virtual="true" names="executeQuery" param-count="1" />
	<param pos="1" name="sql" types=".*?String"/>
</function>

The function tag attributes have the following meanings:

	name - the name of the sink (usually the associated method name followed by "Sink")
	languages - the language to which the sink applies, this can be a comma separate list if it applies to multiple languages
	report - this is always set to "true"
	categories - this is always set to "*"
	title - a title for the issue when reported
	description - a short description of the sink and why it may pose a risk

The signature tag attributes have the following meanings:

	prefix-types - a comma separated list of regular expressions matching classes to which the method belongs
	virtual - this is always set to "true"
	names - a comma separated list of regular expressions matching the names or aliases of the methods involved
	param-count - a comma separated list of the numbers of expected parameters for the method; this helps bundle overloaded methods in the name entry. The format can be ranges such as 1 if the function takes only parameter, 2-3 if the function takes exactly two or three parameters, or 3-* if the function takes a variable number of parameters, in this example 3 or more parameters.

The param tag attributes have the following meanings:

	pos - the index of the parameter being described, from 1
	name - the name of the parameter
	types - a comma separated list of regular expressions matching the type of the parameter
	virtual - this is always set to "true"
	traced - this is set to "true" if the parameter should be traced (user-supplied) input to trigger a vulnerability

If the function tag prefix-types has a value then this should be only the class name prefixed by .*? to match any partially qualified name, for example java.sql.Statement and sql.Statement and Statement should match, so ".*?Statement" is the correct value to supply for the method java.sql.Statement.executeQuery. This rule for prefix-types should also be applied to parameter types, so for example a parameter type value matching "java.lang.String" should result in ".*?String" as the value stored in the param tag types value. When producing types, access modifiers such as private, public, const etc. should be omitted. Treat pointers as arrays (so char* becomes char[]). Escape array parenthesis [] as \[\]. Treat constructors as methods. Include inherited methods if they are public.

If there are multiple parameters to a method which could be tainted inputs leading to a vulnerability, then duplicate the XML for each parameter, setting the param tag attribute traced="true" for each in turn, for example for method TestClass.Example(), instead of outputting:

<function name="ExampleSink" languages="java" report="true" categories="LOG4SHELL, *"
    title="Potential vulnerability." description="The function has been known to be vulnerable.">
    <signature prefix-types=".*?TestClass" virtual="true" names="Example" param-count="1-*" />
    <param pos="1" name="foo" types=".*?String" traced="true" />
    <param pos="2" name="bar" types=".*?String" traced="true" />
</function>

Please output:

<function name="ExampleSink" languages="java" report="true" categories="LOG4SHELL, *"
    title="Potential vulnerability." description="The function has been known to be vulnerable.">
    <signature prefix-types=".*?TestClass" virtual="true" names="Example" param-count="1-*" />
    <param pos="1" name="foo" types=".*?String" traced="true" />
    <param pos="2" name="bar" types=".*?String" />
</function>


<function name="ExampleSink" languages="java" report="true" categories="LOG4SHELL, *"
    title="Potential vulnerability." description="The function has been known to be vulnerable.">
    <signature prefix-types=".*?TestClass" virtual="true" names="Example" param-count="1-*" />
    <param pos="1" name="foo" types=".*?String" />
    <param pos="2" name="bar" types=".*?String" traced="true" />
</function>
```

