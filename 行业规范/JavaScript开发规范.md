[TOC]
###文件名
* 文件夹/文件的命名统一用小写
    - 保证项目有良好的可移植性，可跨平台 

###文件引用路径
* 因为文件命名统一小写，引用也需要注意大小写问题

###js变量
* 变量
    - 命名方式：小驼峰
    - 命名规范：前缀名词
    - 命名建议：语义化
* 常量
    - 命名方式：全部大写
    - 命名规范：使用大写字母和下划线来组合命名，下划线用以分割单词
    - 命名建议：语义化
* 函数
    - 命名方式：小驼峰式命名法。
    - 命名规范：前缀应当为动词。
    - 命名建议：语义化
    | 动词|含义|返回值|
    | ------------ | ------------ | ------------ |
    |can|判断是否可执行某个动作(权限)|函数返回一个布尔值。true：可执行；false：不可执行| 
    |has|判断是否含有某个值|函数返回一个布尔值。true：含有此值；false：不含有此值|
    |is|判断是否为某个值|函数返回一个布尔值。true：为某个值；false：不为某个值|
    |get|获取某个值|函数返回一个非布尔值|
    |set|设置某个值|无返回值、返回是否设置成功或者返回链式对象|
    |load|加载某些数据|无返回值或者返回是否加载完成的结果|
    ```shell
    // 是否可阅读
    function canRead(): boolean {
    return true;
    }
    // 获取名称
    function getName(): string {
    return this.name;
    }
    ```
* 类 & 构造函数
    - 命名方法：大驼峰式命名法，首字母大写。
    - 命名规范：前缀为名称。  

    ```shell
    class Person {
        public name: string;
        constructor(name) {
            this.name = name;
        }
    }
    const person = new Person('mevyn');
    ```

* 类的成员
    - 公共属性和方法：跟变量和函数的命名一样。
    - 私有属性和方法：前缀为_(下划线)，后面跟公共属性和方法一样的命名方式。 
    ```shell
    class Person {
        private _name: string;
        constructor() { }
        // 公共方法
        getName() {
            return this._name;
        }
        // 公共方法
        setName(name) {
            this._name = name;
        }
    }
    const person = new Person();
    person.setName('mervyn');
    person.getName(); // ->mervyn
    ```
###注释规范

* 行内注释
    ```shell
        // 用来显示一个解释的评论
        // -> 用来显示表达式的结果
        // >用来显示 console 的输出结果
    ```
* 单行注释
    ```shell
        // 调用了一个函数；1)单独在一行
        setTitle();
    ```
* 多行注释
    ```shell
        /*
        * 代码执行到这里后会调用setTitle()函数
        * setTitle()：设置title的值
        */
        setTitle();
    ```
* 函数(方法)注释
    ```shell
    /** 
    * 函数说明 
    * @关键字 
    */
    ```
| 注释名|语法|含义|示例|
| ------------ | ------------ | ------------ | ------------ |
|@param|@param 参数名 {参数类型} 描述信息|描述参数的信息|@param name {String} 传入名称|
|@return|@return {返回类型} 描述信息|描述返回值的信息|@return {Boolean} true:可执行;false:不可执行|
|@author|@author 作者信息 [附属信息：如邮箱、日期]|描述此函数作者的信息|@author 张三 2015/07/21|
|@version|@version XX.XX.XX|描述此函数的版本号|@version 1.0.3|
|@example|@example 示例代码|演示函数的使用|@example setTitle(‘测试’)|
```shell
/**
* 合并Grid的行
* @param grid {Ext.Grid.Panel} 需要合并的Grid
* @param cols {Array} 需要合并列的Index(序号)数组；从0开始计数，序号也包含。
* @param isAllSome {Boolean} ：是否2个tr的cols必须完成一样才能进行合并。true：完成一样；false(默认)：不完全一样
* @return void
* @author polk6 2015/07/21 
* @example
* _________________                             _________________
* |  年龄 |  姓名 |                             |  年龄 |  姓名 |
* -----------------      mergeCells(grid,[0])   -----------------
* |  18   |  张三 |              =>             |       |  张三 |
* -----------------                             -  18   ---------
* |  18   |  王五 |                             |       |  王五 |
* -----------------                             -----------------
*/
function mergeCells(grid: Ext.Grid.Panel, cols: Number[], isAllSome: boolean = false) {
  // Do Something
}
```

