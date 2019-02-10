## sqlt的历史和由来

java的数据库访问工具MyBatis给大家留下了深刻的印象，早在几年前，我刚刚接触golang的时候，也希望golang也有类似的工具，对golang稍微熟悉后，发现golang自带模板功能（text/template），于是在另外一个开源库sqlx的基础上，增加模板拼接sql的功能，所以 sqlt 就诞生了。

## 安装

```
go get github.com/twiglab/sqlt
```

sqlt 也支持 go mod

## sqlt 架构简要说明

sqlt 深度依赖sqlx， 是在sqlx的基础上增加了模板功能，底层的数据库方法全部通过sqlx的NamedStmt和PrepareName完成对数据库的访问。

sqlt 对外提供的所有操作全部通过 `Dbop` struct 提供，Dbop struct 组合了sqlx.DB和模板（由Maker接口定义）

```go
type Dbop struct {
	Maker
	*sqlx.DB
}
```

- sqlt 没有隐藏任何使用sqlx的细节，Dbop对外直接暴露sqlx.DB，任何sqlt.DB的方法均可以直接使用，请参考sqlx的文档 
- sqlt 全部采用Parper和NamedStmt 完成对数据库的访问，所以也受到数据库驱动的限制，请详细参考数据库驱动的文档

目前sqlt自带的模板为`text/template`，任何Maker接口的实现都可以作为sqlt的模板使用。
```go
type Maker interface {
	MakeSql(string, interface{}) (string, error)
}
```
（*golang 自带的模板未必最好，欢迎pr更好模板实现*）

## 使用说明

### Dbop的创建
最简单的创建方式为：

```go
dbop := sqlt.Default("postgres", "dbname=testdb sslmode=disable", "tpl/*.tpl")
```
**注意：不要忘记引入数据库驱动**

如果你有现成的数据库链接，或者对模板有特殊的要求，有可以用使用`sqlt.New`方法创建

```go
dbx := sqlt.MustConnect("postgres", "dbname=testdb sslmode=disable")
tpl := sqlt.NewSqlTemplate("tpl/*.tpl")
tpl.SetDebug(true)

dbop := sqlt.New(dbx, tpl)
```

如果你的模板方法中用到了自定义的函数，sqlt也提供了一个 `NewSqlTemplateWithFuncs` 的方法用于创建带有自定义函数的模板 （位于`tpl.go`中）

### 模板

再次说明sqlt默认自带的模板是text/template的封装实现，详细的用法请参考text/template

(*example目录中有完整的例子*)

### sqlt的方法

最新版本中，所有的sqlt的方法都可以直接调用，分别为：
- func Query(execer TExecer, ctx context.Context, id string, param interface{}, h RowsExtractor) (err error) 
- func Exec(execer TExecer, ctx context.Context, id string, param interface{}) (r sql.Result, err error) 
- func ExecRtn(execer TExecer, ctx context.Context, id string, param interface{}, h RowsExtractor) (err error) 

以及对应的Must版本

Texcer 为 `*Dbop`，所有的方法都支持`context.Context`，id 为模板的id，param为传递给模板和用于Prepare的参数
