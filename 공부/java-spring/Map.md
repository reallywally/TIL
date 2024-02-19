```java
public class ReturnMain {  
    public static void main(String[] args) {  
        Map<String, String> map1 = getMapV1();  
        System.out.println("class name is " + map1.getClass().getName());  
  
        Map<String, String> map2 = getMapV2();  
        System.out.println("class name is " + map2.getClass().getName());  
    }  
  
    public static Map<String, String> getMapV1() {  
        return new HashMap<>() {{  
            put("key1", "value1");  
            put("key2", "value2");  
        }};  
    }  
  
    public static Map<String, String> getMapV2() {  
        Map<String, String> map = new HashMap<>();  
        map.put("key1", "value1");  
        map.put("key2", "value2");  
  
        return map;  
    }  
}
```

### 상황은
getMapV1 처럼 맵으로 내용 정리해서 리턴 받는 코드를 만들었는데 컴파일까지도 문제가 없다가 디비 저장할때 이슈가 발생했다.

### 원인은
![](https://i.imgur.com/P8aXiqp.png)
조금 납득하기 어려웠는데 분명히 Map<String, String>으로 받았지만 실제로는 자기 클래스에 이너 클래스로 타입이 생성되었다.
