# 配置文件


## 格式


### 定义

`Lexicon` 是 `Dong Engine` 定义的一种配置文件格式。具体而言，`Lexicon` 是一个有序的集合，由零个或多个允许重复的 `Node` 组成。

每个 `Node` 包含一个不可变的 `name` 和一个数据项。数据项可以是 `String` 类型，也可以是另一个 `Lexicon`。


### 范例

以下是一个合法的 `Lexicon`

```Lexicon
Config
{
    title = System Settings
    author = admin
    modules
    {
        logging
        {
            level = debug
            output = file
        }
        logging  // 重复节点示例
        {
            level = error
        }
        network
        {
            port = 8080
            security
            {
                ssl = true
                protocols = TLSv1.3
            }
        }
    }
    metadata = v2.1
}
```


## 调用

Java 类 `Lexicon` 提供以下核心方法操作配置文件：


### 解析方法

从字符串解析配置

```java
public static Lexicon parse(String content) throws LexiconParseException;
```

从文件解析配置

```java
public static Lexicon parse(File file) throws IOException, LexiconParseException;
```

示例
```java
Lexicon config = Lexicon.parse(new File("system.conf"));
```


### 数据获取方法

获取节点的字符串值（若不是 String 类型返回 null）

```java
public String getStringValue(String nodeName);
```

获取子配置集（若不是 Lexicon 类型返回 null）

```java
public Lexicon getChildLexicon(String nodeName);
```

获取所有同名节点（支持重复节点）

```java
public List<Node> getAllNodes(String nodeName);
```

示例

```java
String title = config.getStringValue("title"); // "System Settings"
Lexicon modules = config.getChildLexicon("modules");
List<Node> loggingNodes = modules.getAllNodes("logging"); // 两个 logging 节点
```


### 节点遍历方法

按顺序遍历所有节点

```java
public List<Node> getNodes();
```

获取单个节点（多个同名时返回第一个）

```java
public Node getNode(String nodeName);
```

示例

```java
for (Node node : config.getNodes()) {
    if (node.isChildLexicon()) {
    Lexicon child = node.asChildLexicon();
    // 处理嵌套配置
    }
}
```


### 节点操作接口

Node 类提供类型检查和方法：

```java
public class Node {
// 不可变属性
public final String name;

    // 类型检查
    public boolean isStringValue();
    public boolean isChildLexicon();

    // 值获取
    public String asStringValue();
    public Lexicon asChildLexicon();
}
```


### 完整调用示例

```java
public class ConfigLoader {
public static void main(String[] args) {
Lexicon config = Lexicon.parse(configFile);

        // 读取顶层值
        String version = config.getStringValue("metadata"); // "v2.1"
        
        // 遍历网络配置
        Lexicon network = config
            .getChildLexicon("modules")
            .getChildLexicon("network");
        
        int port = Integer.parseInt(
            network.getStringValue("port") // "8080"
        );
        
        // 处理重复节点
        config.getChildLexicon("modules")
            .getAllNodes("logging")
            .forEach(node -> {
                Lexicon logConfig = node.asChildLexicon();
                String level = logConfig.getStringValue("level"); 
                // 依次输出: debug, error
            });
    }
}
```

⚠️ 注意

- 若节点类型不匹配（如对字符串节点调用 getChildLexicon()），将返回 null

- 节点名大小写敏感

- 重复节点需通过 getAllNodes() 访问

- 修改需通过 LexiconBuilder（本文档略）