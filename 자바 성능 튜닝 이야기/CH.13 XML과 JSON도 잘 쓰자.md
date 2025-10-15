# CH.13 XML과 JSON도 잘 쓰자.
## 자바에서 사용하는 XML 파서의 종류는?
- XML의 가장 큰 장점은 누구나 데이터의 구조를 정의하고 그 정의된 구조를 공유함으로써 일관된 데이터 전송 및 처리를 할 수 있다는 점이다.
- 자바에서는 XML을 파싱하기 위해서 JAXP를 제공한다.
- JAXP는 SAX, DOM, XSLT에서 사용하는 기본 API를 제공한다.
- SAX는 순차적 방식으로 XML을 처리한다.
- DOM은 모든 XML을 읽어서 트리(Tree)를 만든 후 XML을 처리하는 방식이다.
- SAX는 각 XML의 노드를 읽는 대로 처리하기 때문에 메모리에 부담이 DOM에 비해서 많지 않다.
- DOM은 모든 XML을 메모리에 올려서 작업하기 때문에 메모리에 부담이 가게 된다.
- XSLT는 SAX, DOM, InputStream을 통해서 들어온 데이터를 원하는 형태의 화면으로 구성하는 작업을 수행한다.
- 세 가지 XML 파서 중 서버단 프로그램에서 사용하기 적합한 파서는 SAX와 DOM이다.
## SAX 파서는 어떻게 사용할까?
- SAX 파서는 순차적으로 처리하는 이벤트 기반의 모델이다.
- 모든 이벤트를 다 처리할 필요는 없지만, 원하는 데이터를 만들려면 데이터를 어떻게 처리할지 결정해서 구현한다.
- `SAXParserFactory`: 파싱을 하는 파서객체를 생성하기 위한 추상 클래스이다. 
- `SAXParser`: 여러 종류의 `parse()` 메서드를 제공하는 추상 클래스이다.
- `DefaultHandler`: 아래에 있는 `ContentHandler`, `ErrorHandler`, `DTDHandler`, `EntityResolver`를 구현한 클래스이다.
- `ContentHandler`: XML의 태그의 내용을 읽기 위한 메서드를 정의한 인터페이스다. `startDocument`, `endDocument`, `startElement`, `endElement` 메서드가 정의되어 있다.
- `ErrorHandler`: 에러를 처리하는 메서드가 정의되어 있는 인터페이스이다.
- `DTDHandler`: 기본 DTD 관련 이벤트를 식별하기 위한 인터페이스이다.
- `EntityResolve`: URI를 통한 식별을 하기 위한 인터페이스이다.
```java
public void addNode(String nodeName) {
    if (!elementMap.containsKey(nodeName)) {
        elementMap.put(nodeName, 1);
    } else {
        elementMap.put(nodeName, elementMap.get(nodeName) + 1);
    }
}

public void setNodeCountData() {
    Set<String> keySet = elementMap.keySet();
    Object[] keyArray = keySet.toArray();
    Arrays.sort(keyArray);

    for (Object tempKey : keyArray) {
        returnData.append("Element=").append(tempKey).append(" Count=")
                .append(elementMap.get(tempKey.toString())).append("<BR>");
    }
}

public String getData() {
    return returnData.toString();
}

public void print(String data) {
    returnData.append(data).append("<BR>");
}
```
- `addNode()` 메서드에서는 `HashMap`에 해당 엘리먼트가 있는지를 확인한 후, 엘리먼트 개수를 추가하는 작업을 수행한다.
```java
@State(Scope.Thread)
@BenchmarkMode({ Mode.AverageTime })
@OutputTimeUnit(TimeUnit.MICROSECONDS)
public class XMLParser {
    @GenerateMicroBenchmark
    public void withSAXParse100() throws Exception {
        ParseSAX handler = new ParseSAX();
        SAXParserFactory factory = SAXParserFactory.newInstance();
        SAXParser saxParser = factory.newSAXParser();
        saxParser.parse("dummy100.xml", handler );
    }

    @GenerateMicroBenchmark
    public void withSAXParse1000() throws Exception {
        ParseSAX handler = new ParseSAX();
        SAXParserFactory factory = SAXParserFactory.newInstance();
        SAXParser saxParser = factory.newSAXParser();
        saxParser.parse("dummy1000.xml", handler );
    }
}
```

| 대상 | 평균 응답 시간 (마이크로초) |
|---|---|
| SAX 100 | 847 |
| SAX 1,000 | 3,925 |

## DOM 파서는 어떻게 사용할까?
- DOM 파서는 XML을 트리 형태의 데이터로 먼저 만든 후, 그 데이터를 가공하는 방식을 사용한다.
- DocumentBuilderFactory: 파싱을 하는 파서객체를 생성하기 위한 추상 클래스
- DocumentBuilder: 여러 종류의 parse() 메서드를 제공하는 추상 클래스, 이 클래스의 parse() 메서드를 호출하면 파싱을 실시한다.
- Document: SAX와 다르게 파싱을 처리한 결과를 저장하는 클래스
- Node: XML과 관련된 모든 데이터의 상위 인터페이스. 단일 노드에 대한 정보를 포함하고 있다.
```java
public class ParseDOM {
    HashMap<String, Integer> elementMap=new HashMap<String, Integer>();
    private StringBuffer returnData=new StringBuffer();
    
    public void parseDOM(String XMLName) {
        DocumentBuilderFactory factory =
            DocumentBuilderFactory.newInstance();

        try {
            DocumentBuilder builder = factory.newDocumentBuilder();
            Document document = builder.parse(XMLName);
            //파싱하는 부분은 아래 소스에 나옴.
            Node rootNode=document.getChildNodes().item(0);
            addNode(rootNode.getNodeName());
            readNodes(rootNode);
            setNodeCountData();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
```java
public void readNodes(Node node) {
    NodeList childs=node.getChildNodes();
    int childCount=childs.getLength();
    for(int loop=0;loop<childCount;loop++) {
        Node tempNode=childs.item(loop);
        if(tempNode.hasChildNodes()) {
            readNodes(tempNode);//재귀 호출
        }
        String nodeName=tempNode.getNodeName();
        if(!nodeName.equals("#comment") && !nodeName.equals("#text")) {
            addNode(nodeName);
        }
    }
}

public void addNode(String nodeName) {
    if(!elementMap.containsKey(nodeName)) {
        elementMap.put(nodeName, 1);
    } else {
        elementMap.put(nodeName, elementMap.get(nodeName)+1);
    }
}

public void setNodeCountData() {
    Set keySet=elementMap.keySet();
    Object[] keyArray=keySet.toArray();
    Arrays.sort(keyArray);

    for (Object tempKey: keyArray) {
        returnData.append("Element=")
                .append(tempKey)
                .append(" Count=")
                .append(elementMap.get(tempKey.toString()))
                .append("<BR>");
    }
}

public String getData() {
    return returnData.toString();
}
```
- 노드의 `getChildNodes()` 메서드를 호출하여 자식노드의 목록을 얻는다.
- 자식노드의 개수만큼 반복하여 자식노드의 정보를 읽는다.
- 만약 그 자식 노드도 자식이 있다면 이 메서드를 호출해야 하므로, 재귀적으로 처리하였다.
```java
@GenerateMicroBenchmark
public void withDOMParse100() {
    ParseDOM pd=new ParseDOM();
    pd.parseDOM("dummy100.xml");
}

@GenerateMicroBenchmark
public void withDOMParse1000() {
    ParseDOM pd=new ParseDOM();
    pd.parseDOM("dummy1000.xml");
}
```
| 대상 | 평균 응답 시간 (마이크로초) |
|---|---|
| DOM 100 | 1,395 |
| DOM 1,000 | 7,129 |
- DOM 파서를 사용하면 보는 것과 같이 100건일 경우 1.4ms, 1,000건일 경우 7.1ms가 소요된다.

| 대상 | 응답 시간 (마이크로초) | 대상 | 응답 시간 (마이크로초) |
|---|---|---|---|
| SAX 100 | 847 | DOM 100 | 1,395 |
| SAX 1,000 | 3,925 | DOM 1,000 | 7,129 |
- 데이터의 크기가 커지면 커질수록 두 파서간의 차이가 커지는 것을 볼 수 있다.
- 처리하는 데 소요되는 대부분의 시간은 parse() 메서드에서 처리하는 CPU 시간이다.
- 대기시간은 없지만, XML을 처리하는 과정에서 CPU에 순간적으로 많은 부하가 발생한다는 것이다.
- 내부적으로 처리하는 `readNodes()` 메서드와 `addNode()` 메서드를 제거하고, 순수하게 파서에서 사용하는 메모리 사용량만 측정해 보면 다음과 같다.(데이터가 50만 건이고, 크기가 31MB인 XML 데이터를 처리한 경우이다.)

| | SAX 파서 사용시 | DOM 파서 사용시 |
|---|---|---|
| 메모리 사용량 | 56MB | 292MB |
- SAX 파서는 XML 파일크기의 거의 두배의 메모리를, DOM 파서는 거의 열배의 메모리를 사용한다.
- XML 파일의 크기가 클 때 DOM 파서를 사용한다면, OutOfMemoryError가 빈번히 일어날 확률이 매우 크다.
## XML 파서가 문제가 된 사례
## JSON과 파서들
- JSON 데이터의 구조
  - name/value 형태의 쌍으로 collection 타입
  - 값의 순서가 있는 목록 타입
- JSON도 많은 CPU와 메모리를 점유하며 응답시간도 느리다.
- 가장 많이 사용되는 JSON 파서로는 Jackson JSON과 google-gson 등이 있다.
```java
public class ParseJSON {

    public void parseStream(String json) {
        JsonFactory f = new JsonFactory();
        StringBuilder jsonResult=new StringBuilder();
        try {
            JsonParser jp = f.createJsonParser(new File(json));
            jp.nextToken();
            while (jp.nextToken() != JsonToken.END_ARRAY) {
                String fieldName = jp.getCurrentName();
                if(fieldName!=null) {
                    jp.nextToken();
                    String text = jp.getText();
                    if(fieldName.equals("productName")) {
                        jsonResult.append("Product=").append(text).
                                append("\t");
                    } else if(fieldName.equals("price")) {
                        jsonResult.append("Price=").append(text).
                                append("\n");
                    }
                }
            }
        } catch(Exception e) {
            e.printStackTrace();
        }
    }
}
```
- 이렇게 코드를 작성하면 SAX처럼 스트리밍 방식으로 데이터를 파싱하여 처리할 수 있다.
```java
@State(Scope.Thread)
@BenchmarkMode({ Mode.AverageTime })
@OutputTimeUnit(TimeUnit.MICROSECONDS)
public class JSONParser {

    @GenerateMicroBenchmark
    public void parseStream100(){
        ParseJSON pj=new ParseJSON();
        pj.parseStream("dummy100.json");
    }
    @GenerateMicroBenchmark
    public void parseStream1000(){
        ParseJSON pj=new ParseJSON();
        pj.parseStream("dummy1000.json");
    }
}
```

| 데이터 개수 | XML SAX | XML DOM | JSON |
|---|---|---|---|
| 100 | 847 | 1,395 | 245 |
| 1,000 | 3,925 | 7,129 | 1,379 |
- 이 결과만 보면 XML 파싱이 JSON 보다 매우 느리다고 생각할 수 있다.
- 그런데, 데이터를 전송하기 위해서 XML 및 JSON 데이터를 Serialize와 Deserialize할 경우도 있다.
- JSON 데이터는 Serialize와 Deserialize를 처리하는 성능이 좋지 않다.
## 데이터 전송을 빠르게 하는 라이브러리 소개
- 요즘에는 자바 객체를 전송하는 방식을 많이 사용한다.
- htps://github.com/eishay/jvm-serializers/wiki
- 이 위키에서는 여러 자바기반의 Serializer에 대한 성능비교결과를 제공한다.
- 결과목록을 보면 대표적으로 많이 사용되는 Serializer는 어떤 것이 있는지 알 수 있다.
- 이 목록의 가장 위에 있는 (변환 시간이 가장 작은) 라이브러리가 가장 좋은 것일까? 꼭 그렇지는 않다.
- 문자열을 전송하는 일이 많은지, 배열 형태의 데이터를 전송할 일이 많은지에 따라서 결과가 달라질 수가 있다.
- 일반적으로 많이 사용되는 라이브러리에는 protobuf, Thrift, avro 등이 있다.
- 서버와 서버간, 서버와 클라이언트 사이에 데이터를 주고받기 위해 어떤 자바 라이브러리를 사용할지에 대해서는 성능비교를 통해 사이트에 가장 적합한 라이브러리를 선택하는 것이 가장 좋다.