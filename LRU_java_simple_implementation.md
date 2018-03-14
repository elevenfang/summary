# LRU java简单实现 #

利用LinkedHashMap按访问有序的特点简单实现了LRU缓存
```
import java.util.LinkedHashMap;

public class LRUCache<K, V> extends LinkedHashMap<K, V> {

	private int maxSize;

	public LRUCache(int maxSize) {
		super(16, 0.75F, true);
		this.maxSize = maxSize;
	}

	@Override
	protected boolean removeEldestEntry(java.util.Map.Entry<K, V> eldest) {
		return size() > maxSize;
	}
}
```

```
import org.junit.Test;

public class LRUTest {
	
	@Test
	public void test(){
		LRUCache<String, Integer> cache = new LRUCache<>(5);
		cache.put("a", 8);
		cache.put("e", 10);
		cache.put("b", 1);
		cache.put("c", 5);
		cache.put("d", 4);
		cache.put("a", 3);
		cache.get("c");
		cache.put("f", 2);
		cache.get("a");
		System.out.println(cache.toString());
	}
}
```
output:
{b=1, d=4, c=5, f=2, a=3}
```
default removeEldestEntry return false, here we override it for LRU.

