#### 扁平化

数组的扁平化，就是将一个嵌套多层的数组 array (嵌套可以是任何层数)转换为只有一层的数组。

举个例子，假设有个名为 flatten 的函数可以做到数组扁平化，效果就会如下：

```javascript
var arr = [1, [2, [3, 4]]];
console.log(flatten(arr)) // [1, 2, 3, 4]
```

知道了效果是什么样的了，我们可以去尝试着写这个 flatten 函数了

#### 递归

我们最一开始能想到的莫过于循环数组元素，如果还是一个数组，就递归调用该方法：

```javascript
// 方法 1
var arr = [1, [2, [3, 4]]];

function flatten(arr) {
    var result = [];
    for (var i = 0, len = arr.length; i < len; i++) {
        if (Array.isArray(arr[i])) {
            result = result.concat(flatten(arr[i]))
        }
        else {
            result.push(arr[i])
        }
    }
    return result;
}


console.log(flatten(arr))
```

#### toString

如果数组的元素都是数字，那么我们可以考虑使用 toString 方法，因为：

```javascript
[1, [2, [3, 4]]].toString() // "1,2,3,4"
```

调用 toString 方法，返回了一个逗号分隔的扁平的字符串，这时候我们再 split，然后转成数字不就可以实现扁平化了吗？

```javascript
// 方法2
var arr = [1, [2, [3, 4]]];

function flatten(arr) {
    return arr.toString().split(',').map(function(item){
        return +item
    })
}

console.log(flatten(arr))
```

然而这种方法使用的场景却非常有限，如果数组是 [1, '1', 2, '2'] 的话，这种方法就会产生错误的结果。

#### reduce

既然是对数组进行处理，最终返回一个值，我们就可以考虑使用 reduce 来简化代码：

```javascript
// 方法3
var arr = [1, [2, [3, 4]]];

function flatten(arr) {
    return arr.reduce(function(prev, next){
        return prev.concat(Array.isArray(next) ? flatten(next) : next)
    }, [])
}

console.log(flatten(arr))
```

#### ...

ES6 增加了扩展运算符，用于取出参数对象的所有可遍历属性，拷贝到当前对象之中：

```javascript
var arr = [1, [2, [3, 4]]];
console.log([].concat(...arr)); // [1, 2, [3, 4]]
```

我们用这种方法只可以扁平一层，但是顺着这个方法一直思考，我们可以写出这样的方法：

```javascript
// 方法4
var arr = [1, [2, [3, 4]]];

function flatten(arr) {

    while (arr.some(item => Array.isArray(item))) {
        arr = [].concat(...arr);
    }

    return arr;
}

console.log(flatten(arr))
```

#### underscore

那么如何写一个抽象的扁平函数，来方便我们的开发呢，所有又到了我们抄袭 underscore 的时候了~

在这里直接给出源码和注释，但是要注意，这里的 flatten 函数并不是最终的 _.flatten，为了方便多个 API 进行调用，这里对扁平进行了更多的配置。

```javascript
/**
 * 数组扁平化
 * @param  {Array} input   要处理的数组
 * @param  {boolean} shallow 是否只扁平一层
 * @param  {boolean} strict  是否严格处理元素，下面有解释
 * @param  {Array} output  这是为了方便递归而传递的参数
 * 源码地址：https://github.com/jashkenas/underscore/blob/master/underscore.js#L528
 */
function flatten(input, shallow, strict, output) {

    // 递归使用的时候会用到output
    output = output || [];
    var idx = output.length;

    for (var i = 0, len = input.length; i < len; i++) {

        var value = input[i];
        // 如果是数组，就进行处理
        if (Array.isArray(value)) {
            // 如果是只扁平一层，遍历该数组，依此填入 output
            if (shallow) {
                var j = 0, length = value.length;
                while (j < length) output[idx++] = value[j++];
            }
            // 如果是全部扁平就递归，传入已经处理的 output，递归中接着处理 output
            else {
                flatten(value, shallow, strict, output);
                idx = output.length;
            }
        }
        // 不是数组，根据 strict 的值判断是跳过不处理还是放入 output
        else if (!strict){
            output[idx++] = value;
        }
    }

    return output;

}
```

解释下 strict，在代码里我们可以看出，当遍历数组元素时，如果元素不是数组，就会对 strict 取反的结果进行判断，如果设置 strict 为 true，就会跳过不进行任何处理，这意味着可以过滤非数组的元素，举个例子：

```javascript
var arr = [1, 2, [3, 4]];
console.log(flatten(arr, true, true)); // [3, 4]
```

那么设置 strict 到底有什么用呢？不急，我们先看下 shallow 和 strct 各种值对应的结果：

- shallow true + strict false ：正常扁平一层
- shallow false + strict false ：正常扁平所有层
- shallow true + strict true ：去掉非数组元素
- shallow false + strict true ： 返回一个[]

我们看看 underscore 中哪些方法调用了 flatten 这个基本函数：

#### _.flatten

首先就是 _.flatten：

```javascript
_.flatten = function(array, shallow) {
    return flatten(array, shallow, false);
};
```

在正常的扁平中，我们并不需要去掉非数组元素。

#### _.union

接下来是 _.union：

该函数传入多个数组，然后返回传入的数组的并集，

举个例子：

```javascript
_.union([1, 2, 3], [101, 2, 1, 10], [2, 1]);
=> [1, 2, 3, 101, 10]
```

如果传入的参数并不是数组，就会将该参数跳过：

```javascript
_.union([1, 2, 3], [101, 2, 1, 10], 4, 5);
=> [1, 2, 3, 101, 10]
```

为了实现这个效果，我们可以将传入的所有数组扁平化，然后去重，因为只能传入数组，这时候我们直接设置 strict 为 true，就可以跳过传入的非数组的元素。

```javascript

function unique(array) {
   return Array.from(new Set(array));
}

_.union = function() {
    return unique(flatten(arguments, true, true));
}
```

#### _.difference

是不是感觉折腾 strict 有点用处了，我们再看一个 _.difference：

语法为：

> _.difference(array, *others)

效果是取出来自 array 数组，并且不存在于多个 other 数组的元素。跟 _.union 一样，都会排除掉不是数组的元素。

举个例子：

```javascript
_.difference([1, 2, 3, 4, 5], [5, 2, 10], [4], 3);
=> [1, 3]
```

实现方法也很简单，扁平 others 的数组，筛选出 array 中不在扁平化数组中的值：

```javascript
function difference(array, ...rest) {

    rest = flatten(rest, true, true);

    return array.filter(function(item){
        return rest.indexOf(item) === -1;
    })
}
```

