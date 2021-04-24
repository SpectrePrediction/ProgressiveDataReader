# ProgressiveDataReader
 Universal data reader, which is a partition of my other projects, can meet the data reading under low memory</br>
 通用数据读取器，他是我其他项目的分割，能够满足低内存下的数据读取</br>

## ⛳依赖
几乎没有</br>
### 其中</br>
numpy 以及 opencv是关于数据读取和处理的部分，你可以自定义自己需要读取方式
***
## 🔨使用方法
全部都在[DateReader.py](./DateReader.py)中</br>

### 其最主要的是：</br>
```python
class DataReader (...)

dr = DataReader(r"test.txt", [
        img_padding,  # img_padding没使用修饰，所以传入函数名，如果使用修饰此处使用img_padding()
        img_resize((512, 512)),
        img_BGR2YUV(),
    ],
    read_data_func=one_hot_label_read_func(3),  # 3类类别
    read_data_cache=20,  # 读取数据的缓冲大小
    batch_size=10,  # 读取数据得到的批大小 必填
    is_completion=True,  # 是否填充
    using_thread_num=5,  # 使用线程数
    is_show_progress=True  # 是否可视化读取进程
    # read_txt_func  # 读取文本的函数 使用默认
    # is_shuffle  # 是否乱序 使用默认
)

for epoch, image, label in dr:
    image = np.array(image).astype("float32") / 255.0
    label = np.array(label)

    print(image.shape, label.shape, epoch)

```

### 可以注意到几个要点
- DataReader类通过读取txt中存放的数据地址再进行数据加载
- - 这意味着你可以定义自己的文本文件读取方法以及数据读取方法
- - 例如读取多个图片而非读取图片加标签（默认读取方法就是如此）
- DataReader类有个专门的列表用于存放给数据预处理的函数，或者你可以将这部分放在数据读取的定义中
- - 你可以自定义这些预处理函数，前提是你需要遵守一定的约定
- DataReader类允许设定读取缓冲read_data_cache，并且在使用时渐进式加载下一批的数据
- - 这可以在内存容量无法一次性全部加载数据集时起到作用
- DataReader类允许设置线程数、填充、可视化、乱序等基本功能
***
- 事实上使用的one_hot_label_read_func函数是被修饰的预先定义好的数据读取方法
```python
@data_read_function(DEBUG)
def one_hot_label_read_func(img_path, label_class):
    if '.' in img_path:
        return cv2.imread(img_path)

    return one_hot(img_path, label_class)
```
- 这非常的简陋，但这是一个好例子，你可以在其他地方发现许多函数调用例如：
```python
    img_resize((512, 512))
    img_BGR2YUV()
 ```
- - 以及同一地方却传入函数名的：
```python
    img_padding
 ```
### 他们之间的区别是什么呢？使用data_read_function修饰！</br>
为使用data_read_function修饰：
- [ ] 传入函数名用以被调用
- [ ] 无法定义额外参数或者参数无法传入
- [ ] 一些常量只能写死在函数内非常不便</br>
为此而实现的data_read_function修饰，能够实现：</br>
- [x] 修饰后的函数无需更改写法，允许类似闭包的效果
- [x] 修饰后的函数可以提前函数调用传入自定义的参数
- [x] 修饰后的函数使用起来与正常函数没有区别
- [x] data_read_function允许传入参数来控制是否启动
```python
看看上面我们被修饰的函数one_hot_label_read_func

他的定义形式是one_hot_label_read_func（img_path, label_class）
正常的调用方式是one_hot_label_read_func("test.jpg", 3)
被修饰后的调用方式one_hot_label_read_func(3)("test.jpg")
他们是等效果的

data_read_function对多余参数的数量没有要求（img_path, label_class1， label_class2, ...）
one_hot_label_read_func(img_path, label_class1， label_class2, ...)
修饰后
one_hot_label_read_func(label_class1， label_class2, ...)（img_path）
```
### 除去这两个比较重要的部分，剩下的锦上添花
- log_print 带颜色的打印效果
- CacheList 一个带有队列类似效果的列表（到达数量后最先进入的被删除）
- one_hot、one_hot_by_container 一热编码的工具
- label_read_func、one_hot_label_read_func 预定义的数据读取方法
- img_random_resize_from_dict、img_float_normalization等一些列预处理方法

### 此读取者为项目中开发的附属品，所以有些地方还是有着许多不足
将其开源也正是为了完善它并且方便自己下载🌹
