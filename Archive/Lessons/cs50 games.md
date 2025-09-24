### 2. breakout
1. 不能update / render**空对象**
2. 不能用table.insert（）返回值来获得最新插入的值的索引【用<font color="#ff0000">#table</font>】
3. Q: 每次清除完砖块（含lockbrick）后报错`src/states/PlayState.lua:302: attempt to call method 'update' (a nil value)`  为什么我每次
	```lua
	for k, brick in pairs(self.bricks) do
        if k ~= keyIndex then 
            brick:update(dt)
        end
    end
```

### 3. match-3
1. 避免在迭代table（matches）时**修改subtable**（match）
2. 遍历table时注意table是否**多维**
3. **找准**quads的实际**坐标**
4. love.mousepressed是*全局函数*，而不是特定对象的方法，不可调用self
5. x，y循环中不要忘记每遍历一列（行）是否**重置y（x）**
```lua
if handleMouseClicked then
            local tX = self.board.x
            local tY = self.board.y

            for y = 1, 8 do
                for x = 1, 8 do
                    if gameX > tX and gameX < tX + 32 and gameY > tY and gameY < tY + 32 then
                        self.boardHighlightX = x - 1
                        self.boardHighlightY = y - 1
                        gSounds['select']:play()
                        isGridClicked = true
                    end
                    tX = tX + 32
                end
                tY = tY + 32
                tX = self.board.x
            end
        end
```

### 4. mario
1. 不同文件间**函数传参**对象要对应
2. 对table中的对象操作时，注意在其他地方函数中调用的是**局部变量**，而不是实际table中的元素
	```lua
	 onCollide = function(player, object)
                            if not object.hit then
                                -- when player collide with lock using key then removed
                                if player.haveKey then 
                                    gSounds['unlock']:play()
                                    player.haveKey = false
                                    object = nil
                                end
                            end
                        end
```
3. 搞清楚方法中的某个函数（onCollide）执行位置是在该方法内还是其他方法
4. lua函数可嵌套（函数是一等公民），**闭包**

### 5.zelda
1. remove一定要**从后往前遍历table**，在同一个循环中从前往后remove会使table<font color="#c00000">后续元素索引</font>改变
2. 死去的敌人<span style="background:#fff88f">不一定</span>要从table  remove，可能会有某些需要时间发生的函数先调用了包括死去的敌人的所有对象，然后发生remove会导致该函数索引空值
3.<font color="#ff0000"> ！！！</font>创建/加入GameObject，注意传参时用  **{}还是()**   
    以及调用 类 方法/属性函数 应该分别用 object **:** function() / object **.** function()
```lua
local gem = GameObject {
	texture = 'gems',
	x = (x - 1) * TILE_SIZE,
	y = (blockHeight - 1) * TILE_SIZE - 4,
	width = 16,
	height = 16,
	frame = math.random(#GEMS),
	collidable = true,
	consumable = true,
	solid = false,

	-- gem has its own function to add to the player's score
	onConsume = function(player, object)
		gSounds['pickup']:play()
		player.score = player.score + 100
	end
}
```
```lua
table.insert(self.objects, 
	GameObject (
		GAME_OBJECT_DEFS['heart'],
		entity.x,
		entity.y
	)
) 
```
4. 尽量避免用timer.after来remove ，Timer异步执行，在计时器执行时，`i` 的值可能已经发生了变化，导致删除了不正确的 `self.objects` 中的元素
	可以用**标记**来避免多次执行
```lua
if object.state == 'broken' then
	-- delay removing
	Timer.after(0.1, function()
		table.remove(self.objects, i)
		print('timer after!!!!!!!!')
	end)
end
```

### 10. portal
1. Text 脚本中引用trigger而不是trigger引用Text实例中脚本组件重点变量。

## 11. Final PJ
1. <font color="#ff0000">优化</font>：子类 Player1，Player2  override 父类 Player 方法
