## Effective Java 3/E

#### Item 33.  타입 안전 이종 컨테이너를 고려하라

##### 타입 안전 이종 컨테이너

- 일반적으로 매개변수화 할 수 있는 타입의 수는 제한적이기 때문에 타입의 수에 유연한 수단이 필요함

  > 예를 들어, 임의 개수의 열(Column)을 갖는 한 행을 저장하는 경우

###### 타입 안전 이종 컨테이너를 만드는 방법

1. 키를 매개변수화함
2. 컨테이너에 값을 넣거나 뺄 때, 매개변수화한 키를 함께 전달



###### 타입 안전 이종 컨테이너를 이용한 Favorites 클래스

```java
public class Favorites {
	private Map<Class<?>, Object> favorites = new HashMap<>();
	
	public <T> void putFavorites(Class<T> type, T instance) {
		favorites.put(Objects.requireNonNull(type), instance);
	}
	
	public <T> T getFavorite(Class<T> type) {
		return type.cast(favorites.get(type)); // 동적 형변환
	}
}

/*
 *     	
  	@SuppressWarnings("unchecked")
	@HotSpotIntrinsicCandidate
	public T cast(Object obj) {
		if (obj != null && !isInstance(obj))
    		throw new ClassCastException(cannotCastMsg(obj));
		return (T) obj;
	}
 */
```

- KEY: 매개변수화한 Class 객체

  > T.class의 타입은 Class<T>

- VALUE: Key의 인스턴스 객체

- value를 저장할 때에는 Object로 저장되어 타입 링크 정보가 버려지지만 getFavorites에서 원래의 타입 정보를 되살려서 반환

- Class<\T>의 cast()는 T의 인스턴스를 반환함

  > type.cast()는 type의 타입 매개변수 인스턴스를 반환하기 때문에 타입 안전?
  > _질문1._ Class 클래스가 제네릭이라는 이점을 완벽히 활용?_?
  > cast함수가 타입 안전하게 T 타입을 반환하기 때문인가??

```java
public class FavClient {

	@SuppressWarnings({ "unchecked", "rawtypes" })
	public static void main(String[] args) {
		Favorites fv = new Favorites();
        
        // 정상적인 사용
        fv.putFavorites(String.class, "JAVA"); // key: Class<String>
        fv.putFavorites(Integer.class, 8);     // key: Class<Integer>
		
		// raw 타입인 Class로 캐스팅할 경우 어떤 타입에 대한 클래스인지 알지 못하므로
		// put 할 땐 오류 발생하지 않음
		fv.putFavorites((Class)Integer.class, "Not Integer");
		
		// ClassCastException !
		int favoriteInteger = fv.getFavorite(Integer.class); 
	}
}
```

- 제약사항 

  1. raw 타입 Class를 사용할 경우 타입 안전성이 깨짐

     ```java
     	public <T> void putFavorites(Class<T> type, T instance) {
     		favorites.put(Objects.requireNonNull(type), type.cast(instance));
     	}
     ```

     type.cast를 사용해 type 확인해 안정성 확보 가능

     > java.util.Collections의 checkedSet, checkedList 등이 이 방식을 사용

  2. 실체화 불가 타입에는 사용할 수 없음
     List<\String>, List<\Integer> 등에는 사용 불가능

     > 런타임에는 모두 List.class이므로

     → 슈퍼타입토큰(?)으로 우회할 수 있음

###### 한정적 타입토큰을 활용해 허용할 타입을 제한

```java
public <T extends Annotation> T getAnnotation(Class<T> annotationType);

// java.lang.reflect.Field
    public <T extends Annotation> T getAnnotation(Class<T> annotationClass) {
        Objects.requireNonNull(annotationClass);
        return annotationClass.cast(declaredAnnotations().get(annotationClass));
    }
```

- Class<?> 타입의 객체를 getAnnotation에 넘기려면?

  - Class<? extends Annotation>으로 캐스팅할 수 있지만 비검사 경고

    → Class 클래스가 동적 형변환을 수행하는 메서드를 제공함 -> asSubClass()