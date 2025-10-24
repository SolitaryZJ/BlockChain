# Gorm教程 - 补充教程

### Polymorphism

> Gorm支持一对一  一对多的多态

> ```
> type Dog struct {
>     ID   int
>     Name string
>     Toys []Toy `gorm:"polymorphic:Owner;"`
> }
> 
> type Cat struct {
> 	ID   int
>     Name string
>     Toys []Toy `gorm:"polymorphic:Owner;"`
> }
> 
> type Toy struct {
>     ID        int
>     Name      string
>     OwnerID   int
>     OwnerType string
> }
> 
> ```

> - polymorphic: 同时指定type和ID（value+Type value+ID）
> - polymorphicType: 单独指定type
> - polymorphicId: 单独指定ID
> - polymorphicValue: 指定type的值，默认为表名

### Session

> GORM 提供了 `Session` 方法，这是一个 [`New Session Method`](https://gorm.io/zh_CN/docs/method_chaining.html)，它允许创建带配置的新建会话模式：
>

> ```
> // Session 配置
> type Session struct {
>     DryRun                   bool
>     PrepareStmt              bool
>     NewDB                    bool
>     Initialized              bool
>     SkipHooks                bool
>     SkipDefaultTransaction   bool
>     DisableNestedTransaction bool
>     AllowGlobalUpdate        bool
>     FullSaveAssociations     bool
>     QueryFields              bool
>     Context                  context.Context
>     Logger                   logger.Interface
>     NowFunc                  func() time.Time
>     CreateBatchSize          int
> }
> ```

### Hook

> 前面的课程其实已经讲过了HOOK，这里讲一下继承的时候HOOK的调用机制

### 事务

> 为了确保数据一致性，GORM 会在事务里执行写入操作（创建、更新、删除）。如果没有这方面的要求，您可以在初始化时禁用它，这将获得大约 30%+ 性能提升。
>
> ```
> // 全局禁用
> db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
>   	SkipDefaultTransaction: true,
> })
> 
> // 持续会话模式
> tx := db.Session(&Session{SkipDefaultTransaction: true})
> tx.First(&user, 1)
> tx.Find(&users)
> tx.Model(&user).Update("Age", 18)
> 
> 
> db.Transaction(func(tx *gorm.DB) error {
>     // 在事务中执行一些 db 操作（从这里开始，您应该使用 'tx' 而不是 'db'）
>     if err := tx.Create(&Animal{Name: "Giraffe"}).Error; err != nil {
>       // 返回任何错误都会回滚事务
>       return err
>     }
> 
>     if err := tx.Create(&Animal{Name: "Lion"}).Error; err != nil {
>       return err
>     }
> 
>     // 返回 nil 提交事务
>     return nil
> })
> 
> 
> // 嵌套事务
> db.Transaction(func(tx *gorm.DB) error {
>   tx.Create(&user1)
> 
>   tx.Transaction(func(tx2 *gorm.DB) error {
>     tx2.Create(&user2)
>     return errors.New("rollback user2") // Rollback user2
>   })
> 
>   tx.Transaction(func(tx3 *gorm.DB) error {
>     tx3.Create(&user3)
>     return nil
>   })
> 
>   return nil
> })
> 
> 
> // 手动事务
> // 开始事务
> tx := db.Begin()
> 
> // 在事务中执行一些 db 操作（从这里开始，您应该使用 'tx' 而不是 'db'）
> tx.Create(...)
> 
> // ...
> 
> // 遇到错误时回滚事务
> tx.Rollback()
> 
> // 否则，提交事务
> tx.Commit()
> 
> 
> 
> tx := db.Begin()
> tx.Create(&user1)
> 
> tx.SavePoint("sp1")
> tx.Create(&user2)
> tx.RollbackTo("sp1") // Rollback user2
> 
> tx.Commit() // Commit user1
> 
> ```



### 自定义数据类型

> ```
> type UserD struct {
>     ID     int
>     Skills strArr
> }
> 
> func (arr strArr) Value() (driver.Value, error) {
>     return strings.Join(arr, ","), nil
> }
> 
> func (arr *strArr) Scan(value interface{}) error {
>       bytes, ok := value.([]byte)
>       if !ok {
>          return errors.New(fmt.Sprint("Failed to unmarshal JSONB value:", value))
>       }
> 
>       *arr = strings.Split(string(bytes), ",")
>       return nil
> }
> ```
>
> 有许多第三方包实现了 `Scanner`/`Valuer` 接口，可与 GORM 一起使用，例如：
>
> ```
> import (
>     "github.com/google/uuid"
>     "github.com/lib/pq"
> )
> 
> type Post struct {
>     ID     uuid.UUID `gorm:"type:uuid;default:uuid_generate_v4()"`
>     Title  string
>     Tags   pq.StringArray `gorm:"type:text[]"`
> }
> ```

### 性能

> GORM 已经优化了许多东西来提高性能，其默认性能对大多数应用来说都够用了。但这里还是有一些关于如何为您的应用改进性能的方法。
>
> - 禁用默认事务
>
> - 缓存预编译语句
    >
    >   - ```
>     // 全局模式
>     db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
>       PrepareStmt: true,
>     })
>     
>     // 会话模式
>     tx := db.Session(&Session{PrepareStmt: true})
>     tx.First(&user, 1)
>     tx.Find(&users)
>     tx.Model(&user).Update("Age", 18)
>     ```
>
> - 选择字段
    >
    >   - ```
>     db.Select("Name", "Age").Find(&Users{})
>     ```
>
> - 自主选择索引
    >
    >   - ```
>     import "gorm.io/hints"
>     
>     db.Clauses(hints.UseIndex("idx_user_name")).Find(&User{})
>     ```
>
> - 读写分离

### Scopes

> 作用域允许你复用通用的逻辑，这种共享逻辑需要定义为类型`func(*gorm.DB) *gorm.DB`。
>
> ```
> func AmountGreaterThan1000(db *gorm.DB) *gorm.DB {
>   return db.Where("amount > ?", 1000)
> }
> 
> func PaidWithCreditCard(db *gorm.DB) *gorm.DB {
>   return db.Where("pay_mode_sign = ?", "C")
> }
> 
> func PaidWithCod(db *gorm.DB) *gorm.DB {
>   return db.Where("pay_mode_sign = ?", "C")
> }
> 
> func OrderStatus(status []string) func (db *gorm.DB) *gorm.DB {
>   return func (db *gorm.DB) *gorm.DB {
>     return db.Where("status IN (?)", status)
>   }
> }
> 
> db.Scopes(AmountGreaterThan1000, PaidWithCreditCard).Find(&orders)
> ```
>
> ```
> // 分页
> func Paginate(r *http.Request) func(db *gorm.DB) *gorm.DB {
>   return func (db *gorm.DB) *gorm.DB {
>     q := r.URL.Query()
>     page, _ := strconv.Atoi(q.Get("page"))
>     if page <= 0 {
>       page = 1
>     }
> 
>     pageSize, _ := strconv.Atoi(q.Get("page_size"))
>     switch {
>     case pageSize > 100:
>       pageSize = 100
>     case pageSize <= 0:
>       pageSize = 10
>     }
> 
>     offset := (page - 1) * pageSize
>     return db.Offset(offset).Limit(pageSize)
>   }
> }
> 
> db.Scopes(Paginate(r)).Find(&users)
> db.Scopes(Paginate(r)).Find(&articles)
> ```
>
> ```
> // 指定表
> func TableOfYear(user *User, year int) func(db *gorm.DB) *gorm.DB {
>   return func(db *gorm.DB) *gorm.DB {
>         tableName := user.TableName() + strconv.Itoa(year)
>         return db.Table(tableName)
>   }
> }
> 
> DB.Scopes(TableOfYear(user, 2019)).Find(&users)
> // SELECT * FROM users_2019;
> 
> DB.Scopes(TableOfYear(user, 2020)).Find(&users)
> // SELECT * FROM users_2020;
> 
> // Table form different database
> func TableOfOrg(user *User, dbName string) func(db *gorm.DB) *gorm.DB {
>   return func(db *gorm.DB) *gorm.DB {
>         tableName := dbName + "." + user.TableName()
>         return db.Table(tableName)
>   }
> }
> 
> DB.Scopes(TableOfOrg(user, "org1")).Find(&users)
> // SELECT * FROM org1.users;
> 
> DB.Scopes(TableOfOrg(user, "org2")).Find(&users)
> // SELECT * FROM org2.users;
> ```
>
> ```
> // 动态条件
> func CurOrganization(r *http.Request) func(db *gorm.DB) *gorm.DB {
>   return func (db *gorm.DB) *gorm.DB {
>     org := r.Query("org")
> 
>     if org != "" {
>       var organization Organization
>       if db.Session(&Session{}).First(&organization, "name = ?", org).Error == nil {
>         return db.Where("org_id = ?", organization.ID)
>       }
>     }
> 
>     db.AddError("invalid organization")
>     return db
>   }
> }
> 
> db.Model(&article).Scopes(CurOrganization(r)).Update("Name", "name 1")
> // UPDATE articles SET name = "name 1" WHERE org_id = 111
> db.Scopes(CurOrganization(r)).Delete(&Article{})
> // DELETE FROM articles WHERE org_id = 111
> 
> ```
>
>

### 设置

> GORM 提供了 `Set`, `Get`, `InstanceSet`, `InstanceGet` 方法来允许用户传值给 [勾子](https://gorm.io/zh_CN/docs/hooks.html) 或其他方法
>
> ```
> type User struct {
>   gorm.Model
>   CreditCard CreditCard
>   // ...
> }
> 
> func (u *User) BeforeCreate(tx *gorm.DB) error {
>   myValue, ok := tx.Get("my_value")
>   // ok => true
>   // myValue => 123
> }
> 
> type CreditCard struct {
>   gorm.Model
>   // ...
> }
> 
> func (card *CreditCard) BeforeCreate(tx *gorm.DB) error {
>   myValue, ok := tx.Get("my_value")
>   // ok => true
>   // myValue => 123
> }
> 
> myValue := 123
> db.Set("my_value", myValue).Create(&User{})
> ```
>
> ```
> type User struct {
>   gorm.Model
>   CreditCard CreditCard
>   // ...
> }
> 
> func (u *User) BeforeCreate(tx *gorm.DB) error {
>   myValue, ok := tx.InstanceGet("my_value")
>   // ok => true
>   // myValue => 123
> }
> 
> type CreditCard struct {
>   gorm.Model
>   // ...
> }
> 
> // 在创建关联时，GORM 创建了一个新 `*Statement`，所以它不能读取到其它实例的设置
> func (card *CreditCard) BeforeCreate(tx *gorm.DB) error {
>   myValue, ok := tx.InstanceGet("my_value")
>   // ok => false
>   // myValue => nil
> }
> 
> myValue := 123
> db.InstanceSet("my_value", myValue).Create(&User{})
> ```
>
> 