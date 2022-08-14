# MockService
Java Swing application to create and run mockservices.

The objective for the MockService is to easily create a mockservice for any kind of project.
It generates its response messages using one or more templates. The template can among other information access the payload message from the request.
To use elements from the payload you make use of the variables, that can have xpath or jsonpath selecting values and place them in variables.

The mockservice functions in the following way:

- Receive a message and determine the method and content-type
- When echo is selected just copy all headers and payload to response

Otherwise
- Resolve all variables using received headers and payload
- Add the headers from the UI headerTable after resolving all headers that use variables
- Parse all templates where the expression are valid.
    - the last successful parsed template determines the responsecode and contenttype of the response
- when all template expressions are invalid the freemarker fallback template is used with the responsecode an contentype that are defined with that

## Table of Contents.
- [MockService](#mockservice)
  - [control-panel](#control-panel)
  - [context-panel](#context-panel)
  - [log-panel](#log-panel)
  - [header-panel](#header-panel)
  - [variables-panel](#variables-panel)
  - [templates-panel](#templates-panel)
  - [Headers](#headers)
  - [Loopback](#loopback)
  - [Variables](#variables)
  - [jsonpath](#jsonpath)
  - [Attachments](#attachments)
- [templates](#templates)
  - [Expressions](#expressions)
  - [Template specific headers](#template-specific-headers)
    - [Expression examples](#expression-examples)
  - [Fallback template](#fallback-template)
  - [Freemarker support tools](#freemarker-support-tools)
      - [Load a file from disk from within your template.](#load-a-file-from-disk-from-within-your-template)
      - [Load a file and return its Object representation.](#load-a-file-and-return-its-object-representation)
      - [Delete a file.](#delete-a-file)
      - [Base64n Encode and Decode Strings.](#base64n-encode-and-decode-strings)
      - [Convert xml to json or vise versa.](#convert-xml-to-json-or-vise-versa)
      - [Parse xml or json string to Object representation.](#parse-xml-or-json-string-to-object-representation)
- [Save and Load](#save-and-load)

![Overview](https://github.com/mvturnho/Mockservice-Release/raw/main/images/Overview.png)

The userinterface contains several functional components:
## control-panel

![Overview](https://github.com/mvturnho/Mockservice-Release/raw/main/images/control_panel.png)

From here you can load and save your mockservice configurations. By clicking on the title you may change it so you have an indication what the current running instance of the Mockservice is servicing

## context-panel

![Overview](https://github.com/mvturnho/Mockservice-Release/raw/main/images/context_panel.png)

With the contextpanel you can configure the supported protocol, port and contextpath and if the service should behave like a loopback service.

## log-panel

![Overview](https://github.com/mvturnho/Mockservice-Release/raw/main/images/log_panel.png)

The log panel contains the history of all received and processed massages.

## header-panel

![Overview](https://github.com/mvturnho/Mockservice-Release/raw/main/images/header_panel.png)

## variables-panel

![Overview](https://github.com/mvturnho/Mockservice-Release/raw/main/images/var1_panel.png)

![Overview](https://github.com/mvturnho/Mockservice-Release/raw/main/images/var2_panel.png)

![Overview](https://github.com/mvturnho/Mockservice-Release/raw/main/images/varcsv_dialog.png)

## templates-panel

![Overview](https://github.com/mvturnho/Mockservice-Release/raw/main/images/template_panel.png)

![Overview](https://github.com/mvturnho/Mockservice-Release/raw/main/images/templ_expr.png)

![Overview](https://github.com/mvturnho/Mockservice-Release/raw/main/images/templ_respc.png)

![Overview](https://github.com/mvturnho/Mockservice-Release/raw/main/images/templ_conttype.png)

![Overview](https://github.com/mvturnho/Mockservice-Release/raw/main/images/templ_headers.png)

## Headers
In the Headers tab you can add headers to the response message. With copy-headers selected the request headers are copied to the response message.
When you want to use a variable as a header just use `$variable_name`
You may also concatenate these variable names to form one header value.

|header   | value            |
|---------|------------------|
|`header1`| `$var1$var2$var3`|

## Loopback
When the loopback checkbox is selected, the service functions as a loopback service. So every received message is returned as the response, including all headers.

## Variables
In your templates and groovy scripts you always have access to the following default variables;

|variable           | content       |
|-------------------|---------------|
| `version`         | The version of the Mockservice Tool|
| `templateName`    | the name of the template |
| `payload`         | the message payload from the request |
| `templateFunction`| is the resolver to call functions defined in the variables tab`|
| `method`          | the method of the received request  |
| `context`         | contextpath of the request message  |
| `contentType`     | the contenttype of the received payload message |
| `messagePathInfo` | when using a wildchar * in your contextPath, messagePathInfo is the part that is contained in the wildchar *|
| `templateDir`     | The base templatedir from where the tool loads its files |
| `url_xxxxx`       | when the request has queryparameters they are exposed as variables with the url_ prefix |


In the variables tab you can use a variety of options to make data available to the freemarker or xslt templates.
This allows you to set variables or use functions and add namespaces if needed.

the variable always has a variable_name. This name can then be directly accessed from the template.
The value of the variable can be just a string or any of the following indicators.

| variable            | expression                           |                                                                                      |
|---------------------|--------------------------------------|--------------------------------------------------------------------------------------|
| `name`                  | `const://Harry `                         | fill variable name with constant value Harry.                                          |
| `action`                | `transport://my_action?none`             | getst the value of header my_action into the action variable. Default value is none  |
| `responsestring`        | `file://response.xml`                    | places the content of the file response.xml in the responsestring variable           |
| `xmlns://stuf`          | `http://www.egem.nl/StUF/sector/bg/0310` | add a namespace for stufbg to the namespace resolver, this is needed for the xpath |
| `elementname`           | `xpath:////stufbg:identificatie[0]?unknown` | xpath value for the given xpath expression           | 
| `bookname`              | `jsonpath:////code?geen_waarde`          | the jsonpath works just like xpath only on a json payload. This payload should have a root element |
| `fileatt`               | `attachment://attname?not found`         | add the attachment of the attachment in the request to the template variable You |
| `ordernumber`           | `expression://context.substring(context.lastIndexOf("/")+1,context.length())` | the expression is evaluated and put in the var |
| `businesskey`           | `templatevar://action`                   | set the businesskey. This is visible at the history table items |
| `function://func_name`  | `groovy://test.groovy`                   | initialize the function with name funct_name. This function can then be called from the Freemarker template with tunnelFunction |
| `var_name`              | `function://func_name/action`            | evaluate the function func_name and store the result in variable var_name |
| `prefix`                | `context:///mock/v1/api/test/{uid}/user/{zid}` | get the elements from the received uri when the contextpath matches, place the value in prefix_uid and prefix_zid variables |
| `mapping://map_name`    | `file://mapping.csv`                     | create a mapping with name map_name and fil its key/values with the content of the file |
| `var_name`              | `function://mapping/var_name/key`        | fil variable var_name with the value found in de mapping map_name for the given key (key is a variable) |
| `init://var`            | `10`                                     | Init this variable with name var only when it does not have a value yet |
| `int_var`               | `inc://10`                               | Increment the value of int_var with 10 |
| `int_var`               | `dec://10`                               | Decrement the value of int_var with 10 |
| `int_var`               | `mul://10`                               | Multiply the value of int_var by 10 |
| `int_var`               | `div://10`                               | Divide the value of int_var by 10 |

## jsonpath
JsonPath expressions always refer to a JSON structure in the same way as XPath expression are used in combination
with an XML document.

## Attachments
When you use a variable to get the attachment from a multipart message like so:

`varname  attachment://test`

Then the resulting variable is the attachment object. From your template you can access the following methods;

- `name`
- `contentType`
- `content`
- `size`
- `hexBytes`

So from you template you may use:

`${test}` Just use the content of the attachment

`${varname.contentType}` Results in the contenttype

`${varname.name}` Attachment name in the multipartformdata message

`${varname.size}` Size of the attachment



# templates

In the templates tab you have a 'template file directory'. All templates are loaded from this directory.
You can add templates that need to be used to generate a response for the mockserver. Based on the expression the template is evaluated or skipped. When no templates were applicable 
the fallbacktemplate is used.

Every template defines a template-file, expression, value, responsecode and a content-type for the response.

When 'evaluate all templates' is selected the parser continues to evaluate the next template when the expression is valid. So when an expression evaluates to be valid, 
the template is applied on the payload. When the next expression is valid that template is applied to the resulting payload of the previous template.

For templates the following filetypes are support
- freemarker .ftl
- Xslt .xsl
- Groovy .groovy
- all other files as plain text files 

## Expressions

for template evaluation we use MVEL2 http://mvel.documentnode.com/

You can leave out the @{} the expression evalueates like java style syntax.
For strings (templatevars) you may use all String object methods

When no expression is defined the template will always be processed, unless "evaluate all templates" is off and 
another template was already processed.

## Template specific headers

Per template row you can define specific headers that will be added to the response when the expression is valid.

### Expression examples

- `context.endsWith("samenwerken")` value is then `true` or `false`
- `method.equals("POST")`
- `context.contains("verwerken")"`
- `context.indexOff("test")` value could be 20 when the text should occur on that position in the contextpath

## Fallback template

The Fallback template is a Freemarker template and can be edited from the editor in the bottom, also you define the Fallback responsecode and contenttype here.

## Freemarker support tools

Within the freemarker template factory ther is a special toolset that can be used from the tools refference.

#### Load a file from disk from within your template. 

This can also be done using the variables, but in some cases you may 
only need the file from within a specific template.

`<#assign xml_text = tools.loadFile("filename.xml")>`

#### Load a file and return its Object representation.

`<#assign xmlNodeModel = tools.load("filename.xml")>`  Loads a file into the xml NodeModel.

`<#assign jsonObject = tools.load("filename.json")>` Loads the json file into a HashMap Object.

#### Delete a file.

`<#assign result = tools.deleteFile("fileName")>`

#### Base64n Encode and Decode Strings.

`<#assign encodedString = tools.b64Encode(contentString)>`

`<#assign decodedString = tools.b64Decode(encodedString)>`

#### Convert xml to json or vise versa.

`<#assign jsonString = tools.xmlToJson(xmlString)>`
`<#assign xmlString = tools.jsonToXml(jsonString)>` 

#### Parse xml or json string to Object representation.

`<#assign jsonObject = tools.parseJson("jsonString")>`
`<#assign xmlObject = tools.parseXml("xmlString")>`

# Save and Load

You can save and load your configurations. The directory for the save file should be the same as where the templates reside.
This way the mockconfigurations are portable between systems.
