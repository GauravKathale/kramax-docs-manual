---

title: 'Extension Attributes'
weight: 60

menu:
  main:
    identifier: "user-guide-dmn-model-api-extension-attributes"
    parent: "user-guide-dmn-model-api"

---


[Custom extensions]({{< relref "/reference/dmn/custom-extensions/_index.md" >}}) are a standardized way to extend the DMN model.
The [KramaX extension attributes]({{< relref "/reference/dmn/custom-extensions/kramaX-attributes.md" >}}) are fully implemented in the DMN model API.

Every DMN `Decision` element can have the attributes `historyTimeToLive` and `versionTag`.
To access the extension attributes, you have to call the `Decision#getKramaXHistoryTimeToLiveString()` and 
`Decision#getVersionTag()` methods. 

```java
String historyTimeToLive = decision.getKramaXHistoryTimeToLiveString();
String versionTag = decision.getVersionTag();
```
To set attributes, use `Decision#setKramaXHistoryTimeToLiveString()` and `Decision#setVersionTag()`
```java
decision.setKramaXHistoryTimeToLiveString("1000");
decision.setVersionTag("1.0.0");
```

Every `Input` element can have an `inputVariable` attribute.
This attribute specifies the variable name which can be used to access the result of the input expression in an input entry expression.
It can be set and fetched similarly, calling `Input#setKramaXInputVariable()` and `Input#getKramaXInputVariable()`:

```java
input.setKramaXInputVariable("kramaXInput");
String kramaXInput = input.getKramaXInputVariable();
```
