<html><head>
<title> XML
</title>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<link href="../stylesheet.css" rel="stylesheet" type="text/css">

<style type="text/css">
    body { counter-reset: chapter 12; }
</style>

<script type="text/javascript" async="" src="http://api.flattr.com/js/0.6/load.js?mode=auto"></script><script type="text/javascript" src="../toc.js"> 
   <!-- empty -->
</script>

<script type="text/javascript">
/* <![CDATA[ */
    (function() {
        var s = document.createElement("script"), t = document.getElementsByTagName("script")[0];
        s.type = "text/javascript";
        s.async = true;
        s.src = "http://api.flattr.com/js/0.6/load.js?mode=auto";
        t.parentNode.insertBefore(s, t);
    })();
/* ]]> */
</script>


</head>

<body>

<div class="chapter">
<h1> XML </h1>
</div>

<div id="generated-toc" class="generate_from_h2"><p><span class="hidden"><a href="#aftertoc">skip table of contents</a></span></p><p id="toggle-container"><a id="generated_toc_d_toggle" href="#">Show table of contents</a></p><ul style="display: none;"><li class="notmissing"><a href="#heading_toc_j_0">Introduction</a></li><li class="notmissing"><a href="#heading_toc_j_1">Parsing XML</a></li><li class="notmissing"><a href="#heading_toc_j_2">Unmarshalling XML</a></li><li class="notmissing"><a href="#heading_toc_j_3">Marshalling XML</a></li><li class="notmissing"><a href="#heading_toc_j_4">XHTML</a></li><li class="notmissing"><a href="#heading_toc_j_5">HTML</a></li><li class="notmissing"><a href="#heading_toc_j_6">Conclusion</a></li></ul><p id="aftertoc"></p></div>

<div class="preface">
<p>
  XML is a significant markup language mainly intended as a means
  of serialising data structures as a text document.
  Go has basic support for XML document processing.
</p>
</div>

<h2 id="heading_toc_j_0" tabindex="-1">Introduction</h2>
<p>
  XML is now a widespread way of representing complex data structures
  serialised into text format. It is used to describe documents
  such as DocBook and XHTML. It is used in specialised markup
  languages such as MathML and CML (Chemistry Markup Language).
  It is used to encode data as SOAP messages for Web Services,
  and the Web Service can be specified using WSDL (Web Services
  Description Language). 
</p>

<p>
  At the simplest level, XML allows you to define your own tags
  for use in text documents. Tags can be nested and can be
  interspersed with text. Each tag can also contain attributes
  with values. For example, 
  <code>
    </code></p><pre><code>&lt;person&gt;
  &lt;name&gt;
    &lt;family&gt; Newmarch &lt;/family&gt;
    &lt;personal&gt; Jan &lt;/personal&gt;
  &lt;/name&gt;
  &lt;email type="personal"&gt;
    jan@newmarch.name
  &lt;/email&gt;
  &lt;email type="work"&gt;
    j.newmarch@boxhill.edu.au
  &lt;/email&gt;
&lt;/person&gt;
    </code></pre><code>
  </code>
<p></p>

<p>
  The structure of any XML document can be described in a number of ways:
  </p><ul>
    <li>
      A document type definition DTD is good for describing structure
    </li>
    <li>
      XML schema are good for describing the data types used by an
      XML document
    </li>
    <li>
      RELAX NG is proposed as an alternative to both
    </li>
  </ul>
<p></p>

<p>
  There is argument over the relative value of each way of defining
  the structure of an XML document. We won't buy into that, as Go
  does not suport any of them. Go cannot check for validity of any
  document against a schema, but only for well-formedness.
</p>

<p>
  Four topics are discussed in this chapter: parsing an XML stream,
  marshalling and unmarshalling Go data into XML, and XHTML.
</p>

<h2 id="heading_toc_j_1" tabindex="-1"> Parsing XML </h2>
<p>
  Go has an XML parser which is created using <code>NewParser</code>.
  This takes an <code>io.Reader</code> as parameter and returns a pointer
  to <code>Parser</code>. The main method of this type is
  <code>Token</code> which returns the next token in the input
  stream. The token is one of the types <code>StartElement</code>,
  <code>EndElement</code>, <code>CharData</code>, <code>Comment</code>,
  <code>ProcInst</code> or <code>Directive</code>.
</p>

<p>
  The types are
  </p><dl>
    <dt>
      <code>StartElement</code>
    </dt>
    <dd>
      The type <code>StartElement</code> is a structure with two
      field types:
      <code>
	<pre>type StartElement struct {
    Name Name
    Attr []Attr
}

type Name struct {
    Space, Local string
}

type Attr struct {
    Name  Name
    Value string
}
	</pre>
      </code>
    </dd>
    <dt>
      <code>EndElement</code>
    </dt>
    <dd>
      This is also a structure
      <code>
	<pre>type EndElement struct {
    Name Name
}
	</pre>
      </code>
    </dd>
    <dt>
      <code>CharData</code>
    </dt>
    <dd>
      This type represents the text content enclosed by a tag
      and is a simple type
      <code>
	<pre>type CharData []byte
	</pre>
      </code>
    </dd>
    <dt>
      <code>Comment</code>
    </dt>
    <dd>
      Similarly for this type
      <code>
	<pre>type Comment []byte
	</pre>
      </code>
    </dd>
    <dt>
      <code>ProcInst</code>
    </dt>
    <dd>
      A ProcInst represents an XML processing instruction of the 
      form &lt;?target inst?&gt;
      <code>
	<pre>type ProcInst struct {
    Target string
    Inst   []byte
}
	</pre>
      </code>
    </dd>
    <dt>
      <code>Directive</code>
    </dt>
    <dd>
      A Directive represents an XML directive of the form 
      &lt;!text&gt;. The bytes do not include the &lt;! and &gt; markers.
      <code>
	<pre>type Directive []byte
	</pre>
      </code>
    </dd>
  </dl>
<p></p>

<p>
  A program to print out the tree structure of an XML document is
  </p><pre><code><font color="black">
<font color="firebrick">/* Parse XML
 */
</font>
<font color="purple">package</font> main

<font color="purple">import</font> (
	<font color="DarkRed">"encoding/xml"</font>
	<font color="DarkRed">"fmt"</font>
	<font color="DarkRed">"io/ioutil"</font>
	<font color="DarkRed">"os"</font>
	<font color="DarkRed">"strings"</font>
)

<font color="purple">func</font> main() {
	<font color="purple">if</font> len(os.<font color="blue">Args</font>) != 2 {
		fmt.<font color="blue">Println</font>(<font color="DarkRed">"Usage: "</font>, os.<font color="blue">Args</font>[0], <font color="DarkRed">"file"</font>)
		os.<font color="blue">Exit</font>(1)
	}
	file := os.<font color="blue">Args</font>[1]
	bytes, err := ioutil.<font color="blue">ReadFile</font>(file)
	checkError(err)
	r := strings.<font color="blue">NewReader</font>(string(bytes))

	parser := xml.<font color="blue">NewDecoder</font>(r)
	depth := 0
	<font color="purple">for</font> {
		token, err := parser.<font color="blue">Token</font>()
		<font color="purple">if</font> err != nil {
			<font color="purple">break</font>
		}
		<font color="purple">switch</font> t := token.(<font color="purple">type</font>) {
		<font color="purple">case</font> xml.<font color="blue">StartElement</font>:
			elmt := xml.<font color="blue">StartElement</font>(t)
			name := elmt.<font color="blue">Name</font>.<font color="blue">Local</font>
			printElmt(name, depth)
			depth++
		<font color="purple">case</font> xml.<font color="blue">EndElement</font>:
			depth--
			elmt := xml.<font color="blue">EndElement</font>(t)
			name := elmt.<font color="blue">Name</font>.<font color="blue">Local</font>
			printElmt(name, depth)
		<font color="purple">case</font> xml.<font color="blue">CharData</font>:
			bytes := xml.<font color="blue">CharData</font>(t)
			printElmt(<font color="DarkRed">"\""</font>+string([]<font color="green">byte</font>(bytes))+<font color="DarkRed">"\""</font>, depth)
		<font color="purple">case</font> xml.<font color="blue">Comment</font>:
			printElmt(<font color="DarkRed">"Comment"</font>, depth)
		<font color="purple">case</font> xml.<font color="blue">ProcInst</font>:
			printElmt(<font color="DarkRed">"ProcInst"</font>, depth)
		<font color="purple">case</font> xml.<font color="blue">Directive</font>:
			printElmt(<font color="DarkRed">"Directive"</font>, depth)
		<font color="purple"><font color="purple">default</font></font>:
			fmt.<font color="blue">Println</font>(<font color="DarkRed">"Unknown"</font>)
		}
	}
}

<font color="purple">func</font> printElmt(s string, depth <font color="green">int</font>) {
	<font color="purple">for</font> n := 0; n &lt; depth; n++ {
		fmt.<font color="blue">Print</font>(<font color="DarkRed">"  "</font>)
	}
	fmt.<font color="blue">Println</font>(s)
}

<font color="purple">func</font> checkError(err error) {
	<font color="purple">if</font> err != nil {
		fmt.<font color="blue">Println</font>(<font color="DarkRed">"Fatal error "</font>, err.<font color="blue">Error</font>())
		os.<font color="blue">Exit</font>(1)
	}
}
</font></code></pre>

  <em>Note that the parser includes all CharData, including the
  whitespace between tags.</em>
<p></p>

<p>
  If we run this program against the <code>person</code> data
  structure given earlier, it produces
<code>
</code></p><pre><code>person
  "
  "
  name
    "
    "
    family
      " Newmarch "
    family
    "
    "
    personal
      " Jan "
    personal
    "
  "
  name
  "
  "
  email
    "
    jan@newmarch.name
  "
  email
  "
  "
  email
    "
    j.newmarch@boxhill.edu.au
  "
  email
  "
"
person
"
"
</code></pre><code>
</code>
Note that as no DTD or other XML specification has been used,
the tokenizer correctly prints out all the white space
(a DTD may specify that the whitespace can be ignored, but
without it that assumption cannot be made.)

<p>
  There is a potential trap in using this parser. It re-uses space
  for strings, so that once you see a token you need to copy its
  value if you want to refer to it later. Go has methods such as
  <code>func (c CharData) Copy() CharData</code> to make a copy
  of data.
</p>

<h2 id="heading_toc_j_2" tabindex="-1"> Unmarshalling XML </h2>
<p>
  Go provides a function <code>Unmarshal</code> and a method
  <code>func (*Parser) Unmarshal</code> to unmarshal XML into
  Go data structures. The unmarshalling is not perfect:
  Go and XML are different languages.
</p>

<p>
  We consider a simple example before looking at the details.
  We take the XML document given earlier of
  <code>
    </code></p><pre><code>&lt;person&gt;
  &lt;name&gt;
    &lt;family&gt; Newmarch &lt;/family&gt;
    &lt;personal&gt; Jan &lt;/personal&gt;
  &lt;/name&gt;
  &lt;email type="personal"&gt;
    jan@newmarch.name
  &lt;/email&gt;
  &lt;email type="work"&gt;
    j.newmarch@boxhill.edu.au
  &lt;/email&gt;
&lt;/person&gt;
    </code></pre><code>
  </code>
<p></p>

<p>
  We would like to map this onto the Go structures
  <code>
    </code></p><pre><code>type Person struct {
	Name Name
	Email []Email
}

type Name struct {
	Family string
	Personal string
}

type Email struct {
	Type string
	Address string
}
    </code></pre><code>
  </code>
  This requires several comments:
  <ol>
    <li>
      Unmarshalling uses the Go reflection package. This requires that
      all fields by public i.e. start with a capital letter.
      Earlier versions of Go used case-insensitive matching to match fields such
      as the XML string "name" to the field <code>Name</code>.
      Now, though, <em>case-sensitive</em> matching is used. To perform a
      match, the structure fields must be tagged to show the XML string that
      will be matched against. This changes <code>Person</code> to
      <code>
	<pre>type Person struct {
	Name Name `xml:"name"`
	Email []Email `xml:"email"`
}
	</pre>
      </code>
    </li>
    <li>
      While tagging of fields can attach XML strings to fields, it can't do
      so with the names of the structures. An additional field is required,
      with field name "XMLName". This only affects the top-level struct, 
      <code>Person</code>
      <code>
	<pre>type Person struct {
        XMLName Name `xml:"person"`
	Name Name `xml:"name"`
	Email []Email `xml:"email"`
}
	</pre>
      </code>
    </li>
    <li>
      Repeated tags in the map to a slice in Go
    </li>
    <li>
      Attributes within tags will match to fields in a structure only
      if the Go field has the tag ",attr". This occurs with the field
      <code>Type</code> of <code>Email</code>, where matching the attribute
      "type" of the "email" tag requires <code>`xml:"type,attr"`</code>
    </li>
    <li>
      If an XML tag has no attributes and only has character data, then
      it matches a <code>string</code> field by the same name
      (case-sensitive, though). So the tag <code>`xml:"family"`</code> with
      character data "Newmarch" maps to the string field <code>Family</code>
    </li>
    <li>
      But if the tag has attributes, then it must map to a structure.
      Go assigns the character data to the field with tag 
      <code>,chardata</code>. This occurs with the "email" data
      and the field <code>Address</code> with tag <code>,chardata</code>
    </li>
  </ol>
<p></p>

<p>
  A program to unmarshal the document above is
  </p><pre><code><font color="black">
<font color="firebrick">/* Unmarshal
 */
</font>
<font color="purple">package</font> main

<font color="purple">import</font> (
	<font color="DarkRed">"encoding/xml"</font>
	<font color="DarkRed">"fmt"</font>
	<font color="DarkRed">"os"</font>
	<font color="firebrick">//"strings"
</font>)

<font color="purple">type</font> <font color="blue">Person</font> <font color="purple">struct</font> {
	<font color="blue">XMLName</font> <font color="blue">Name</font>    `xml:<font color="DarkRed">"person"</font>`
	<font color="blue">Name</font>    <font color="blue">Name</font>    `xml:<font color="DarkRed">"name"</font>`
	<font color="blue">Email</font>   []<font color="blue">Email</font> `xml:<font color="DarkRed">"email"</font>`
}

<font color="purple">type</font> <font color="blue">Name</font> <font color="purple">struct</font> {
	<font color="blue">Family</font>   string `xml:<font color="DarkRed">"family"</font>`
	<font color="blue">Personal</font> string `xml:<font color="DarkRed">"personal"</font>`
}

<font color="purple">type</font> <font color="blue">Email</font> <font color="purple">struct</font> {
	<font color="blue">Type</font>    string `xml:<font color="DarkRed">"type,attr"</font>`
	<font color="blue">Address</font> string `xml:<font color="DarkRed">",chardata"</font>`
}

<font color="purple">func</font> main() {
	str := `&lt;?xml version=<font color="DarkRed">"1.0"</font> encoding=<font color="DarkRed">"utf-8"</font>?&gt;
&lt;person&gt;
  &lt;name&gt;
    &lt;family&gt; <font color="blue">Newmarch</font> &lt;/family&gt;
    &lt;personal&gt; <font color="blue">Jan</font> &lt;/personal&gt;
  &lt;/name&gt;
  &lt;email <font color="purple">type</font>=<font color="DarkRed">"personal"</font>&gt;
    jan@newmarch.name
  &lt;/email&gt;
  &lt;email <font color="purple">type</font>=<font color="DarkRed">"work"</font>&gt;
    j.newmarch@boxhill.edu.au
  &lt;/email&gt;
&lt;/person&gt;`

	<font color="purple">var</font> person <font color="blue">Person</font>

	err := xml.<font color="blue">Unmarshal</font>([]<font color="green">byte</font>(str), &amp;person)
	checkError(err)

	<font color="firebrick">// now use the person structure e.g.
</font>	fmt.<font color="blue">Println</font>(<font color="DarkRed">"Family name: \""</font> + person.<font color="blue">Name</font>.<font color="blue">Family</font> + <font color="DarkRed">"\""</font>)
	fmt.<font color="blue">Println</font>(<font color="DarkRed">"Second email address: \""</font> + person.<font color="blue">Email</font>[1].<font color="blue">Address</font> + <font color="DarkRed">"\""</font>)
}

<font color="purple">func</font> checkError(err error) {
	<font color="purple">if</font> err != nil {
		fmt.<font color="blue">Println</font>(<font color="DarkRed">"Fatal error "</font>, err.<font color="blue">Error</font>())
		os.<font color="blue">Exit</font>(1)
	}
}
</font></code></pre>

<p></p>

<p>
  (Note the spaces are correct.).
  The strict rules are given in the package specification.
</p>

<h2 id="heading_toc_j_3" tabindex="-1"> Marshalling XML </h2>

<p>
  Go 1 also has support for marshalling data structures into an
  XML document. The function is
  </p><pre>    <code>
func Marshal(v interface}{) ([]byte, error)
    </code>
  </pre>
  This was used as a check in the last two lines of the previous program.


<!--
<p>
  At present there is no support for marshalling a Go data structure
  into XML. In this section we present a simple marshalling
  function that will give
  a basic serialisation. The result can be unmarshalled using
  the Go function <code>Unmarshal</code> of the previous section.
</p>

<p>
  A straightforward but naive approach would be to write code that
  walks over your data structures, printing out results as it goes.
  But if is customised to your data types, then you wil need to change
  code each time the types change.
</p>

<p>
  A better approach, and one that is used by the Go serialisation
  libraries is to use the <code> reflection</code> package.
  This is a package that allows you to examine data types and
  data structures from within a running program. The idea of
  reflection has been present in artificial intelligence
  programming for many years, but is still seen as a rather arcane
  technique for mainstream languages.
</p>

<p>
  Go has two principal reflection types:  
  <code>reflect.Type</code> gives information about the Go types,
  while <code>reflect.Value</code> gives information about a
  particular data value. <code>Value</code> has a method
  <code>Type()</code> that can return the type.
</p>

<p>
  The simplest types and values correspond to primitive types.
  For example, there is <code>IntType</code>, <code>BoolType</code>
  etc, which can be used as values in type switches to determine the
  precise type of a <code>Type</code>. The corresponding value types
  are <code>IntValue</code> and <code>BoolValue</code> with 
  methods such as <code>Get</code> to return the value.
</p>

<p>
  A <code>StructType</code> is more complex, as it has methods
  to access the fields by 
  <code>
    <pre>
func (t *StructType) Field(i int) (f StructField)
    </pre>
  </code>
  and a <code>StructField</code> has methods such as
  <code>Name</code> to return the string value of the field's
  label. This is useful for examing the type structure.
</p>

<p>
  A <code>StructValue</code> is useful for examining the value
  of fields of a data value. It has a method
  <code>
    <pre>
func (v *StructValue) Field(i int) Value
    </pre>
  </code>
  which can be used to extract the value of each field.
</p>

<p>
  The reflection process is basically stsrted by calling
  <code>NewValue</code> on a data object, and then examining
  its type and recursively walking through the values.
  What we do with each value is to surround it by tags,
  made of field names of the structures encountered.
</p>

<p>
  There are two complexities: the first is that the initial
  data value will tpyically be a structure, and this doesn't
  have a field name as it is not itself part of a structure.
  For this starting case, we use the type name of the structure
  as XML tag name.
</p>

<p>
  The second complexity comes with arrays or slices. In this case
  we need to work through each element of the array/slice,
  each time repeating the field name from the enclosing
  structure.
</p>

<p>
  We define thre functions: <code>Marshal</code> which takes an
  initial data value. This prepares the XML document and creates
  the toplevel tag from the structure's type name.
  The second function <MarshalValue /> recurses through the
  type values, switching on data types and writing tags from
  field names and values as XML character data.
  The third function handles the special case of slices,
  as the tag name needs to be kept for all of the elements
  of this slice.
</p>

<p>
  We ignore pointers, channels, etc. We also do not produce
  attributes, just tags and character data.
  The program is
  <pre><code><font color="black">
<font color="firebrick">/* Marshal
 */
</font>
<font color="purple">package</font> main

<font color="purple">import</font> (
	<font color="DarkRed">"fmt"</font>
	<font color="DarkRed">"io"</font>
	<font color="DarkRed">"os"</font>
	<font color="DarkRed">"reflect"</font>
	<font color="DarkRed">"bytes"</font>
)

<font color="purple">type</font> <font color="blue">Person</font> <font color="purple">struct</font> {
	<font color="blue">Name</font>  <font color="blue">Name</font>
	<font color="blue">Email</font> []<font color="blue">Email</font>
}

<font color="purple">type</font> <font color="blue">Name</font> <font color="purple">struct</font> {
	<font color="blue">Family</font>   string
	<font color="blue">Personal</font> string
}

<font color="purple">type</font> <font color="blue">Email</font> <font color="purple">struct</font> {
	<font color="blue">Kind</font>    string <font color="DarkRed">"attr"</font>
	<font color="blue">Address</font> string <font color="DarkRed">"chardata"</font>
}

<font color="purple">func</font> main() {
	person := <font color="blue">Person</font>{
		<font color="blue">Name</font>: <font color="blue">Name</font>{<font color="blue">Family</font>: <font color="DarkRed">"Newmarch"</font>, <font color="blue">Personal</font>: <font color="DarkRed">"Jan"</font>},
		<font color="blue">Email</font>: []<font color="blue">Email</font>{<font color="blue">Email</font>{<font color="blue">Kind</font>: <font color="DarkRed">"home"</font>, <font color="blue">Address</font>: <font color="DarkRed">"jan"</font>},
			<font color="blue">Email</font>{<font color="blue">Kind</font>: <font color="DarkRed">"work"</font>, <font color="blue">Address</font>: <font color="DarkRed">"jan"</font>}}}

	buff := bytes.<font color="blue">NewBuffer</font>(nil)
	<font color="blue">Marshal</font>(person, buff)
	fmt.<font color="blue">Println</font>(buff.<font color="blue">String</font>())
}

<font color="purple">func</font> <font color="blue">Marshal</font>(e <font color="purple">interface</font>{}, w io.<font color="blue">Writer</font>) {
	<font color="firebrick">// make it a legal XML document
</font>	w.<font color="blue">Write</font>([]<font color="green">byte</font>(<font color="DarkRed">"&lt;?xml version=\"1.1\" encoding=\"UTF-8\" ?&gt;\n"</font>))

	<font color="firebrick">// topvel e is a value and has no structure field, 
</font>	<font color="firebrick">// so use its type
</font>	typ := reflect.<font color="blue">TypeOf</font>(e)
	name := typ.<font color="blue">Name</font>()

	startTag(name, w)
	<font color="blue">MarshalValue</font>(reflect.<font color="blue">ValueOf</font>(e), w)
	endTag(name, w)
}

<font color="purple">func</font> <font color="blue">MarshalValue</font>(v reflect.<font color="blue">Value</font>, w io.<font color="blue">Writer</font>) {
	t := v.<font color="blue">Type</font>()
	<font color="purple">switch</font> t.<font color="blue">Kind</font>() {
	<font color="purple">case</font> reflect.<font color="blue">Struct</font>:
		<font color="purple">for</font> n := 0; n &lt; t.<font color="blue">NumField</font>(); n++ {
			field := t.<font color="blue">Field</font>(n)

			vv := v

			<font color="firebrick">// special case if it is a slice
</font>
			<font color="purple">if</font> vv.<font color="blue">Field</font>(n).<font color="blue">Type</font>().<font color="blue">Kind</font>() == reflect.<font color="blue">Slice</font> {
				<font color="firebrick">// slice
</font>				<font color="blue">MarshalSliceValue</font>(field.<font color="blue">Name</font>,
					vv.<font color="blue">Field</font>(n), w)
			} <font color="purple">else</font> {
				<font color="firebrick">// not a slice
</font>				startTag(field.<font color="blue">Name</font>, w)
				<font color="blue">MarshalValue</font>(vv.<font color="blue">Field</font>(n), w)
				endTag(field.<font color="blue">Name</font>, w)
			}
		}
	<font color="purple">case</font> reflect.<font color="blue">Int</font>, reflect.<font color="blue">Int8</font>, reflect.<font color="blue">Int16</font>, reflect.<font color="blue">Int32</font>, reflect.<font color="blue">Int64</font>, reflect.<font color="blue">Uint</font>, reflect.<font color="blue">Uint8</font>, reflect.<font color="blue">Uint16</font>, reflect.<font color="blue">Uint32</font>, reflect.<font color="blue">Uint64</font>, reflect.<font color="blue">Uintptr</font>:
	<font color="purple">case</font> reflect.<font color="blue">Bool</font>:
	<font color="purple">case</font> reflect.<font color="blue">String</font>:
		vv := v
		w.<font color="blue">Write</font>([]<font color="green">byte</font>(<font color="DarkRed">"   "</font> + vv.<font color="blue">String</font>() + <font color="DarkRed">"\n"</font>))
	<font color="purple"><font color="purple">default</font></font>:
	}
}

<font color="purple">func</font> <font color="blue">MarshalSliceValue</font>(tag string, v reflect.<font color="blue">Value</font>, w io.<font color="blue">Writer</font>) {
	<font color="purple">for</font> n := 0; n &lt; v.<font color="blue">Len</font>(); n++ {
		startTag(tag, w)
		<font color="blue">MarshalValue</font>(v.<font color="blue">Index</font>(n), w)
		endTag(tag, w)
	}
}

<font color="purple">func</font> startTag(s string, w io.<font color="blue">Writer</font>) {
	w.<font color="blue">Write</font>([]<font color="green">byte</font>(<font color="DarkRed">"&lt;"</font> + s + <font color="DarkRed">"&gt;\n"</font>))
}

<font color="purple">func</font> endTag(s string, w io.<font color="blue">Writer</font>) {
	w.<font color="blue">Write</font>([]<font color="green">byte</font>(<font color="DarkRed">"&lt;/"</font> + s + <font color="DarkRed">"&gt;\n"</font>))
}

<font color="purple">func</font> checkError(err error) {
	<font color="purple">if</font> err != nil {
		fmt.<font color="blue">Println</font>(<font color="DarkRed">"Fatal error "</font>, err.<font color="blue">Error</font>())
		os.<font color="blue">Exit</font>(1)
	}
}
</font></code></pre>

-->
<p></p>

<h2 id="heading_toc_j_4" tabindex="-1"> XHTML </h2>
<p>
  HTML does not conform to XML syntax. It has unterminated tags
  such as '&lt;br&gt;'. XHTML is a cleanup of HTML to make it
  compliant to XML. Documents in XHTML can be managed using
  the techniques above for XML.
</p>

<h2 id="heading_toc_j_5" tabindex="-1"> HTML </h2>
<p>
  There is some support in the XML package to handle HTML documents even though
  they are not XML-compliant. The XML parser discussed earlier can handle many HTML
  documents if it is modified by
  </p><pre>    <code>
	parser := xml.NewDecoder(r)
	parser.Strict = false
	parser.AutoClose = xml.HTMLAutoClose
	parser.Entity = xml.HTMLEntity
    </code>
  </pre>


<h2 id="heading_toc_j_6" tabindex="-1"> Conclusion </h2>
<p>
Go has basic support for dealing with XML strings.
It does not as yet have mechanisms for dealing with XML
specification languages such as XML Schema or Relax NG.
</p>

<p> 
</p><hr>
Copyright © Jan Newmarch jan@newmarch.name
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/"><img alt="Creative Commons License" style="border-width:0" src="http://i.creativecommons.org/l/by-nc-sa/3.0/88x31.png"></a><br>This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/">Creative Commons Attribution-NonCommercial-ShareAlike 3.0 Unported License</a>.
<hr>

<p></p>
<p>If you like this book, please contribute using Flattr
<iframe src="http://button.flattr.com/view/?e=1&amp;url=http%3A%2F%2Fjan.newmarch.name%2Fgo%2Findex.html&amp;" class="FlattrButton" width="55" height="62" frameborder="0" scrolling="no" title="Flattr" border="0" marginheight="0" marginwidth="0" allowtransparency="true"></iframe>
<br> or donate using PayPal
</p><form action="https://www.paypal.com/cgi-bin/webscr" method="post">
<input type="hidden" name="cmd" value="_s-xclick">
<input type="hidden" name="encrypted" value="-----BEGIN PKCS7-----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-----END PKCS7-----
">
<input type="image" src="https://www.paypalobjects.com/WEBSCR-640-20110401-1/en_AU/i/btn/btn_donateCC_LG.gif" border="0" name="submit" alt="PayPal - The safer, easier way to pay online.">
<img alt="" border="0" src="https://www.paypalobjects.com/WEBSCR-640-20110401-1/en_AU/i/scr/pixel.gif" width="1" height="1">
</form>



</body></html>

# ASTAXIE

XML is a commonly used data communication format in web services. Today, it's assuming a more and more important role in web development. In this section, we're going to introduce how to work with XML through Go's standard library.

I will not make any attempts to teach XML's syntax or conventions. For that, please read more documentation about XML itself. We will only focus on how to encode and decode XML files in Go.

Suppose you work in IT, and you have to deal with the following XML configuration file:

	<?xml version="1.0" encoding="utf-8"?>
	<servers version="1">
	    <server>
	        <serverName>Shanghai_VPN</serverName>
	        <serverIP>127.0.0.1</serverIP>
	    </server>
	    <server>
	        <serverName>Beijing_VPN</serverName>
	        <serverIP>127.0.0.2</serverIP>
	    </server>
	</servers>

The above XML document contains two kinds of information about your server: the server name and IP. We will use this document in our following examples.

## Parse XML

How do we parse this XML document? We can use the `Unmarshal` function in Go's `xml` package to do this.

	func Unmarshal(data []byte, v interface{}) error

the `data` parameter receives a data stream from an XML source, and `v` is the structure you want to output the parsed XML to. It is an interface, which means you can convert XML to any structure you desire. Here, we'll only talk about how to convert from XML to the `struct` type since they share similar tree structures.

Sample code:

	package main
	
	import (
	    "encoding/xml"
	    "fmt"
	    "io/ioutil"
	    "os"
	)
	
	type Recurlyservers struct {
	    XMLName     xml.Name `xml:"servers"`
	    Version     string   `xml:"version,attr"`
	    Svs         []server `xml:"server"`
	    Description string   `xml:",innerxml"`
	}
	
	type server struct {
	    XMLName    xml.Name `xml:"server"`
	    ServerName string   `xml:"serverName"`
	    ServerIP   string   `xml:"serverIP"`
	}
	
	func main() {
	    file, err := os.Open("servers.xml") // For read access.     
	    if err != nil {
	        fmt.Printf("error: %v", err)
	        return
	    }
	    defer file.Close()
	    data, err := ioutil.ReadAll(file)
	    if err != nil {
	        fmt.Printf("error: %v", err)
	        return
	    }
	    v := Recurlyservers{}
	    err = xml.Unmarshal(data, &v)
	    if err != nil {
	        fmt.Printf("error: %v", err)
	        return
	    }
	
	    fmt.Println(v)
	}

XML is actually a tree data structure, and we can define a very similar structure using structs in Go, then use `xml.Unmarshal` to convert from XML to our struct object. The sample code will print the following content:

	{{ servers} 1 [{{ server} Shanghai_VPN 127.0.0.1} {{ server} Beijing_VPN 127.0.0.2}]
	<server>
	    <serverName>Shanghai_VPN</serverName>
	    <serverIP>127.0.0.1</serverIP>
	</server>
	<server>
	    <serverName>Beijing_VPN</serverName>
	    <serverIP>127.0.0.2</serverIP>
	</server>
	}

We use `xml.Unmarshal` to parse the XML document to the corresponding struct object. You should see that we have something like `xml:"serverName"` in our struct. This is a feature of structs called `struct tags` for helping with reflection. Let's see the definition of `Unmarshal` again:

	func Unmarshal(data []byte, v interface{}) error

The first argument is an XML data stream. The second argument is storage type and supports the struct, slice and string types. Go's XML package uses reflection for data mapping, so all fields in v should be exported. However, this causes a problem: how can it know which XML field corresponds to the mapped struct field? The answer is that the XML parser parses data in a certain order. The library will try to find the matching struct tag first. If a match cannot be found then it searches through the struct field names. Be aware that all tags, field names and XML elements are case sensitive, so you have to make sure that there is a one to one correspondence for the mapping to succeed.

Go's reflection mechanism allows you to use this tag information to reflect XML data to a struct object. If you want to know more about reflection in Go, please read the package documentation on struct tags and reflection.

Here are some rules when using the `xml` package to parse XML documents to structs:

- If the field type is a string or []byte with the tag `",innerxml"`, `Unmarshal` will assign raw XML data to it, like `Description` in the above example: 

	Shanghai_VPN127.0.0.1Beijing_VPN127.0.0.2

- If a field is called `XMLName` and its type is `xml.Name`, then it gets the element name, like `servers` in above example.
- If a field's tag contains the corresponding element name, then it gets the element name as well, like `servername` and `serverip` in the above example.
- If a field's tag contains `",attr"`, then it gets the corresponding element's attribute, like `version` in above example.
- If a field's tag contains something like `"a>b>c"`, it gets the value of the element c of node b of node a.
- If a field's tag contains `"="`, then it gets nothing.
- If a field's tag contains `",any"`, then it gets all child elements which do not fit the other rules.
- If the XML elements have one or more comments, all of these comments will be added to the first field that has the tag that contains `",comments"`. This field type can be a string or []byte. If this kind of field does not exist, all comments are discard.

These rules tell you how to define tags in structs. Once you understand these rules, mapping XML to structs will be as easy as the sample code above. Because tags and XML elements have a one to one correspondence, we can also use slices to represent multiple elements on the same level.

Note that all fields in structs should be exported (capitalized) in order to parse data correctly.

## Produce XML

What if we want to produce an XML document instead of parsing one. How do we do this in Go? Unsurprisingly, the `xml` package provides two functions which are `Marshal` and `MarshalIndent`, where the second function automatically indents the marshalled XML document. Their definition as follows:

	func Marshal(v interface{}) ([]byte, error)
	func MarshalIndent(v interface{}, prefix, indent string) ([]byte, error)

The first argument in both of these functions is for storing a marshalled XML data stream.

Let's look at an example to see how this works:

	package main
	
	import (
	    "encoding/xml"
	    "fmt"
	    "os"
	)
	
	type Servers struct {
	    XMLName xml.Name `xml:"servers"`
	    Version string   `xml:"version,attr"`
	    Svs     []server `xml:"server"`
	}
	
	type server struct {
	    ServerName string `xml:"serverName"`
	    ServerIP   string `xml:"serverIP"`
	}
	
	func main() {
	    v := &Servers{Version: "1"}
	    v.Svs = append(v.Svs, server{"Shanghai_VPN", "127.0.0.1"})
	    v.Svs = append(v.Svs, server{"Beijing_VPN", "127.0.0.2"})
	    output, err := xml.MarshalIndent(v, "  ", "    ")
	    if err != nil {
	        fmt.Printf("error: %v\n", err)
	    }
	    os.Stdout.Write([]byte(xml.Header))
	
	    os.Stdout.Write(output)
	}

The above example prints the following information:

	<?xml version="1.0" encoding="UTF-8"?>
	<servers version="1">
	<server>
	    <serverName>Shanghai_VPN</serverName>
	    <serverIP>127.0.0.1</serverIP>
	</server>
	<server>
	    <serverName>Beijing_VPN</serverName>
	    <serverIP>127.0.0.2</serverIP>
	</server>
	</servers>

As we've previously defined, the reason we have `os.Stdout.Write([]byte(xml.Header))` is because both `xml.MarshalIndent` and `xml.Marshal` do not output XML headers on their own, so we have to explicitly print them in order to produce XML documents correctly.

Here we can see that `Marshal` also receives a v parameter of type `interface{}`. So what are the rules when marshalling to an XML document? 

- If v is an array or slice, it prints all elements like a value.
- If v is a pointer, it prints the content that v is pointing to, printing nothing when v is nil.
- If v is a interface, it deal with the interface as well.
- If v is one of the other types, it prints the value of that type.

So how does `xml.Marshal` decide the elements' name? It follows the proceeding rules:

- If v is a struct, it defines the name in the tag of XMLName.
- The field name is XMLName and the type is xml.Name.
- Field tag in struct.
- Field name in struct.
- Type name of marshal.

Then we need to figure out how to set tags in order to produce the final XML document.

- XMLName will not be printed.
- Fields that have tags containing `"-"` will not be printed.
- If a tag contains `"name,attr"`, it uses name as the attribute name and the field value as the value, like `version` in the above example.
- If a tag contains `",attr"`, it uses the field's name as the attribute name and the field value as its value.
- If a tag contains `",chardata"`, it prints character data instead of element.
- If a tag contains `",innerxml"`, it prints the raw value.
- If a tag contains `",comment"`, it prints it as a comment without escaping, so you cannot have "--" in its value.
- If a tag contains `"omitempty"`, it omits this field if its value is zero-value, including false, 0, nil pointer or nil interface, zero length of array, slice, map and string.
- If a tag contains `"a>b>c"`, it prints three elements where a contains b and b contains c, like in the following code:

	FirstName string   `xml:"name>first"`
	LastName  string   `xml:"name>last"`
	
	<name>
	<first>Asta</first>
	<last>Xie</last>
	</name>

You may have noticed that struct tags are very useful for dealing with XML, and the same goes for the other data formats we'll be discussing in the following sections. If you still find that you have problems with working with struct tags, you should probably read more documentation about them before diving into the next section.

## Links

- [Directory](preface.md)
- Previous section: [Text files](07.0.md)
- Next section: [JSON](07.2.md)