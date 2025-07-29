+++
title = "Kotlin 資料存取設計：Unit of Work 與 Repository 模式實戰"
description = "深入解析 Unit of Work 與 Repository Pattern 的核心概念與協同運作，並以 Kotlin ORM 框架實作為例。帶你了解如何打造高一致性、易維護的資料存取層。適合想提升系統架構設計的開發者。"
date = "2025-05-25T10:21:35Z"
updated = "2025-07-28T04:22:20.194Z"
draft = false

[taxonomies]
tags = [ "Prompt Engineering", "Design Pattern" ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/YyHPp6unwvsnaVwkFsWb9K?invite=dOLYGeJyZJqVX>"
+++
{% editor_note() %}
其實在寫 Kotlin 的不是我，是另一位認識的網友。  
我幫他 code review 之後發現 repository 沒有一起實作 UoW。

我的問句是一種 *使用 AI 來產生詳細指引* 的提問法，當資深人員在產出指南給新人時非常適合這種提示詞技巧。
{% end %}

{% chat(speaker="jim") %}
解釋 Unit of Work 的意義，和 Repository Pattern 為什麼會一起使用。並說明使用 kotlin 推薦的 ORM 如何實作它們。
{% end %}

{% chat(speaker="felo") %}
本報告旨在闡釋 Unit of Work (UoW) 模式的意義，以及其與 Repository 模式協同使用的原因。同時，將說明如何運用 Kotlin 語言中推薦的物件關聯對應 (ORM) 框架來實作這兩種模式。
{% end %}

Repository 模式主要用於抽象化資料存取邏輯，提供一個中介層來隔離應用程式的業務邏輯與資料庫互動的細節 [^74][^84]。Unit of Work 模式則專注於管理單一業務交易中的所有資料變更操作，確保這些操作要麼全部成功提交，要麼在發生錯誤時全部回滾，從而維護資料的一致性與完整性 [^14][^45]。這兩種模式結合使用，能夠建立一個職責分明、易於測試且更具彈性的資料存取層 [^11][^107]。

## Repository 模式 (Repository Pattern)

**定義與目的**
Repository 模式是一種軟體設計模式，其核心思想是在應用程式的領域模型（或業務邏輯）與資料對應層（資料存取層）之間建立一個中介層 [^84][^112]。它扮演著類似記憶體中領域物件集合的角色，封裝了存取資料來源所需的邏輯 [^1][^112]。透過 Repository，應用程式的其餘部分可以透過一個清晰定義的介面來執行資料操作（如新增、讀取、更新、刪除，即 CRUD），而無需關心底層資料庫的具體實作細節，例如是 SQL 資料庫、NoSQL 資料庫還是遠端 API [^1][^4]。每個 Repository 通常對應一個聚合根 (Aggregate Root)，這是領域驅動設計 (DDD) 中的一個重要概念，確保了交易的一致性 [^112]。

**優點**
採用 Repository 模式能帶來多項益處：

* **關注點分離 (Separation of Concerns)**：清晰地將資料存取邏輯與業務邏輯分開，使得程式碼結構更清晰，更易於理解與維護 [^1][^9]。
* **提升可測試性 (Improved Testability)**：由於資料存取邏輯被隔離，可以輕易地使用模擬 (mock) 的 Repository 進行單元測試，而無需實際連接資料庫 [^1][^112]。
* **彈性與可維護性 (Flexibility and Maintainability)**：Repository 提供了一個穩定的資料存取介面。如果需要更換底層資料來源（例如從本地資料庫切換到雲端資料庫，或更換 ORM 框架），只需修改 Repository 的實作，而不會影響到應用程式的其他部分 [^1]。
* **減少重複程式碼 (Reduced Code Duplication)**：將通用的資料存取功能集中管理，避免在多處撰寫相似的資料庫操作程式碼 [^9][^81]。

**Kotlin 中的基本實作概念**
在 Kotlin 中實作 Repository 模式通常涉及以下步驟 [^1][^4]：

1. **定義資料類別 (Data Class)**：表示要操作的資料實體。

    ```kotlin
    data class User(val id: Long, val name: String, val email: String)
    ```

2. **定義 Repository 介面 (Repository Interface)**：宣告資料存取操作的方法。

    ```kotlin
    interface UserRepository {
        suspend fun getUserById(id: Long): User?
        suspend fun getAllUsers(): List<User>
        suspend fun addUser(user: User)
        suspend fun updateUser(user: User)
        suspend fun deleteUser(userId: Long)
    }
    ```

3. **實作 Repository 介面 (Repository Implementation)**：建立一個具體類別來實作介面中定義的方法，這個類別將使用 ORM 框架或其他方式與實際資料來源互動。

    ```kotlin
    // 具體實作會依賴選擇的 ORM 框架，例如 Exposed、Ktorm 或 Room
    class UserRepositoryImpl(/* ...dependencies like DataSource or DAO... */) : UserRepository {
        // override suspend fun getUserById(id: Long): User? { /* ... */ }
        // ... 其他方法的實作
    }
    ```

## Unit of Work 模式 (Unit of Work Pattern)

**定義與目的**
Unit of Work (UoW) 模式的核心職責是追蹤在一個業務交易 (business transaction) 過程中所有受影響的物件（新增、修改、刪除），並協調這些變更的寫入以及解決並行問題 [^45][^66]。它確保所有這些操作被視為一個不可分割的單元：要麼全部成功執行並提交到資料庫，要麼在任何一個操作失敗時全部回滾，從而保持資料庫的一致性和完整性 [^14][^17][^57]。可以將 UoW 想像成一個業務交易的容器，它收集了所有需要執行的資料庫操作，然後一次性地應用它們 [^13]。

**優點**
使用 Unit of Work 模式的主要優點包括：

* **確保資料完整性 (Data Integrity)**：透過將多個操作綑綁在單一交易中，UoW 防止了部分更新導致的資料不一致問題 [^14][^67]。
* **減少資料庫呼叫 (Reduced Database Calls)**：變更會先在記憶體中被追蹤，直到最後才統一提交，這有助於最小化與資料庫的往返次數，尤其是在進行批次操作時能提升效能 [^14][^60]。
* **簡化交易管理 (Simplified Transaction Management)**：開發者可以專注於業務邏輯，而不必在多個地方手動管理交易的開始、提交和回滾 [^54]。
* **提升可測試性 (Enhanced Testability)**：與 Repository 模式結合時，可以更容易地測試包含多個資料操作的業務邏輯單元 [^14]。

## Repository 與 Unit of Work 模式的協同作業 (Synergy of Repository and Unit of Work Patterns)

**為何結合使用**
Repository 模式和 Unit of Work 模式經常一起使用，因為它們共同構成了一個強大的資料存取層抽象 [^11][^146]。Repository 負責封裝單個實體或聚合根的資料存取邏輯（即 *如何* 進行 CRUD 操作），而 Unit of Work 則負責協調跨越多個 Repository 或多個操作的交易（即 *何時* 將這些操作作為一個整體提交或回滾）[^107][^113]。

**運作方式**
在一個典型的應用場景中：

1. 應用程式的服務層 (Service Layer) 或業務邏輯層會使用一個或多個 Repository 來執行資料操作（例如，新增一個使用者，然後為該使用者新增一個訂單）[^67]。
2. 這些 Repository 的操作（如 `addUser`, `saveOrder`）並不會立即寫入資料庫，而是將變更的實體註冊到 Unit of Work 實例中 [^67][^113]。例如，`UserRepository.add(newUser)` 會通知 UoW 有一個新的 `User` 物件需要被持久化。
3. 當所有業務操作完成後，服務層會呼叫 Unit of Work 的 `commit()` 方法 [^67]。
4. Unit of Work 接著會以原子方式執行所有已註冊的變更。如果所有操作都成功，交易便提交；若有任何失敗，則整個交易回滾，資料庫狀態恢復到操作開始前的樣子 [^14][^67]。

許多 ORM 框架（如 Entity Framework）的 `DbContext` 或 Session 物件本身就隱含了 Unit of Work 的功能，例如 EF 中的 `SaveChanges()` 方法就扮演了 UoW 中 `commit()` 的角色 [^71][^112]。

{% editor_note() %}
Entity Framework 是 C# .NET 的 ORM，Kotlin 小夥伴請略過它。
{% end %}

## 使用 Kotlin 和 ORM 實作 (Implementation with Kotlin and ORM)

在 Kotlin 中，可以選用多種 ORM 框架來實作 Repository 和 Unit of Work 模式。

### 推薦的 Kotlin ORM 框架

* **Exposed**: 由 JetBrains 開發的輕量級 SQL 框架，可以作為 SQL DSL 或輕量級 ORM 使用 [^30][^153]。它提供了強大的交易管理功能 [^153]。
* **Ktorm**: 一個基於純 JDBC 的輕量級 ORM 框架，提供型別安全的 SQL DSL 和方便的序列 API [^6][^118][^152]。
* **Room**: Android Jetpack 的一部分，為 Android 應用程式提供 SQLite 資料庫的抽象層，簡化資料庫操作 [^2][^33][^117]。DAOs (Data Access Objects) 在 Room 中常扮演 Repository 的角色。
* **Hibernate/JPA**: 傳統的 Java ORM 框架，也可以在 Kotlin 中使用，例如透過 Quarkus 的 Panache 擴充功能簡化 Hibernate 的使用 [^24][^125]。

### 實作範例概念

**Repository 實作 (使用 Exposed 概念)**
首先，定義 Exposed 的 Table 物件和對應的資料類別：

```kotlin
import org.jetbrains.exposed.sql.*
import org.jetbrains.exposed.sql.transactions.transaction
import org.jetbrains.exposed.dao.id.LongIdTable

// 資料類別
data class Product(val id: Long, val name: String, val price: Double)

// Exposed Table 定義
object ProductsTable : LongIdTable("products") {
    val name = varchar("name", 255)
    val price = double("price")
}

// Repository 介面
interface ProductRepository {
    fun findById(id: Long): Product?
    fun findAll(): List<Product>
    fun save(product: Product): Product
    fun delete(id: Long)
}

// Exposed Repository 實作
class ExposedProductRepository(private val db: Database) : ProductRepository {
    init {
        transaction(db) {
            SchemaUtils.create(ProductsTable) // 確保資料表存在
        }
    }

    private fun toProduct(row: ResultRow): Product =
        Product(
            id = row[ProductsTable.id].value,
            name = row[ProductsTable.name],
            price = row[ProductsTable.price]
        )

    override fun findById(id: Long): Product? = transaction(db) {
        ProductsTable.select { ProductsTable.id eq id }
            .map(::toProduct)
            .singleOrNull()
    }

    override fun findAll(): List<Product> = transaction(db) {
        ProductsTable.selectAll().map(::toProduct)
    }

    override fun save(product: Product): Product = transaction(db) {
        if (product.id == 0L) { // 假設 id 0L 代表新產品
            val newId = ProductsTable.insertAndGetId {
                it[name] = product.name
                it[price] = product.price
            }
            product.copy(id = newId.value)
        } else {
            ProductsTable.update({ ProductsTable.id eq product.id }) {
                it[name] = product.name
                it[price] = product.price
            }
            product
        }
    }

    override fun delete(id: Long) {
        transaction(db) {
            ProductsTable.deleteWhere { ProductsTable.id eq id }
        }
    }
}
```

**Unit of Work 實作 (使用 Exposed 概念)**
Exposed 的 `transaction { ... }` 區塊本身就提供了交易的原子性，可以視為一個 Unit of Work 的執行單位 [^153]。如果需要一個更明確的 UoW 物件來管理交易，特別是當服務層需要協調多個 Repository 操作時，可以這樣設計：

```kotlin
interface AppUnitOfWork {
    fun <T> execute(action: Transaction.() -> T): T
}

class ExposedAppUnitOfWork(private val db: Database) : AppUnitOfWork {
    override fun <T> execute(action: Transaction.() -> T): T {
        return transaction(db) {
            // 可以在此處加入通用的交易設定，例如日誌記錄
            // addLogger(StdOutSqlLogger)
            action()
            // 交易成功則自動 commit，發生例外則自動 rollback
        }
    }
}

// 服務層使用 UoW 和 Repository
class ProductService(
    private val productRepository: ProductRepository,
    private val unitOfWork: AppUnitOfWork
    // 可能還有其他 Repositories
) {
    fun createProductAndUpdateStock(productName: String, productPrice: Double, stockChange: Int) {
        unitOfWork.execute { // 所有在此區塊內的操作都在同一個交易中
            val newProduct = productRepository.save(Product(0L, productName, productPrice))
            // 假設有 StockRepository
            // val stockRepository = ExposedStockRepository(this.db) // 'this.db' 來自 Transaction
            // stockRepository.updateStock(newProduct.id, stockChange)

            // 如果任何操作失敗，整個 execute 區塊會回滾
            println("Product ${newProduct.name} created and stock updated within a transaction.")
        }
    }
}
```

在這個範例中，`ExposedAppUnitOfWork` 的 `execute` 方法包裹了 Exposed 的 `transaction` 函數，確保傳入的 `action` 區塊內的所有資料庫操作都在單一交易中執行。Repository 的方法（如 `save`）在被 `execute` 呼叫時，會自動參與到這個由 UoW 管理的交易中。

**使用 Ktorm 的概念**
Ktorm 的 Repository 實作會使用其 `Database` 物件和序列 API (sequence API) [^118][^152]。對於 Unit of Work，Ktorm 依賴於底層的 JDBC 交易管理。開發者需要手動控制 JDBC `Connection` 的 `autoCommit` 屬性，並在操作完成後呼叫 `commit()` 或在發生錯誤時呼叫 `rollback()`。一個 UoW 封裝類可以簡化這個過程：

```kotlin
// Ktorm Entity 和 Table (簡化)
interface KProduct : Entity<KProduct> { /* ... */ }
object KProductsTable : Table<KProduct>("products") { /* ... */ }
val Database.products get() = this.sequenceOf(KProductsTable)


// Ktorm Repository 介面 (同上)
// class KtormProductRepository(private val db: Database) : ProductRepository { /* ... */ }

// Ktorm Unit of Work 概念
interface KtormAppUnitOfWork {
    fun <T> execute(block: (Database) -> T): T
}

class DefaultKtormUnitOfWork(private val dataSource: javax.sql.DataSource) : KtormAppUnitOfWork {
    override fun <T> execute(block: (Database) -> T): T {
        dataSource.connection.use { connection ->
            connection.autoCommit = false // 開始交易
            try {
                val db = Database.connect(connection) // Ktorm Database 使用此連線
                val result = block(db)
                connection.commit() // 提交交易
                return result
            } catch (e: Exception) {
                connection.rollback() // 回滾交易
                throw e
            }
        }
    }
}
```

服務層在使用 `KtormAppUnitOfWork` 時，會將需要交易性執行的 Repository 操作放在 `execute` 的 lambda 區塊中，並將傳入的 `Database` 實例傳遞給 Repository。

總結來說，Repository 模式提供了資料存取的抽象，而 Unit of Work 模式確保了這些存取操作的交易一致性。在 Kotlin 中，Exposed 和 Ktorm 等 ORM 框架都提供了實作這兩種模式的機制，Exposed 的 `transaction` 區塊使其更為簡潔，而 Ktorm 則更依賴手動的 JDBC 交易管理，但兩者都能有效地幫助開發者建立穩健且可維護的資料存取層。選擇哪種 ORM 取決於專案的具體需求、複雜性以及開發團隊的偏好。

[^1]: [Repository Design Pattern in kotlin | by App Dev Insights](https://medium.com/@appdevinsights/repository-design-pattern-in-kotlin-1d1aeff1ad40)
[^2]: [Implementing Room Databases with Kotlin Coroutines in ...](https://mrappbuilder.medium.com/implementing-room-databases-with-kotlin-coroutines-in-android-e1b5e11e1578)
[^4]: [Repository Design Pattern - Medium](https://medium.com/@ahmetbostanciklioglu/repository-pattern-in-kotlin-a94194ba910e)
[^6]: [kotlin-orm/ktorm - GitHub](https://github.com/kotlin-orm/ktorm)
[^9]: [Unit of Work With Generic Repository - DZone](https://dzone.com/articles/unit-of-work-with-generic-repository-implementatio)
[^11]: [Implementing the Unit of Work and Repository Pattern](https://dev.to/shanenullain/decoupling-python-code-implementing-the-unit-of-work-and-repository-pattern-43kf)
[^13]: [How can I better understand the concepts of unit of work and ...](https://www.reddit.com/r/dotnet/comments/119oi7j/how_can_i_better_understand_the_concepts_of_unit/)
[^14]: [Unit Of Work Pattern - Medium](https://medium.com/@arjunarora171997/unit-of-work-pattern-290cd46ffcf4)
[^17]: [Unit of Work Pattern in Java: Orchestrating Efficient ...](https://java-design-patterns.com/patterns/unit-of-work/)
[^24]: [Simplified Hibernate ORM with Panache and Kotlin - Quarkus](https://quarkus.io/guides/hibernate-orm-panache-kotlin)
[^30]: [Guide to the Kotlin Exposed Framework - Baeldung](https://www.baeldung.com/kotlin/exposed-persistence)
[^33]: [Persist data with Room - Android Developers](https://developer.android.com/codelabs/basic-android-kotlin-compose-persisting-data-room)
[^45]: [Unit of Work - Martin Fowler](https://martinfowler.com/eaaCatalog/unitOfWork.html)
[^54]: [A Deep Dive into the Unit of Work Pattern: Pros and Cons](https://www.linkedin.com/pulse/deep-dive-unit-work-pattern-pros-cons-george-gyftos-3qpjf)
[^57]: [Demystifying the Unit of Work Design Pattern - Medium](https://medium.com/@sohail.aslam9987/demystifying-the-unit-of-work-design-pattern-a-comprehensive-guide-6eee92231558)
[^60]: [Unit Of Work | Java Design Patterns (中文)](https://java-design-patterns.com/zh/patterns/unit-of-work/)
[^66]: [Unit of Work](https://martinfowler.com/eaaCatalog/unitOfWork.html)
[^67]: [Thinking In Design Pattern——Unit Of Work（工作单元）模式探索-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1018155)
[^71]: [Designing the infrastructure persistence layer - .NET](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)
[^74]: [Repository Design Pattern | GeeksforGeeks](https://www.geeksforgeeks.org/repository-design-pattern/)
[^81]: [Repository Pattern and Unit of Work with Asp.Net Core 5](https://medium.com/c-sharp-programming/repository-pattern-and-unit-of-work-with-asp-net-core-5-60f5df3e9dea)
[^84]: [Repository - Martin Fowler](https://martinfowler.com/eaaCatalog/repository.html)
[^107]: [Unit of Work Pattern - Cosmic Python](https://www.cosmicpython.com/book/chapter_06_uow.html)
[^112]: [Designing the infrastructure persistence layer - .NET | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)
[^113]: [掀起你的盖头来：Unit Of Work-工作单元 - 田园里的蟋蟀 - 博客园](https://www.cnblogs.com/xishuai/p/3750154.html)
[^117]: [Implementing Room Databases with Kotlin Coroutines in ...](https://mrappbuilder.medium.com/implementing-room-databases-with-kotlin-coroutines-in-android-e1b5e11e1578)
[^118]: [kotlin-orm/ktorm - GitHub](https://github.com/kotlin-orm/ktorm)
[^125]: [Simplified Hibernate ORM with Panache - Quarkus](https://quarkus.io/guides/hibernate-orm-panache)
[^146]: [Implementing the Unit of Work and Repository Pattern](https://dev.to/shanenullain/decoupling-python-code-implementing-the-unit-of-work-and-repository-pattern-43kf)
[^152]: [GitHub - kotlin-orm/ktorm: A lightweight ORM framework for Kotlin with strong-typed SQL DSL and sequence APIs.](https://github.com/kotlin-orm/ktorm)
[^153]: [Guide to the Kotlin Exposed Framework | Baeldung on Kotlin](https://www.baeldung.com/kotlin/exposed-persistence)
