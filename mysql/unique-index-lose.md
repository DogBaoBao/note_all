# Mysql联合唯一索引存在空值时唯一约束失效

## 问题

> 因为 gorm 的默认填充有 `deleted_at` 字段，一开始设置了 open\_code 为唯一索引，软删除再插入新数据会出现唯一索引冲突，所以把 `open_code + delete_at` 创建了联合唯一索引

然而这之后我重复插入（重复注册账号），发现 `open_code` 并不会出现唯一索引冲突，唯一登录的 `open_code` 字段重复，正常应该报索引冲突的错误。

## 背景

表有联合唯一索引如下图：

![](../.gitbook/assets/image%20%281%29.png)

但是还是可以插入重复的 open\_code

![](../.gitbook/assets/image.png)

其中脱敏的 open\_code 都是重复数据

## 原因

> A UNIQUE index creates a constraint such that all values in the index must be distinct. An error occurs if you try to add a new row with a key value that matches an existing row. This constraint does not apply to NULL values except for the BDB storage engine. For other engines, a UNIQUE index allows multiple NULL values for columns that can contain NULL.

唯一约束对NULL值不适用。原因可以这样解释： 比如我们有一个单列的唯一索引，既然实际会有空置的情况，那么这列一定不是`NOT NULL`的，如果唯一约束对空值也有起作用，就会导致仅有一行数据可以为空，这可能会和实际的业务需求想冲突的，所以通常Mysql的存储引擎的唯一索引对NULL值是不适用的。 这也就倒是联合唯一索引的情况下，只要某一列为空，就不会报唯一索引冲突。

## 解决方案

### 方案一

比如在 Java 中，我会为空的列定义一个为空的特殊值来表示 NULL，时间的话可以选择一个历史时间 1997-07-01 00:00:00，数字类型的话可以是 -1。

这个方案需要把唯一性的字段和删除时间加联合所以索引，并且在查询的时候，要带上默认的删除时间条件。

我们在 gorm 里面，默认是 `deleted_at` 为 NULL，在查找的时候会如下效果

```sql
SELECT count(*) FROM `xxx_user`  WHERE `xxx_user`.`deleted_at` IS NULL 
```

 因此还需要做一定的改造， 让查询带上 deleted\_at = '默认时间'  的条件（注意：已经删除的可能也需要查询）

### 方案二

每次删除的时候，给唯一性字段加上时间信息，比如：

```sql
UPDATE `gdcloud_user` SET name = concat(name, "2020-09-11 13:17:17.887"), `deleted_at`="2020-09-11 13:17:17.887" WHERE name = "tiecheng"
```

* 这个方案使用很不方便，唯一字段不只 name 的时候软删除要 SET 多个字段
* 加了时间信息后这个字段的长度一般都会大很多，占用了不必要的空间
* 为图方便的话例子使用了 concat 函数，数据库操作不建议使用函数。当然也可以先查询出来所有的字段在内存做处理完直接设置值

### gorm v1 源码分析

#### callback 在哪调用

```go
func (scope *Scope) callCallbacks(funcs []*func(s *Scope)) *Scope {
   defer func() {
      if err := recover(); err != nil {
         if db, ok := scope.db.db.(sqlTx); ok {
            db.Rollback()
         }
         panic(err)
      }
   }()
   for _, f := range funcs {
      (*f)(scope)
      if scope.skipLeft {
         break
      }
   }
   return scope
}
```

 gorm 的很多操作都是通过 callback 来执行的，比如时间字段填充。其中 `(*f)(scope)` 是执行一个一个 callback 的入口。

####  查询 SQL `delete_at IS NULL` 如何拼接

会通过 `github.com/jinzhu/gorm.rowQueryCallback` 进入 SQL 拼接

```go
func (scope *Scope) prepareQuerySQL() {
	if scope.Search.raw {
		scope.Raw(scope.CombinedConditionSql())
	} else {
		scope.Raw(fmt.Sprintf("SELECT %v FROM %v %v", scope.selectSQL(), scope.QuotedTableName(), scope.CombinedConditionSql()))
	}
	return
}
```

 关心这里的 scope.CombinedConditionSql\(\)，是产生 Where 条件的，跟进去看：

```go
func (scope *Scope) whereSQL() (sql string) {
	var (
		quotedTableName                                = scope.QuotedTableName()
		deletedAtField, hasDeletedAtField              = scope.FieldByName("DeletedAt")
		primaryConditions, andConditions, orConditions []string
	)

	if !scope.Search.Unscoped && hasDeletedAtField {
		sql := fmt.Sprintf("%v.%v IS NULL", quotedTableName, scope.Quote(deletedAtField.DBName))
		primaryConditions = append(primaryConditions, sql)
	}
	
	......
	
}
```

 一眼就看到了，贼尴尬的地方是写死的 `%v.%v IS NULL`，这里如果不能扩展的话就只能给修改唯一字段了，或者不要默认的软删除了。

#### 删除

![](../.gitbook/assets/image%20%282%29.png)

#### 查询

![](../.gitbook/assets/image%20%283%29.png)

#### 全局

![](../.gitbook/assets/image%20%285%29.png)

更新和创建也类似，这里就不一一列举了。

### 如何处理

在 gorm 项目的 issue 也有人提到 [https://github.com/go-gorm/gorm/issues/3184](https://github.com/go-gorm/gorm/issues/3184)，作者建议用 [Upsert](https://gorm.io/zh_CN/docs/create.html#Upsert-%E5%8F%8A%E5%86%B2%E7%AA%81) 去解决。

这个特性在 MySQL 官方文档：[https://dev.mysql.com/doc/refman/8.0/en/insert-on-duplicate.html](https://dev.mysql.com/doc/refman/8.0/en/insert-on-duplicate.html)

具体操作看我另一篇文章 [https://go.funnycode.cn/rep/gorm/gorm-v2/soft-delete-imporve](https://go.funnycode.cn/rep/gorm/gorm-v2/soft-delete-imporve)







