## Gameplay

Gameplay - 1
![picture 0](images/MindMap/IMG_20250623-215811407.jpg)
Gameplay - 2
![picture 1](images/MindMap/IMG_20250623-215814915.jpg)

## UObject

UObject - 1
![picture 2](images/MindMap/IMG_20250623-215829290.png)
UObject - 2
![picture 3](images/MindMap/IMG_20250623-215837843.png)  
UObject - 3
![picture 4](images/MindMap/IMG_20250623-215849366.png)  
UObject - 4
![picture 5](images/MindMap/IMG_20250623-215853165.jpg)  
UObject - 5
![picture 6](images/MindMap/IMG_20250623-224508371.jpg)  
UObject - 6
![picture 7](images/MindMap/IMG_20250628-203615661.png)

1. 大部分是不言自明的，从左到右是类型信息的收集和消费过程。从上到下是依据代码的执行顺序。
2. 红色箭头代表数据的产生添加，蓝色箭头代表数据的消费使用。这二者一起表达了类型信息的数据流向。
3. 浅蓝色箭头和矩形，代表内存中 UClass \* 以及类型对象的创建和构造
4. 信息收集里黄色的 3 个矩形，代表它们的数据会一直在内存中，用来做查找用，不会被清空.

UObject - 7
![picture 9](images/MindMap/IMG_20250629-200854506.png)  
UObject - 8
![picture 10](images/MindMap/IMG_20250629-200907124.png)


## 智能指针

### 共享指针
![picture 11](images/MindMap/IMG_20250705-093558624.png)
