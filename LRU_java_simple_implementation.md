# LRU实现 #

利用LinkedHashMap特有的按访问顺序排序的简单实现了LRU缓存
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
