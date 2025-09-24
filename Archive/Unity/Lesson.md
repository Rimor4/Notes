# Eight Diagram Card

1. 发现莫名奇妙的Null Reference, 结果是把**脚本挂载到了错误的对象上**，并缺少赋值：
   记得要根据LogError定位准问题发生的位置。

2. **TextMeshPro**的`textinfo`属性需要在`text`赋值后及时用`ForceMeshUpdate(true)`更新，且要加参数`true`确保即使GameObject为Active的false也要执行**强制更新** ??? ***有的不论怎么`textinfo`都为空***

3. Unity `Destroy()`函数真正卸载物体将在下一帧发生，有时候后面的逻辑需要用到`yield return null`.

4. 把某prefab添加到content父物体下报错"Trying to add CardFrame (UnityEngine.UI.Image) for graphic rebuild while we are already inside a graphic rebuild loop."
   或者报错"Invalid AABB aabb UnityEngine.StackTraceUtility:ExtractStackTrace ()", 可能是因为RectTransform 没有合理的大小（例如宽度或高度为 0 或 **NaN**）。

5. `Scroll Rect`设置的Content动态添加子物体后，需要调用`LayoutRebuilder.ForceRebuildLayoutImmediate(content);`来刷新布局。

6. 对象池达到 `maxSize` 后，回收对象会被销毁，而不是保留在池中。重新获取对象时，如果对象已经被销毁，会导致 `MissingReferenceException` 错误。
