# 表格操作

## 单元格刷新

- **格式**：

    ```json
    {
        "t": "v",
        "i": 3,
        "v": "asdf",
        "r": 5,
        "c": 7
    }
    ```

- **说明**：

    |参数|说明|
    | ------------ | ------------ |
    |t|操作类型表示符号|
    |i|当前sheet的index值|
    |v|单元格的值|
    |r|单元格的行号|
    |c|单元格的列号|

- **后台更新**：

    单元格更新主要是更新 `jfgridfile[i].celldata` 参数，该参数是一个数组：
    ```json
    [
        {r:0, c:1, v: "值1"},
        {r:10, c:11, v:"值2"},
        {r:10, c:11, v:{f:"=sum", v:"100"}}
    ]
    ```
    存储了sheet中所有单元格中的值，Luckysheet在建立的时候会根据 `jfgridfile[i].row` 和 `jfgridfile[i].column` 的行列数量大小新建一个表格data，然后再使用 `data[r][c]=v` 的方式填充表格数据，空数据单元格以null表示。

    后台在保存前台推送的数据时，首先需要把参数转换为 `{r:0, c:1:v:100}` 的格式，然后更新 `jfgridfile[i].celldata` 字段，如果存在该单元格则更新，如果没有则添加，如果存在该单元格但是v为null则删除该单元格。

- **前台查看**：

    可以修改任意单元格的数值，然后到chrome控制台中查看"t"=="v"的操作。
  
## config操作

- **格式**：

  ```json
  {
    "t": "cg",
    "i": 3,
    "v": {
        "7": 192
    },
    "k": "rowlen"
  }
  ```

- **说明**：

    |参数|说明|
    | ------------ | ------------ |
    |t|操作类型表示符号|
    |i|当前sheet的index值|
    |v|需要更新的内部key-value|
    |k|操作的key名称|

- **后台更新**：

    更新 `jfgridfile[i].config.[k][v.key] = v.value` ，如果config中不存k，则新建一个k属性并设置为空，如果k中不存在v.key中，则新建一个v.key再更新v.value。

    1. 修改行高度举例：
       - 输入：`{"t":"cg","i":3,"v":{"3":10, "5":70, "10":100},"k":" rowlen"}`
       - 更新：`jfgridfile[3].config.["rowlen"]["3"] = 10`

    2. 修改列宽度举例：
       - 输入：`{"t":"cg","i":1,"v":{"20":74, "15":170, "6":40},"k":" columlen"}`
       - 更新：`jfgridfile[1].config.["columlen"]["20"] = 74`
        
    3. 合并单元格举例：
       - 输入：`{"t":"cg","i":1,"v":{"5_10":{row:[1,3], column:[3,5]},"k":" merge "}`
       - 更新：`jfgridfile[1].config.["merge"]["5_10"] = {row:[1,3], column:[3,5]}`
 
## 通用保存

- **格式**：

  ```json
  {
    "t": "all",
    "i": 3,
    "v": {
        "v": 1,
        "m":1,
    },
    "k": "freezen",
    "s": false
  }
  ```

- **说明**：

    |参数|说明|
    | ------------ | ------------ |
    |t|操作类型表示符号|
    |i|当前sheet的index值|
    |v|需要更新的内部key-value|
    |k|需要保存的key-value中的value|
    |s|如果是true则v保存为字符串，否则按照对象进行保存|

- **后台更新**：

    `jfgridfile[3].[k]= v`
    如果s为true，则为 `jfgridfile[3].[k]= JSON.stringify(v)`

    1. 数据透视表：
       - 输入：`{"t":"all","i":1,"v":{………},"k":"pivotTable", "s": false}`
       - 更新：`jfgridfile[1].["pivotTable"] = {………}`

    2. 冻结行列：
       - 输入：`{"t":"all","i":3,"v":{………},"k":"freezen", "s": false}`
       - 更新：`jfgridfile[3].["freezen"] = {………}`

    3. 筛选范围：
       - 输入：`{"t":"all","i":3,"v":{………},"k":"filter_select", "s": true }`
       - 更新：`jfgridfile[3].["filter_select"] = JSON.stringify ({………})`
        
    4. Sheet名称：
       - 输入：`{"t":"all","i":1,"v":"文档","k":"name", "s": false}`
       - 更新：`jfgridfile[1].["name"] = "文档"`

    5. Sheet颜色：
       - 输入： `{"t":"all","i":2,"v":"#FFF000","k":"color", "s": false}`
       - 更新：`jfgridfile[2].["color"] = "#FFF000"`

## 函数链操作

- **格式**：

  ```json
  {
    "t": "fc",
    "i": 0,
    "v": {
        "r": 3,
        "c": 7,
        "index": 0,
        "func": [
            true,
            187282,
            "=SUM(E4:G4)"
        ]
    },
    "op": "add",
    "pos": 0
  }
  ```

- **说明**：

    |参数|说明|
    | ------------ | ------------ |
    |t|操作类型表示符号|
    |i|当前sheet的index值|
    |v|对象值，这里对象的内部字段不需要单独更新，所以存为文本即可|
    |op|操作类型,add为新增，update为更新，del为删除|
    |pos|更新或者删除的函数位置|

- **后台更新**：

    calcChain为一个数组
    - 如果op的值add则添加到末尾 `jfgridfile[i].calcChain.push (v)`， 
    - 如果op的值update则更新 `jfgridfile[i].calcChain[pos]= v`，
    - 如果op的值del则删除 `jfgridfile[i].calcChain.splice(pos, 1)`。

- **前台查看**：

    可以修改任意单元格的数值，然后到chrome控制台中查看"t"=="v"的操作。

## 行列操作

### 删除行或列

- **格式**：

  ```json
  {
    "t": "drc",
    "i": 3,
    "v": {
        "index": 6,
        "len": 2
    },
    "rc": "r"
  }
  ```

- **说明**：

    <table>
        <tr>
            <td colspan="2">参数</td> 
            <td>说明</td> 
        </tr>
        <tr>
            <td colspan="2">t</td> 
            <td>操作类型表示符号</td> 
        </tr>
        <tr>
            <td colspan="2">i</td> 
            <td>当前sheet的index值</td> 
        </tr>
        <tr>
            <td colspan="2">rc</td> 
            <td>行操作还是列操作，值r代表行，c代表列</td> 
        </tr>
        <tr>
            <td rowspan="2">v</td> 
            <td>index</td> 
            <td>从第几行或者列开始删除</td> 
        </tr>
        <tr>
            <td>len</td> 
            <td>删除多少行或者列</td> 
        </tr>
        
    </table>

- **后台更新**：
  
    如果rc的值是r删除行， 如果rc的值为c则删除列， 例如rc=r，index=4，len=5，则代表从第4行开始删除之后的5行（4、5、6、7、8）。

    主要是对 `jfgridfile[i].celldata` 中的单元格进行操作，删除参数中所描述符合条件的单元格并且更新其他单元格的行列值，以上述为例，首先查找单元格中r值在4到8的所有单元格并删除，然后把本来行号9以后的单元格的r值减去5，最后把 `jfgridfile[i].row` 减去5。
    如果v值为 `"#__qkdelete#"`（不含引号），则此处为需要删除的单元格。

- **前台查看**：

    可以删除行或者列，然后到chrome控制台中查看"t"=="drc"的操作。

### 增加行或列

- **格式**：

  ```json
  {
    "t": "arc",
    "i": 0,
    "v": {
        "index": 5,
        "len": 10,
        "data": []
    },
    "rc": "c"
  }
  ```

- **说明**：

    <table>
        <tr>
            <td colspan="2">参数</td> 
            <td>说明</td> 
        </tr>
        <tr>
            <td colspan="2">t</td> 
            <td>操作类型表示符号</td> 
        </tr>
        <tr>
            <td colspan="2">i</td> 
            <td>当前sheet的index值</td> 
        </tr>
        <tr>
            <td colspan="2">rc</td> 
            <td>行操作还是列操作，值r代表行，c代表列</td> 
        </tr>
        <tr>
            <td rowspan="3">v</td> 
            <td>index</td> 
            <td>从第几行或者列开始新增</td> 
        </tr>
        <tr>
            <td>len</td> 
            <td>增加多少行或者列</td> 
        </tr>
        <tr>
            <td>data</td> 
            <td>新增行或者列的内容</td> 
        </tr>
        
    </table>

- **后台更新**：
  
    如果rc的值是r新增行， 如果rc的值为c则新增列， 例如rc=r，index=4，len=5，则代表从第4行开始增加5行，如果data为空则增加空行，如果data不为空则用data中的数组添加新增的行中。
    主要是对 `jfgridfile[i].celldata` 中的单元格进行操作，以上述为例，首先 `jfgridfile[i].row` 加5，然后把r大于4的单元格的整体的r值+5，如果data为空则增加空行则结束，如果data不为空则把二维数组data转换为 `{r:0,c:0,v:100}` 的格式并添加到celldata中，转换的伪代码如下：
    ```javascript
    var ret = [];
    for(var r=0;r<data.length;r++){
        for(var c=0;c<data[0].length;c++){
            if(d[r][c]==null){
                continue;
            }
            ret.push({r:r+5, c:c, v: data[r][c]});
        }
    }
    return ret;
    ```

- **前台查看**：
    可以新增行或者列，然后到chrome控制台中查看"t"=="arc"的操作。如果想查看具有data值的操作，则先删除某几行或几列，然后再撤销删除（Ctrl+Z），就能看到。

## 筛选操作

### 选择筛选条件

- **格式**：

  ```json
  {
    "t": "f",
    "i": 0,
    "v": "{\"caljs\":{},\"selected\":{\"青岛\":\"1\",\"广西\":\"1\",\"重庆\":\"1\"},\"rowhidden\":{\"1\":0,\"2\":0,\"3\":0,\"4\":0,\"6\":0,\"7\":0,\"8\":0,\"9\":0,\"10\":0,\"11\":0,\"12\":0,\"13\":0,\"14\":0,\"15\":0,\"16\":0,\"17\":0,\"18\":0,\"19\":0,\"21\":0,\"22\":0,\"24\":0,\"25\":0,\"26\":0,\"27\":0,\"28\":0,\"29\":0,\"30\":0,\"31\":0,\"32\":0,\"33\":0,\"34\":0,\"35\":0}}",
    "op": "upOrAdd",
    "pos": 1
  }
  ```

- **说明**：

    |参数|说明|
    | ------------ | ------------ |
    |t|操作类型表示符号|
    |i|当前sheet的index值|
    |v|对象值，这里对象的内部字段不需要单独更新，所以存为文本即可|
    |op|操作类型upOrAdd为更新，如果不存在则增加，del为删除|
    |pos|更新或者删除的option位置|

- **后台更新**：
  
    更新 `jfgridfile[i].filter = { pos : v }`， v值为一个JSON格式的字符串。filter为一个键值对，key表示选项位置的索引值（以字符表示），v表示一个json字符串参数。filter代表一个筛选条件的集合。

### 清除筛选

- **格式**：

  ```json
  {
    "t": "fsc",
    "i": 0,
    "v": null
  }
  ```

- **后台更新**：
  
    清除 `jfgridfile[i]. filter = null` ， `jfgridfile[i]. filter_select = null`。

### 恢复筛选

- **格式**：

  ```json
  {
    "t": "fsr",
    "i": 0,
    "v": {
        "filter": [],
        "filter_select": {}
    }
  }
  ```

- **后台更新**：
  
    清除 `jfgridfile[i]. filter = v.filter`， `jfgridfile[i]. filter_select = v. filter_select`。

## sheet操作

### 新建sheet

- **格式**：

  ```json
  {
    "t": "sha",
    "i": null,
    "v": {
        "name": "Sheet4",
        "color": "",
        "status": "0",
        "order": 3,
        "index": 3,
        "data": [],
        "config": {},
        "pivotTable": null,
        "isPivotTable": false
    }
  }
  ```

- **说明**：

    <table>
        <tr>
            <td colspan="2">参数</td> 
            <td>说明</td> 
        </tr>
        <tr>
            <td colspan="2">t</td> 
            <td>操作类型表示符号</td> 
        </tr>
        <tr>
            <td colspan="2">i</td> 
            <td>当前sheet的index值</td> 
        </tr>
        <tr>
            <td rowspan="9">v</td> 
            <td>name</td> 
            <td>隐藏后跳转的sheet的index值</td> 
        </tr>
        <tr>
            <td>color</td> 
            <td>Sheet颜色</td> 
        </tr>
        <tr>
            <td>status</td> 
            <td>激活状态</td> 
        </tr>
        <tr>
            <td>order</td> 
            <td>Sheet摆放顺序</td> 
        </tr>
        <tr>
            <td>index</td> 
            <td>Index索引</td> 
        </tr>
        <tr>
            <td>celldata</td> 
            <td>单元格数据集</td> 
        </tr>
        <tr>
            <td>config</td> 
            <td>设置</td> 
        </tr>
        <tr>
            <td>pivotTable</td> 
            <td>数据透视表设置</td> 
        </tr>
        <tr>
            <td>isPivotTable</td> 
            <td>是否数据透视表</td> 
        </tr>
        
    </table>

- **后台更新**：
  
    添加一行（一个文档）到数据库中。

### 复制sheet

- **格式**：

  ```json
  {
    "t": "shc",
    "i": "新建sheet的位置",
    "v": {
        "copyindex": "copyindex"
    }
  }
  ```

- **后台更新**：
  
    复制表格中的sheet索引值为copyindex并添加到数据库中，添加的设置该新文档的index为i对应的值。

### 删除sheet

- **格式**：

  ```json
  {
    "t": "shd",
    "i": null,
    "v": {
        "deleIndex": 0
    }
  }
  ```

- **说明**：

    <table>
        <tr>
            <td colspan="2">参数</td> 
            <td>说明</td> 
        </tr>
        <tr>
            <td colspan="2">t</td> 
            <td>操作类型表示符号</td> 
        </tr>
        <tr>
            <td rowspan="2">v</td> 
            <td>deleIndex</td> 
            <td>需要删除的sheet索引</td> 
        </tr>
                
    </table>

- **后台更新**：
  
    删除索引为deleIndex对应值得sheet。

### 位置

- **格式**：

  ```json
  {
    "t": "shr",
    "v": {
        "index": "positon"
    }
  }
  ```

- **说明**：

    |参数|说明|
    | ------------ | ------------ |
    |t|操作类型表示符号|
    |v|设置Sheet的排序，为一个键值对，key代表sheet的index，value代表order值。格式为：`{"1": 3, "2":1, "0": 2, "3":0}|

- **后台更新**：
  
    对sheet的index等于key的页设置其order属性为value值。示例：

    `jfgridfile[key1].order = value1`
    `jfgridfile[key2].order = value2`
    `jfgridfile[key3].order = value3`

## sheet属性(隐藏或显示)

- **格式**：

  ```json
  {
    "t": "sh",
    "i": 0,
    "v": 1,
    "op": " hide",
    "cur": 2
  }
  ```

- **说明**：

    |参数|说明|
    | ------------ | ------------ |
    |t|操作类型表示符号|
    |i|当前sheet的index值|
    |op|操作选项，有hide、show。|
    |v|如果hide为1则隐藏，为0或者空则为显示|
    |cur|隐藏后设置索引对应cur的sheet为激活状态。|

- **后台更新**：
  
    更新"i"对应sheet的根路径hide字段为v，当隐藏时status值为0，当显示时为1， 如果为隐藏则更新index对应cur的sheet的status状态为1。

## 表格信息更改

### 表格名称

- **格式**：

  ```json
  {
    "t": "na",
    "i": null,
    "v": "数据"
  }
  ```

- **说明**：

    |参数|说明|
    | ------------ | ------------ |
    |t|操作类型表示符号|
    |v|表格的名称|

- **后台更新**：
  
    根据gridkey更新数据库中的表格名称。

### 缩略图

- **格式**：

  ```json
  {
    "t": "thumb",
    "img": "base64",
    "curindex": "curindx"
  }
  ```

- **说明**：

    |参数|说明|
    | ------------ | ------------ |
    |t|操作类型表示符号|
    |img|当前表格的缩略图，为base64字符串|
    |curindex|当前表格默认打开的sheet|

- **后台更新**：
  
    根据gridkey更新mysql中表格的缩略图字段为img值，同时更新index为curindex值的sheet的status字段为1，设置其他sheet的status值为0。