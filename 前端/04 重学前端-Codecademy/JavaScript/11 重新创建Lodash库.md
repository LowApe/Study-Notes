# 重新创建Lodash库

> 练习 Lodash 的几个处理数组的方法

****
> 输入的数字只要在边界值范围内就靠近那个边界值就等于这个值

```js
clamp(number, lower, upper) {
    let lowerClampedValue = Math.max(number, lower);
    let clampedValue = Math.min(lowerClampedValue, upper);
    return clampedValue;
}

```


****
> 数值在范围之间就返回 boolean 如果范围颠倒,进行互换

```js
inRange(number, start, end) {

    if (typeof end === 'undefined') {
        end = start;
        start = 0;
    }
    if (start > end) {
        start = end + start;
        end = start - end;
        start = start - end;
    }
    if (number >= start && number < end) {
        return true;
    } else {
        return false;
    }
}
```

****
> 将识别单词

```js
words(string) {
    let words = string;
    return words.split(' ');
}
```
****
> 个字符串两边添加相同的空格

```js
pad(string, length) {
    if (length <= string.length) {
        return string;
    }
    let startPaddingLength = Math.floor((length - string) / 2);
    let endPaddingLength = length - string - startPaddingLength;
    let paddedString = ' '.repeat(startPaddingLength) + string + ' '.repeat(endPaddingLength);
    return paddedString;
}
```
****
>  得到键值对中的值

```js
has(object, key) {
    let hasValue = (typeof object[key] != 'undefined');
    return hasValue;
}
```

****
> 键值互换

```js
invert(object) {
    let invertedObject={};
    for (var key in object) {
        //遍历的 key 是键值!!!
        let originalValue = object[key];
        invertedObject[originalValue]=key;
    }
    return invertedObject;
}
```

****
> 寻找对象中的 key 值

```js
findKey(object,predicate){
    for (var key in object) {
        let value =object[key];
        let predicateReturnValue=predicate(value);
        if(predicateReturnValue){
            return key;
        }
        return undefined;
    }
}
```
****

>  从输入索引开始删除前面的

```js

drop(array,n){
    if(n===undefined){
        n=1;
    }
    let droppedArray=array.slice(n);
    return droppedArray;
}
```
****
> 循环删除符合要求的

```js
dropWhile(array,predicate){
    //符合的都删除
    let dropNumber = {};
    let dropArray= {};

    for (var index = 0; index < array.length; index++) {
        let element = array[index];
        if(!predicate(element,index)){
            dropNumber=index;
        }
        dropArray=this.drop(array,dropNumber);
    }

    return dropArray;
}
```
****
> 每个固定位置截取

```js
chunk(array,size){
    if(size === undefined){
        size=1;
    }
    let arrayChunks =[];
    for (var i = 0; i < array.length; i+=size) {
        let arrayChunk = array.slice(i,i+size);
        arrayChunks.push(arrayChunk);
    }
    return arrayChunks;

}
}
```
