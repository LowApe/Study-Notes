# 学习Node-SQLite

## 1. introduce 介绍
作为程序员，最重要的技能之一是能够识别和使用适合特定任务的工具。在数据库管理的上下文中，这将意味着使用SQL来指定，存储，更新和检索数据。在Web编程的上下文中，这将意味着编写JavaScript来自动化，操作和返回相关值 - 用于在网站中呈现或在后端脚本中使用。那么，当我们需要两者时会发生什么？如果我们想从SQL数据库中检索数据（使用我们的数据库管理技能），然后通过JavaScript函数（使用我们的Web编程技能）操作和公开该数据，该怎么办？

在本课中，我们将学习如何从`JavaScript`中管理`SQLite`数据库。我们将看到如何执行数据库管理的所有基本功能- `CREATE`ing `INSERT`ing and `SELECT`ing，然后使用`JavaScript`的全部力量与数据进行交互-写功能，挥舞的对象，并进行计算。重要的是要知道，如果需要，可以纯粹通过`SQL`或纯粹通过`JavaScript`获得此处的许多结果。但是用一种语言执行（和回读）简单的东西可能很难在另一种语言中编写和理解。

在工作区中，有代码打开与`SQLite`数据库的连接。有一个函数`getAverageTemperatureForYear()`可以year作为一个参数。该函数从该年度检索温度，然后计算年份的平均值。我们用不同的年份称它，说明了能够用JavaScript为我们的SQL查询提供动力的能力。

## 2. Opening A Database  打开数据库
为了让这两个世界进行交流，我们将把一个包导入我们的JavaScript代码中。该软件包将允许我们打开与SQLite数据库通信的渠道。一旦我们这样做，我们就可以直接在JavaScript中编写SQL了！

第一项业务是导入将促进此连接的模块。回想一下，要在JavaScript中导入模块，我们可以这样使用`require()`：

```js
const sqlite3 = require('sqlite3');
```
我们将要使用的第一种`sqlite3`方法是打开新数据库的方法。在SQLite中，数据库对应于单个文件，因此打开此数据库所需的唯一参数是SQLite用于保存数据库的文件的路径。
```js
const db = new sqlite3.Database('./db.sqlite');
```
此代码将在当前文件夹中创建一个名为的新数据库文件`db.sqlite`。然后我们将有一个数据库进行交互！

## 3. Retrieving All Rows 检索所有行

在上一个练习中，我们能够导入`'sqlite3'`库并使用它来打开我们的SQLite数据库 - 到目前为止一切顺利！但是我们仍然没有从中检索到任何信息。由于我们可以作为JavaScript对象访问我们的数据库，因此我们尝试在其上运行查询。回想一下，查询是一个与数据库对话并从中请求特定信息的语句。要执行查询并检索返回的所有行，我们使用`db.all()`，如下所示：

```js
db.all("SELECT * FROM Dog WHERE breed='Corgi'", (error, rows) => {
  printQueryResults(rows);
});
```
## 4. Retrieving A Single Row 检索单行

`db.all()`是一个有用的工具，用于获取符合特定条件的所有数据。但是，如果我们只想得到一个特定的行呢？

```js
db.all("SELECT * FROM Dog", (error, rows) => {
  printQueryResults(rows.find(row => row.id === 1));
});
```
在此示例中，我们从数据库中获取所有行。这样做会填充一个`JavaScript`变量，`rows`它包含我们`SELECT`语句的结果（数据库中的所有行）。我们使用JavaScript的`.find()`方法来查找ID为1的行。然后打印出该行。

使用一个小型数据库，这可能没问题，但如果数据库在任何意义上都很大，那将是一个相当大且不必要的负载。幸运的是，我们有一个不同的方法可以从数据库中获取单行：`db.get()`。看到它的实际效果：

```js
db.get("SELECT * FROM Dog WHERE owner_name = 'Charlie'", (error, row) => {
  printQueryResults(row);
});
```

有时候，我们需要知道的是我们的匹配查询记录是否存在（例如：上面的代码会回答“查理是否拥有宠物狗？”这取决于是否`row` `undefined`）。有时我们知道只有一行，因为我们正在搜索特定的ID。有时我们只想要一个符合我们描述的行的示例。在上面的代码中，我们只打印有关一只狗的信息。为实现这一目标，我们使用`db.get()`而不是`db.all()`。

重要的是要注意，即使多行与查询匹配，`db.get()`也只会返回单个结果。在上面的示例中，如果“Charlie”拥有多只狗，则提供的代码仍然只会打印有关一只狗的信息。

## 5. Using Placeholders 使用占位符

现在，当我们确切地知道我们正在寻找什么时，我们知道如何从数据库中检索数据。但是，我们可能并不总是知道在编写程序时我们需要搜索的值。当我们编写JavaScript函数时，我们给函数参数在调用函数时会有许多不同的值。占位符解决了SQL查询领域中的类似问题。有时我们会根据用户的提交搜索我们的数据库。或者我们可能会发现自己想要执行一系列循环遍历某些外部数据的查询。

在这些情况下，我们将不得不使用占位符。占位符是我们想要使用变量内容进行插值的 SQL查询的一部分。我们希望将JavaScript变量的值放在SQL查询中。要正确执行此操作，我们需要将一个特定参数传递给我们的`db.run()`命令，该命令将告诉它如何插入查询。

```js
const furLength1 = "short";
const furLength2 = "long";
const furColor1 = "brown";
const furColor2 = "grey";
const findDogByFur = (length, color) => {
  db.all(
    "SELECT * FROM Dog WHERE fur_length = $furLength AND fur_color = $furColor",
    {
      $furLength: length,
      $furColor: color
    },
    (error, rows) => {
      printQueryResults(rows);
    }
});

findDogByFur(furLength1, furColor1); // prints all dogs with short brown fur.
findDogByFur(furLength2, furColor1); // prints all dogs with long brown fur.
findDogByFur(furLength1, furColor2); // prints all dogs with short grey fur.
findDogByFur(furLength2, furColor2); // prints all dogs with long grey fur

```


正如我们在上面的例子中所看到的，占位符的强大之处在于我们不需要在编写查询时准确地知道我们正在搜索的数据。我们可以使用这些占位符，然后，当我们想要找到的值时，我们可以将它们插入到查询中。这是一个非常有效的工具，可以让我们在数据库查询中利用我们的编程技巧。
