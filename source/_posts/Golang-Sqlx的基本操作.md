---
layout: sqlx-golang
title: Golang Sqlx的基本操作
date: 2018-12-11 23:00:00
categories: Golang的第三方库的使用
tags:
  - Golang sqlx
---

##sqlx使用指南 
   sqlx是一个go语言包，在内置database／sql包之上增加了很多扩展，简化数据库操作代码的书写。
   
### 安装与导入
```
o get "github.com/go-sql-driver/mysql"
go get "github.com/jmoiron/sqlx"

import (
    _ "github.com/go-sql-driver/mysql"
    "github.com/jmoiron/sqlx"
)
```

### 例子
  * 连接
  
```
import (
    _ "github.com/go-sql-driver/mysql"
    "github.com/jmoiron/sqlx"
)

var Db *sqlx.DB
func init() {
    database, err := sqlx.Open("mysql", "sample:sample@tcp(localhost:3306)/sample_dev?charset=utf8mb4")
    if err != nil {
            fmt.Println("open mysql failed,", err)
            return
        }
    
        Db = database
    }    
```

  * 建立表单
```
CREATE TABLE `person` (
  `user_id` int(128) DEFAULT NULL,
  `username` varchar(255) DEFAULT NULL,
  `sex` varchar(16) DEFAULT NULL,
  `email` varchar(128) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 
```

  *例子（insert)
```
package main

import (
    "fmt"

    _ "github.com/go-sql-driver/mysql"
    "github.com/jmoiron/sqlx"
)

type Person struct {
    UserId   int    `db:"user_id"`
    Username string `db:"username"`
    Sex      string `db:"sex"`
    Email    string `db:"email"`
}

func init() {
    database, err := sqlx.Open("mysql", "sample:sample@tcp(localhost:3306)/sample_dev?charset=utf8mb4")
    if err != nil {
        fmt.Println("open mysql failed,", err)
        return
    }
    Db = database
}


func main() {
      r, err := Db.Exec("insert into person(username, sex, email)values(?, ?, ?)", "suoning", "man", "suoning@net263.com")
      if err != nil {
        fmt.Println("exec failed, ", err)
                return
      }
      
      id, err := r.LastInsertId()
         if err != nil {
             fmt.Println("exec failed, ", err)
             return
         }
      
         fmt.Println("insert succ:", id)      
}

```

   * 例子　(update)
```
package main

import (
    "fmt"

    _ "github.com/go-sql-driver/mysql"
    "github.com/jmoiron/sqlx"
)

type Person struct {
    UserId   int    `db:"user_id"`
    Username string `db:"username"`
    Sex      string `db:"sex"`
    Email    string `db:"email"`
}

var Db *sqlx.DB

func init() {

    database, err := sqlx.Open("mysql", "sample:sample@tcp(localhost:3306)/sample_dev?charset=utf8mb4")
    if err != nil {
        fmt.Println("open mysql failed,", err)
        return
    }

    Db = database
}

func main() {

    _, err := Db.Exec("update person set user_id=? where username=?", 20170808, "suoning")
    if err != nil {
        fmt.Println("exec failed, ", err)
        return
    }

}
```

   *例子　(select)
```
package main

import (
    "fmt"

    _ "github.com/go-sql-driver/mysql"
    "github.com/jmoiron/sqlx"
)

type Person struct {
    UserId   int    `db:"user_id"`
    Username string `db:"username"`
    Sex      string `db:"sex"`
    Email    string `db:"email"`
}


var Db *sqlx.DB

func init() {

    database, err := sqlx.Open("mysql", "sample:sample@tcp(localhost:3306)/sample_dev?charset=utf8mb4")
    if err != nil {
        fmt.Println("open mysql failed,", err)
        return
    }

    Db = database
}

func main() {

    var person []Person
    err := Db.Select(&person, "select user_id, username, sex, email from person where user_id=?", 1)
    if err != nil {
        fmt.Println("exec failed, ", err)
        return
    }
    fmt.Println("select succ:", person)

    people := []Person{}
    Db.Select(&people, "SELECT * FROM person ORDER BY user_id ASC")
    fmt.Println(people)
    jason, john := people[0], people[1]
    fmt.Printf("%#v\n%#v", jason, john)
}
```

   * 例子　(delete)
   
```
package main

import (
    "fmt"

    _ "github.com/go-sql-driver/mysql"
    "github.com/jmoiron/sqlx"
)

type Person struct {
    UserId   int    `db:"user_id"`
    Username string `db:"username"`
    Sex      string `db:"sex"`
    Email    string `db:"email"`
}

var Db *sqlx.DB

func init() {

    database, err := sqlx.Open("mysql", "sample:sample@tcp(localhost:3306)/sample_dev?charset=utf8mb4")
    if err != nil {
        fmt.Println("open mysql failed,", err)
        return
    }

    Db = database
}

func main() {

    _, err := Db.Exec("delete from person where username=? limit 1", "suoning")
    if err != nil {
        fmt.Println("exec failed, ", err)
        return
    }

    fmt.Println("delete succ")
}
```

   * 事务
```
package main

import (
    "github.com/astaxie/beego/logs"
    _ "github.com/go-sql-driver/mysql"
    "github.com/jmoiron/sqlx"
)

var Db *sqlx.DB

func init() {
    database, err := sqlx.Open("mysql", "sample:sample@tcp(localhost:3306)/sample_dev?charset=utf8mb4")
    if err != nil {
        logs.Error("open mysql failed,", err)
        return
    }
    Db = database
}

func main()  {
    conn, err := Db.Begin()
    if err != nil {
        logs.Warn("DB.Begin failed, err:%v", err)
        return
    }

    defer func() {
        if err != nil {
            conn.Rollback()
            return
        }
        conn.Commit()
    }()

    // do something
}

```  
   