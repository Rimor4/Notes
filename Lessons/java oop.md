### 扫雷

1. switch 语句检查每一个分支是否要有**break**，否则会出现`value doesn't used`类似迷惑现象。
2. 陷入一段“算法”后，想想是不是这段算法的更高level有改进可能

```java
tb.exploreAdjacentTiles(adjacentY, adjacentX, Direction.LEFT);
tb.exploreAdjacentTiles(adjacentY, adjacentX, Direction.RIGHT);
tb.exploreAdjacentTiles(adjacentY, adjacentX, Direction.UP);
tb.exploreAdjacentTiles(adjacentY, adjacentX, Direction.DOWN);

// 如果新的tb到达边缘, 则在周围再揭开一圈数字
if (tb.isOnEdge) {
	for (int dy = -1; dy <= 1; dy++) {
		for (int dx = -1; dx <= 1; dx++) {
			int neighborX = adjacentX + dx;
			int neighborY = adjacentY + dy;

			if (neighborY >= 0 && neighborY < map.getWidth() && neighborX >= 0 && neighborX < map.getLength()) {
				adjacentTile = map.mines[neighborY][neighborX];

				// 将夹角的格子也揭开
				if (!adjacentTile.isUncovered && !adjacentTile.isMine) {
					adjacentTile.isUncovered = true;
				}
			}
		}
	}
}
```
改进：
```java
if (!adjacentTile.isUncovered && surroundingMines == 0) {
	adjacentTile.isUncovered = true;

	Trailblazer tb = new Trailblazer(map);

	// 递归周围八个格子
	tb.exploreAdjacentTiles(adjacentY, adjacentX, Direction.LEFT);
	tb.exploreAdjacentTiles(adjacentY, adjacentX, Direction.RIGHT);
	tb.exploreAdjacentTiles(adjacentY, adjacentX, Direction.UP);
	tb.exploreAdjacentTiles(adjacentY, adjacentX, Direction.DOWN);
	tb.exploreAdjacentTiles(adjacentY, adjacentX, Direction.TL);
	tb.exploreAdjacentTiles(adjacentY, adjacentX, Direction.TR);
	tb.exploreAdjacentTiles(adjacentY, adjacentX, Direction.BL);
	tb.exploreAdjacentTiles(adjacentY, adjacentX, Direction.BR);
} else if (!adjacentTile.isUncovered) {
	adjacentTile.isUncovered = true;
}
```

### 全家便利店

1. **DateTimeFormatter**格式匹配日期：
	- `yyyy/M/d`表示的是`2022/2/4` `2022/5/23` `2023/12/2`这样的无0数
	- `yyyy/MM/dd`表示的是`2022/02/04` `2022/05/23` `2023/12/02` 这样的有0数
2. 优先队列**不能用迭代器有序打印**，用`poll()`有序打印但是会**清空队列**

### CNN

1. 任意阶张量（任意维数组）递归创建：
`

```java
public class TensorTest {
    private final int max = 10;
    @Test
    // 测试int类型的四维张量
    public void testForInt() {
        Random random = new Random();
        int[] shape = {random.nextInt(max), random.nextInt(max), random.nextInt(max), random.nextInt(max)};   // 四维测试
        assertEquals(4, shape.length);
        Object tensor = generateRandomTensor(shape, 0);

        printTensor(tensor, 4, "");
    }


    // 递归地生成随机四维张量
    private Object generateRandomTensor(int[] shape, int index) {
        int size = shape[index];
        Object tensor;

        if (index == shape.length - 1) {
            tensor = new int[size];
            int[] array = new int[size];
            Random random = new Random();

            for (int i = 0; i < size; i++) {
                array[i] = random.nextInt(max);
            }

            tensor = array;     // 向上转型
        } else {
            tensor = new Object[size];
            for (int i = 0; i < size; i++) {
                ((Object[]) tensor)[i] = generateRandomTensor(shape, index + 1);
            }
        }
        return tensor;
    }

    private static void printTensor(Object tensor, int index, String prefix) {
        if (tensor == null) {
            System.out.println(prefix + "null");
            return;
        }

        // 如果是一维数组
        if (!tensor.getClass().isArray()) {
            System.out.println(prefix + tensor);
            return;
        }

        // 获取数组的长度
        int length = Array.getLength(tensor);

        // 遍历多维数组的每个元素
        for (int i = 0; i < length; i++) {
            // 生成新的前缀
            String newPrefix = prefix + "[" + i + "]";
            // 递归打印数组的每个元素
            printTensor(Array.get(tensor, i), index + 1, newPrefix);
        }
    }
}
```

<font color="#ff0000">修正</font>：使用一维数组T[]data即可，不需要递归，只需确定好index映射关系即可用getindex方法访问张量中的元素
``
2. 检查好**变量名** `inputWidth` `inputHeight`
3. 出现<font color="#ff0000">数组越界错误</font>后，应该立即先看数组**初始化大小**是否正确，而不是直接开始debug（即看索引值是否超出数组大小）


### 中世纪大冒险

1. <span style="background:#b1ffff">优化</span>： Creature**父类**可以声明**final**的字段，然后**构造器含参**，在子类中也可在不含参构造器中通过**super(parameter)** 来为final字段赋值。
2. 使用**反射**来构造类，在项目结构更改后一定要记得更改**类名字符串**!!!!! 👊

```java
private static final String[] CREATURES = { 
"models.creatures.Dwarf", "models.creatures.Elf", "models.creatures.Orc" };
```
