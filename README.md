# Overview

* == general-purpose data-binding functionality + tree-model for [Jackson Data Processor](../../../jackson)
  * built | [Streaming API](../../../jackson-core) (stream parser/generator) package
  * configured -- based on -- [Jackson Annotations](../../../jackson-annotations)
  * 👁️NOT JSON specific 👁️
    * ALTHOUGH some naming does contain 'JSON'
      * REASON: 🧠historical reasons 🧠
  * -- depends on --
    * `jackson-core`
    * `jackson-annotations`
* Jackson
  * use cases
    * originally, JSON data-binding
    * read content / encoded in OTHER data formats 
      * Reason: 🧠-- thanks to -- `jackson-databinding` 🧠
      * requirements
        * parser & generator implementations exist | those data formats

## Status

| Type | Status |
| ---- | ------ |
| Build (CI) | [![Build (github)](https://github.com/FasterXML/jackson-databind/actions/workflows/main.yml/badge.svg)](https://github.com/FasterXML/jackson-databind/actions/workflows/main.yml) |
| Artifact | [![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.fasterxml.jackson.core/jackson-databind/badge.svg)](https://maven-badges.herokuapp.com/maven-central/com.fasterxml.jackson.core/jackson-databind) |
| OSS Sponsorship | [![Tidelift](https://tidelift.com/badges/package/maven/com.fasterxml.jackson.core:jackson-databind)](https://tidelift.com/subscription/pkg/maven-com-fasterxml-jackson-core-jackson-databind?utm_source=maven-com-fasterxml-jackson-core-jackson-databind&utm_medium=referral&utm_campaign=readme) |
| Javadocs | [![Javadoc](https://javadoc.io/badge/com.fasterxml.jackson.core/jackson-databind.svg)](http://www.javadoc.io/doc/com.fasterxml.jackson.core/jackson-databind) |
| Code coverage (2.18) | [![codecov.io](https://codecov.io/github/FasterXML/jackson-databind/coverage.svg?branch=2.18)](https://codecov.io/github/FasterXML/jackson-databind?branch=2.18) |
| OpenSSF Score | [![OpenSSF  Scorecard](https://api.securityscorecards.dev/projects/github.com/FasterXML/jackson-databind/badge)](https://securityscorecards.dev/viewer/?uri=github.com/FasterXML/jackson-databind) |

# Get it!

## Maven

* add it | "pom.xml"

    ```xml
    <properties>
      ...
      <!-- Use the latest version whenever possible. -->
      <jackson.version>2.17.1</jackson.version>
      ...
    </properties>
    
    <dependencies>
      ...
      <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>${jackson.version}</version>
      </dependency>
      ...
    </dependencies>
    ```

  * dependant `jackson-core` & `jackson-annotations` are automatically included
    * if you want to ensure compatible versions -> use [jackson-bom](../../../jackson-bom)

* jars
  * you can download
    * | Maven repository or
    * links on [Wiki](../../wiki)
  * == functional OSGi bundle / import/export declarations
    * -> can be used | OSGi container
  * from Jackson v2.10+
    * `module-info.class` definitions are included -> jar == proper Java module (JPMS)

## Non-Maven dependency resolution

* download
  * `jackson-databind`
  * dependant 
    * `jackson-core` jar
    * `jackson-annotations` jar

-----

## Compatibility

* TODO:
### JDK

Jackson-databind package baseline JDK requirements are as follows:

* Versions 2.0 - 2.7 require JDK 6
* Versions 2.8 - 2.12 require JDK 7 to run (but 2.11 - 2.12 require JDK 8 to build)
* Versions 2.13 and above require JDK 8

### Android

List is incomplete due to compatibility checker addition being done for Jackson 2.13.

* 2.13: Android SDK 24+
* 2.14: Android SDK 26+
* 2.15: Android SDK 26+
* 2.16: Android SDK 26+
* 2.17: Android SDK 26+
* 2.18: (planned) Android SDK 26+

for information on Android SDK versions to Android Release names see [https://en.wikipedia.org/wiki/Android_version_history]

-----

# Use It!

* [Wiki](../../wiki)

## 1 minute tutorial: POJOs <-> JSON


```java
    import com.fasterxml.jackson.databind.ObjectMapper;

    // Note: can use getters/setters as well; here we just use public fields directly:
    public class MyValue {
      public String name;
      public int age;
      // NOTE: if using getters/setters, can keep fields `protected` or `private`
    }
  
    ObjectMapper mapper = new ObjectMapper(); // create once, reuse. Default one, it's valid so far  
  
    // 1. JSON -> POJO
    // 1.1
    MyValue value = mapper.readValue(new File("data.json"), MyValue.class);
    // 1.2
    value = mapper.readValue(new URL("http://some.com/api/entry.json"), MyValue.class);
    // 1.3
    value = mapper.readValue("{\"name\":\"Bob\", \"age\":13}", MyValue.class);
    
    // 2. POJO -> JSON
    // 2.1
    mapper.writeValue(new File("result.json"), myResultObject);
    // 2.2
    byte[] jsonBytes = mapper.writeValueAsBytes(myResultObject);
    // 2.3
    String jsonString = mapper.writeValueAsString(myResultObject);    
```

## 3 minute tutorial: Generic collections, Tree Model

### Generic collections

* JSON <-> JDK `List`s, `Map`s

```java
// 1. JSON -> JDK
// 1.1 Map
Map<String, Integer> scoreByName = mapper.readValue(jsonSource, Map.class);
// 1.2 List
List<String> names = mapper.readValue(jsonSource, List.class); 

// 2. JDK -> JSON 
mapper.writeValue(new File("names.json"), names);
```

* 👁️if your collection has POJO values (_Example:_ `Map`) -> you need to indicate actual type 👁️
  * NOT applicable | `List`
    * Reason: 🧠 you use POJO properties | `List`, != POJO values 🧠

```java
// 1. JSON -> Map
Map<String, ResultValue> results = mapper.readValue(jsonSource,
   new TypeReference<Map<String, ResultValue>>() { } );     // specify the actual type
// why extra work? Java Type Erasure will prevent type detection otherwise
```

### Tree Model

* Jackson's [Tree model](https://github.com/FasterXML/jackson-databind/wiki/JacksonTreeModel)
* use cases
  * object traversal
  * structure highly dynamic / -- NOT map nicely to -- Java classes
* JSON <-> Jackson Tree Model
  * Jackson Tree Model entities
    * `JsonNode`
      * generic one
    * `ObjectNode`
      * uses
        * known in advanced / it's an `Object`
    * `ArrayNode`
      * uses
        * known in advanced / it's an `Array`

```java
// 1. JSON -> Jackson Tree Model
JsonNode root = mapper.readTree("{ \"name\": \"Joe\", \"age\": 13 }");  // JsonNode
String name = root.get("name").asText();
int age = root.get("age").asInt();

// Jackson Tree Model can be modified
root.withObject("/other").put("type", "student");

// 2. Jackson Tree Model -> JSON 
String json = mapper.writeValueAsString(root); // prints below
System.out.println(json);
/*
output of the 'json' String
{
  "name" : "Bob",
  "age" : 13,
  "other" : {
    "type" : "student"
  }
} 
*/
```

### Tree Model + Structured data binding

* Structured Data == ALL POJO
* use cases
  * part of the document is known

```java
// 1. JSON -> Jackson Tree Model
JsonNode root = mapper.readTree(complexJson);           
Person p = mapper.treeToValue(root.get("person"), Person.class); // Jackson Tree Model / contains POJO
Map<String, Object> dynamicmetadata = mapper.treeToValue(root.get("dynamicmetadata"), Map.class); // Jackson Tree Model / contains Map

// Jackson v2.16+
List<Person> friends = mapper.treeToValue(root.get("friends"), new TypeReference<List<Person>>() { }); // Jackson Tree Model / contains List 
                                                                                                     

// 1.1 ways to find a child node | Jackson Tree model
// 1.1.1 consecutive .get()
int singledeep = root.get("deep").get("large").get("hiearchy").get("important").intValue();
// 1.1.2 .at("pathToFindIt")
int singledeeppath = root.at("/deep/large/hiearchy/important").intValue(); 
// 1.1.3 .findValue("childNodeName")
int singledeeppathunique = root.findValue("important").intValue(); 


// 2. POJO or individually -> Jackson Tree Model 
ObjectNode root = mapper.createObjectNode();
// 2.1 POJO
root.putPOJO("person", new Person("Joe")); 
// 2.2 Generics
root.putPOJO("friends", List.of(new Person("Jane"), new Person("Jack"))); 
// 2.3 Collection
Map<String, Object> dynamicmetadata = Map.of("Some", "Metadata");
root.putPOJO("dynamicmetadata", dynamicmetadata); 
root.putPOJO("dynamicmetadata", mapper.valueToTree(dynamicmetadata));
root.set("dynamicmetadata", mapper.valueToTree(dynamicmetadata));
// 2.3 as you go
root.withObject("deep").withObject("large").withObject("hiearchy").put("important", 42);
// 2.4. as you go BUT WITHOUT trying json path -- Jackson v2.16+ --
root.withObjectProperty("deep").withObjectProperty("large").withObjectProperty("hiearchy").put("important", 42); 
// 2.5 -- based on -- json path
root.withObject("/deep/large/hiearchy").put("important", 42);  
mapper.writeValueAsString(root);
```

## 5 minute tutorial: Streaming parser, generator

* incremental / "streaming" model
  * another canonical processing model
  * allows
    * controlling over parsing
  * uses
    * by
      * data-binding
      * Tree Model
* `JsonParser`
  * allows
    * reading file back

```java
ObjectMapper mapper = ...;

File jsonFile = new File("test.json");      // JSON output 

// 1. JsonGenerator
// Jackson v2.11+ 
JsonGenerator g = mapper.createGenerator(jsonFile, JsonEncoding.UTF8);
// Jackson v2.11-
// mapper.getFactory().createGenerator(...)

// 2. populate JsonGenerator
// write JSON: { "message" : "Hello world!" }
g.writeStartObject();
g.writeStringField("message", "Hello world!");
g.writeEndObject();
g.close();

// 3. JsonParser
try (JsonParser p = mapper.createParser(jsonFile)) {
  JsonToken t = p.nextToken(); // Should be JsonToken.START_OBJECT
  t = p.nextToken(); // JsonToken.FIELD_NAME
  if ((t != JsonToken.FIELD_NAME) || !"message".equals(p.getCurrentName())) {
   // handle error
  }
  t = p.nextToken();
  if (t != JsonToken.VALUE_STRING) {
   // similarly
  }
  String msg = p.getText();
  System.out.printf("My message to you is: %s!\n", msg);
}
```

## 10 minute tutorial: configuration

* TODO:
There are two entry-level configuration mechanisms you are likely to use:
[Features](https://github.com/FasterXML/jackson-databind/wiki/JacksonFeatures) and [Annotations](https://github.com/FasterXML/jackson-annotations).

### Commonly used Features

Here are examples of configuration features that you are most likely to need to know about.

Let's start with higher-level data-binding configuration.

```java
// SerializationFeature for changing how JSON is written

// to enable standard indentation ("pretty-printing"):
mapper.enable(SerializationFeature.INDENT_OUTPUT);
// to allow serialization of "empty" POJOs (no properties to serialize)
// (without this setting, an exception is thrown in those cases)
mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
// to write java.util.Date, Calendar as number (timestamp):
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);

// DeserializationFeature for changing how JSON is read as POJOs:

// to prevent exception when encountering unknown property:
mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
// to allow coercion of JSON empty String ("") to null Object value:
mapper.enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);
```

In addition, you may need to change some of low-level JSON parsing, generation details:

```java
// JsonParser.Feature for configuring parsing settings:

// to allow C/C++ style comments in JSON (non-standard, disabled by default)
// (note: with Jackson 2.5, there is also `mapper.enable(feature)` / `mapper.disable(feature)`)
mapper.configure(JsonParser.Feature.ALLOW_COMMENTS, true);
// to allow (non-standard) unquoted field names in JSON:
mapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);
// to allow use of apostrophes (single quotes), non standard
mapper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);

// JsonGenerator.Feature for configuring low-level JSON generation:

// to force escaping of non-ASCII characters:
mapper.configure(JsonGenerator.Feature.ESCAPE_NON_ASCII, true);
```

Full set of features are explained on [Jackson Features](https://github.com/FasterXML/jackson-databind/wiki/JacksonFeatures) page.

### Annotations: changing property names

The simplest annotation-based approach is to use `@JsonProperty` annotation like so:

```java
public class MyBean {
   private String _name;

   // without annotation, we'd get "theName", but we want "name":
   @JsonProperty("name")
   public String getTheName() { return _name; }

   // note: it is enough to add annotation on just getter OR setter;
   // so we can omit it here
   public void setTheName(String n) { _name = n; }
}
```

There are other mechanisms to use for systematic naming changes, including use of "Naming Strategy" (via `@JsonNaming` annotation).

You can use [Mix-in Annotations](https://github.com/FasterXML/jackson-docs/wiki/JacksonMixinAnnotations) to associate any and all Jackson-provided annotations.

### Annotations: Ignoring properties

There are two main annotations that can be used to ignore properties: `@JsonIgnore` for individual properties; and `@JsonIgnoreProperties` for per-class definition

```java
// means that if we see "foo" or "bar" in JSON, they will be quietly skipped
// regardless of whether POJO has such properties
@JsonIgnoreProperties({ "foo", "bar" })
public class MyBean
{
   // will not be written as JSON; nor assigned from JSON:
   @JsonIgnore
   public String internal;

   // no annotation, public field is read/written normally
   public String external;

   @JsonIgnore
   public void setCode(int c) { _code = c; }

   // note: will also be ignored because setter has annotation!
   public int getCode() { return _code; }
}
```

As with renaming, note that annotations are "shared" between matching fields, getters and setters: if only one has `@JsonIgnore`, it affects others.
But it is also possible to use "split" annotations, to for example:

```java
public class ReadButDontWriteProps {
   private String _name;
   @JsonProperty public void setName(String n) { _name = n; }
   @JsonIgnore public String getName() { return _name; }
}
```

in this case, no "name" property would be written out (since 'getter' is ignored); but if "name" property was found from JSON, it would be assigned to POJO property!

For a more complete explanation of all possible ways of ignoring properties when writing out JSON, check ["Filtering properties"](http://www.cowtowncoder.com/blog/archives/2011/02/entry_443.html) article.

### Annotations: using custom constructor

Unlike many other data-binding packages, Jackson does not require you to define "default constructor" (constructor that does not take arguments).
While it will use one if nothing else is available, you can easily define that an argument-taking constructor is used:

```java
public class CtorBean
{
  public final String name;
  public final int age;

  @JsonCreator // constructor can be public, private, whatever
  private CtorBean(@JsonProperty("name") String name,
    @JsonProperty("age") int age)
  {
      this.name = name;
      this.age = age;
  }
}
```

Constructors are especially useful in supporting use of
[Immutable objects](http://www.cowtowncoder.com/blog/archives/2010/08/entry_409.html).

Alternatively, you can also define "factory methods":

```java
public class FactoryBean
{
    // fields etc omitted for brevity

    @JsonCreator
    public static FactoryBean create(@JsonProperty("name") String name) {
      // construct and return an instance
    }
}
```

Note that use of a "creator method" (`@JsonCreator` with `@JsonProperty` annotated arguments) does not preclude use of setters: you
can mix and match properties from constructor/factory method with ones that
are set via setters or directly using fields.

## Tutorial: fancier stuff, conversions

One useful (but not very widely known) feature of Jackson is its ability
to do arbitrary POJO-to-POJO conversions. Conceptually you can think of conversions as sequence of 2 steps: first, writing a POJO as JSON, and second, binding that JSON into another kind of POJO. Implementation just skips actual generation of JSON, and uses more efficient intermediate representation.

Conversions work between any compatible types, and invocation is as simple as:

```java
ResultType result = mapper.convertValue(sourceObject, ResultType.class);
```

and as long as source and result types are compatible -- that is, if to-JSON, from-JSON sequence would succeed -- things will "just work".
But here are a couple of potentially useful use cases:

```java
// Convert from List<Integer> to int[]
List<Integer> sourceList = ...;
int[] ints = mapper.convertValue(sourceList, int[].class);
// Convert a POJO into Map!
Map<String,Object> propertyMap = mapper.convertValue(pojoValue, Map.class);
// ... and back
PojoType pojo = mapper.convertValue(propertyMap, PojoType.class);
// decode Base64! (default byte[] representation is base64-encoded String)
String base64 = "TWFuIGlzIGRpc3Rpbmd1aXNoZWQsIG5vdCBvbmx5IGJ5IGhpcyByZWFzb24sIGJ1dCBieSB0aGlz";
byte[] binary = mapper.convertValue(base64, byte[].class);
```

Basically, Jackson can work as a replacement for many Apache Commons components, for tasks like base64 encoding/decoding, and handling of "dyna beans" (Maps to/from POJOs).

## Tutorial: Builder design pattern + Jackson
The Builder design pattern is a creational design pattern and can be used to create complex objects step by step.
If we have an object that needs multiple checks on other dependencies, In such cases, it is preferred to use builder design pattern.

Let's consider the person structure, which has some optional fields

```java
public class Person {
    private final String name;
    private final Integer age;
 
    // getters
}
```

Let’s see how we can employ its power in deserialization. First of all, let’s declare a private all-arguments constructor, and a Builder class.
```java
private Person(String name, Integer age) {
    this.name = name;
    this.age = age;
}
 
static class Builder {
    String name;
    Integer age;
    
    Builder withName(String name) {
        this.name = name;
        return this;
    }
    
    Builder withAge(Integer age) {
        this.age = age;
        return this;
    }
    
    public Person build() {
        return new Person(name, age);
    } 
}
```
First of all, we need to mark our class with `@JsonDeserialize` annotation, passing a builder parameter with a fully qualified domain name of a builder class.
After that, we need to annotate the builder class itself as `@JsonPOJOBuilder`.

```java
@JsonDeserialize(builder = Person.Builder.class)
public class Person {
    //...
    
    @JsonPOJOBuilder
    static class Builder {
        //...
    }
}
```

A simple unit test will be:

```java
String json = "{\"name\":\"Hassan\",\"age\":23}";
Person person = new ObjectMapper().readValue(json, Person.class);
 
assertEquals("Hassan", person.getName());
assertEquals(23, person.getAge().intValue());
```

If your builder pattern implementation uses other prefixes for methods or uses other names than build() for the builder method Jackson also provide a handy way for you.

For example, if you have a builder class that uses the "set" prefix for its methods and use the create() method instead of build() for building the whole class, you have to annotate your class like:
```java
@JsonPOJOBuilder(buildMethodName = "create", withPrefix = "set")
static class Builder {
    String name;
    Integer age;
    
    Builder setName(String name) {
        this.name = name;
        return this;
    }
    
    Builder setAge(Integer age) {
        this.age = age;
        return this;
    }
    
    public Person create() {
        return new Person(name, age);
    } 
}
```

To deserialize JSON fields under a different name than their object counterparts,
the @JsonProperty annotation can be used within the builder on the appropriate fields.

```java
@JsonPOJOBuilder(buildMethodName = "create", withPrefix = "set")
static class Builder {
    @JsonProperty("known_as")
    String name;
    Integer age;
    //...
}
```
This will deserialize the JSON property `known_as` into the builder field `name`. If a mapping like this is not provided (and further annotations aren't supplied to handle this), an `Unrecognized field "known_as"` exception will be thrown during deserialization if the field is provided anyways.

If you wish to refer to properties with more than one alias for deserialization, the `@JsonAlias` annotation can be used.

```java
@JsonPOJOBuilder(buildMethodName = "create", withPrefix = "set")
static class Builder {
    @JsonProperty("known_as")
    @JsonAlias({"identifier", "first_name"})
    String name;
    Integer age;
    //...
}
```
This will deserialize JSON fields with `known_as`, as well as `identifer` and `first_name` into `name`. Rather than an array of entries, a single alias can be used by specifying a string as such `JsonAlias("identifier")`.  
Note: to use the `@JsonAlias` annotation, a `@JsonProperty` annotation must also be used.




Overall, Jackson library is very powerful in deserializing objects using builder pattern.

# Contribute!

We would love to get your contribution, whether it's in form of bug reports, Requests for Enhancement (RFE), documentation, or code patches.

See [CONTRIBUTING](https://github.com/FasterXML/jackson/blob/master/CONTRIBUTING.md) for details on things like:

* Community, ways to interact (mailing lists, gitter)
* Issue tracking ([GitHub Issues](https://github.com/FasterXML/jackson-databind/issues))
* Paperwork: CLA (just once before the first merged contribution)

## Limitation on Dependencies by Core Components

One additional limitation exists for so-called core components (streaming api, jackson-annotations and jackson-databind): no additional dependencies are allowed beyond:

* Core components may rely on any methods included in the supported JDK
    * Minimum Java version is Java 7 for Jackson 2.7 - 2.12 of `jackson-databind` and most non-core components
    * Minimum Java version is Java 8 for Jackson 2.13 and later
* Jackson-databind (this package) depends on the other two (annotations, streaming).

This means that anything that has to rely on additional APIs or libraries needs to be built as an extension,
usually a Jackson module.

## Branches

`master` branch is for developing the next major Jackson version -- 3.0 -- but there
are active maintenance branches in which much of development happens:

* `2.16` is the branch for "next" minor version to release (as of July 2023)
* `2.15` is the current stable minor 2.x version
* `2.14` is for selected backported fixes

Older branches are usually not maintained after being declared as closed
on [Jackson Releases](https://github.com/FasterXML/jackson/wiki/Jackson-Releases) page,
but exist just in case a rare emergency patch is needed.
All released versions have matching git tags (e.g. `jackson-dataformats-binary-2.12.3`).

-----

## Differences from Jackson 1.x

Project contains versions 2.0 and above: source code for last (1.x) release, 1.9, is available at
[Jackson-1](../../../jackson-1) repo.

Main differences compared to 1.x "mapper" jar are:

* Maven build instead of Ant
* Java package is now `com.fasterxml.jackson.databind` (instead of `org.codehaus.jackson.map`)

-----

## Support

### Community support

Jackson components are supported by the Jackson community through mailing lists, Gitter forum, Github issues. See [Participation, Contributing](../../../jackson#participation-contributing) for full details.


### Enterprise support

Available as part of the [Tidelift](https://tidelift.com/subscription/pkg/maven-com-fasterxml-jackson-core-jackson-databind) Subscription.

The maintainers of `jackson-databind` and thousands of other packages are working with Tidelift to deliver commercial support and maintenance for the open source dependencies you use to build your applications. Save time, reduce risk, and improve code health, while paying the maintainers of the exact dependencies you use. [Learn more.](https://tidelift.com/subscription/pkg/maven-com-fasterxml-jackson-core-jackson-databind?utm_source=maven-com-fasterxml-jackson-core-jackson-databind&utm_medium=referral&utm_campaign=enterprise&utm_term=repo)

-----

## Further reading

* [Overall Jackson Docs](../../../jackson-docs)
* [Project wiki page](https://github.com/FasterXML/jackson-databind/wiki)

Related:

* [Core annotations](https://github.com/FasterXML/jackson-annotations) package defines annotations commonly used for configuring databinding details
* [Core parser/generator](https://github.com/FasterXML/jackson-core) package defines low-level incremental/streaming parsers, generators
* [Jackson Project Home](../../../jackson) has links to all modules
* [Jackson Docs](../../../jackson-docs) is project's documentation hub

