---
layout: post
title: 'Entity Framework Code First Migration'
date: 2013-03-12 07:04:00
categories: Database
---

本練習將介紹 Entity Framework Code First Migration 整個使用的概觀:
主題包含:

+   啟動 Migrations.
+   建立與執行 Migrations.
+   自訂 Migrations.
+   資料遷移和使用SQL指令.
+   遷移指定的版本.
+   建立 SQL Script.
+   程式啟動時自動更新資料結構.

<!-- more -->
####建置初始化Model和資料庫
----

在我們開始介紹如何使用資料庫遷移(Migrations)之前我們需要一個專案和 Model。在這個練習中我們將模擬一個 Blog 建立 Blog 和 Post 的 Model。

1. 新增主控台應用程式 (Console Project) 並命名為 MigrationsDemo。
2. 透過 Nuget 安裝最新版的 EntityFramewrok。
2-1. 工具->程式庫套件管理員->套件管理員主控台。
2-2. 執行 Install-Package EntityFramework 指令。
2-3. 加入一個 Model.cs 類別程式碼如下:

{% highlight csharp %}
using System.Data.Entity;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Data.Entity.Infrastructure;
namespace MigrationsDemo
{
    public class BlogContext:DbContext
    {
        public DbSet<Blog> Blogs { get; set; }
    }
    public class Blog
    {
        public int BlogId { get; set; }
        public string Name { get; set; }
    }
}
{% endhighlight %}

上面的程式碼定義了 Blog 類別和一個 BlogContext 類別 (BlogContext 類別繼承了 DbContext，而 DbContext，而 類別是簡化過的 Context 類別官方建議在Code first 模式下使用。)現在我們有了 Model 可以用它來執行資料存取，接著更新 Program.cs 程式碼如下:

{% highlight csharp %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace MigrationsDemo
{
    class Program
    {
        static void Main(string[] args)
        {
            using (var db = new BlogContext())
            {
                db.Blogs.Add(new Blog { Name="Another Blog"});
                db.SaveChanges();

                foreach (var blog in db.Blogs)
                {
                    Console.WriteLine(blog.Name);
                }

                Console.WriteLine("Press any key to exit...");
                Console.ReadKey();
            }
        }
    }
}

{% endhighlight %}

執行應用程式您將會看到 MigrationsDemo.BlogContext 被建立了。您可能會疑惑此時我們完全沒有設定任何相關的連線字串也沒有指定任何資料庫。
如果您練習的電腦有安裝 SQL Express 則預設 Code First 機制會先搜尋本機的 .\SQLEXPRESS。如果您沒有安裝 SQL Express 則 Code First 會嘗試使用 
(LocalDb)\v11.0 來建立資料庫。
一般來說 Visual Studio 2010 預設安裝會包含 SQL Express。而 Visual Studio 2012 則預設安裝包含 LocalDb。注意如果您兩個都有安裝的話 SQL Express 會優先使用。

#### 啟動 Migrations
---

在 Code First 模式下 Migrations 資料庫遷移指的是一種機制讓我們可以修改資料庫，資料表結構。

接著編輯 Model，在 Blog類別中加入一個 Url 的屬性(Property)。

{% highlight csharp %}
public string Url{ get; set; }
{% endhighlight %}

完成之後讓我們直接運行程式(F5)。這個時候您會得到一個 InvalidOperationException 例外。意思是 Model 已經變更了，因此無法跟已存在的資料庫對應。

此時例外的提示資訊會建議我們啟動 Code First Migrations 接著我們就在【套件管理員主控台】執行:

{% highlight sh %}
Enable-Migrations
{% endhighlight %}

這個指令會加入一個 Migrations 目錄。裡面會新增兩個檔案:

*設定類別 (Configuration.cs)
這個類別可以讓您自訂一些資料庫遷移時的行為，意思就是當我們要變更資料結構時可以追加一些設定。
例如: 使用 Seed() 在建立資料表時幫您補上預設的資料，或者幫欄位設定預設值。

*資料遷移檔 ([時間戳記]_InitialCreate.cs)
主要的資料遷移檔案是一個關於建立資料表，修改資料結構的 Script 。因為我們已經利用 Code First 幫我們建立了一個資料庫，所以這個
遷移檔就會根據目前的資料庫結構產生一個對應的遷移檔。在我們這個範例中您會看到:

{% highlight csharp %}
CreateTable(
    "dbo.Blogs",
    c => new
    {
        BlogId = c.Int(nullable: false, identity: true),
        Name = c.String(),
    })
    .PrimaryKey(t => t.BlogId);
{% endhighlight %}

很明顯的我們看到程式建立了一張資料表 Blog 只包含 BlogId 和 Name 欄位。如果此時我們都沒有建立資料庫，這個遷移檔就不會被建立，直到我們第一次呼叫<code> Add-Migration</code> 指令才會產生。另外當我們要做一次結構變更時就是使用:

{% highlight sh %}
Add-Migration [name]
{% endhighlight %}

####建立與執行 Migrations
---

Code First 的 Migrations 機制主要有兩個指令。這兩個指令很重要必須要熟悉它們。

* Add-Migration 
  會基於您這次對 Model 做得變更產生出新的資料遷移檔 (Migration)
* Update-Database
  就會把關於 Migration 和 Configuration 檔的內容對資料庫產生實際的變動。
  
現在我們需要對我們已經變更的 Model 產生一個 Migration 檔

1. 執行下面的指令

{% highlight sh %}
Add-Migration AddBlogUrl
{% endhighlight %}

2. 在 Migraions 目錄底下我們看到了一個新增的遷移檔。看看內容

{% highlight csharp %}
public override void Up()
{
    AddColumn("dbo.Blogs", "Url", c => c.String());
}
{% endhighlight %}

大概就可以知道資料遷移檔幫我們對 Table 新增了欄位。
接著就可以執行:

{% highlight sh %}
Update-Database
{% endhighlight %}

Code First Migrations 機制就會幫我們去比對每一個資料庫遷移檔然後對資料庫做修改。

####自訂 Migrations
----

目前為止，我們建立並執行了 Migration 但我們並沒有改變任何關於遷移檔的設定。
讓我們來看看如何修改預設產生的遷移檔。

1. 對 Model 修改，增加一個 Rating 屬性:

{% highlight csharp %}
public int Rating { get; set; }
{% endhighlight %}

2. 接著再新增一個 Post 類別:

{% highlight csharp %}
 public class Post
    {
        public int PostId { get; set; }
        [MaxLength(200)]
        public string Title { get; set; }
        public string Content { get; set; }

        public int BlogId { get; set; }
        public Blog Blog { get; set; }
    }
{% endhighlight %}
 
3. 建立完成後我們再對 Blog 類別增加:

{% highlight csharp %}
public virtual List<Post> Posts { get; set; }
{% endhighlight %}

再次使用 Add-Migration 來替我們產生遷移檔執行

{% highlight sh %}
Add-Migration AddPostClass
{% endhighlight %}

Code Fist Migrations 機制很盡責地幫我們對應結構產生了 Migration 檔，但在這邊我們想要作一些改變。
1. Posts.Title 欄位不得重複(unique)

{% highlight sh %}
.Index(p => p.Title, unique: true);
{% endhighlight %}

2. 我們也增加了一個不得為 null 的 Blogs.Rating 欄位。如果有任何資料已存在資料表中將會得到 CLR 資料型別的預設值，例如 Rating 是 int 如果 Blog 資料表已經有資料那這欄位就會得到 0 。在這邊我們也透過修改遷移檔把預設值變成 3 。
完成的範例如下:

{% highlight csharp %}
namespace MigrationsDemo.Migrations
{
    using System;
    using System.Data.Entity.Migrations;
    
    public partial class AddPostClass : DbMigration
    {
        public override void Up()
        {
            CreateTable(
                "dbo.Posts",
                c => new
                    {
                        PostId = c.Int(nullable: false, identity: true),
                        Title = c.String(maxLength: 200),
                        Content = c.String(),
                        BlogId = c.Int(nullable: false),
                    })
                .PrimaryKey(t => t.PostId)
                .ForeignKey("dbo.Blogs", t => t.BlogId, cascadeDelete: true)
                .Index(t => t.BlogId)
                .Index(p => p.Title, unique: true);

            AddColumn("dbo.Blogs", "Rating", c => c.Int(nullable: false, defaultValue: 3));
        }
        
        public override void Down()
        {
            DropIndex("Posts", new[] { "Title" });
            DropIndex("dbo.Posts", new[] { "BlogId" });
            DropForeignKey("dbo.Posts", "BlogId", "dbo.Blogs");
            DropColumn("dbo.Blogs", "Rating");
            DropTable("dbo.Posts");
        }
    }
}

{% endhighlight %}

當我們完成所有的編輯就可以使用 Update-Database 去修改資料庫實際的表格結構，這一次我們使用 -Verbose 參數來看看實際執行Migration時的詳細資料:

{% highlight sh %}
Update-Database -Verbose
{% endhighlight %}

當我們使用這個指令時就會看到:

{% highlight sh %}
PM> Update-database -verbose
Using StartUp project 'MigrationsDemo'.
Using NuGet project 'MigrationsDemo'.
Specify the '-Verbose' flag to view the SQL statements being applied to the target database.
Target database is: 'MigrationsDemo.BlogContext' (DataSource: (localdb)\v11.0, Provider: System.Data.SqlClient, Origin: Convention).
Applying code-based migrations: [201303100605244_AddPostClass].
Applying code-based migration: 201303100605244_AddPostClass.
CREATE TABLE [dbo].[Posts] (
    [PostId] [int] NOT NULL IDENTITY,
    [Title] [nvarchar](200),
    [Content] [nvarchar](max),
    [BlogId] [int] NOT NULL,
    CONSTRAINT [PK_dbo.Posts] PRIMARY KEY ([PostId])
)
CREATE INDEX [IX_BlogId] ON [dbo].[Posts]([BlogId])
CREATE UNIQUE INDEX [IX_Title] ON [dbo].[Posts]([Title])
ALTER TABLE [dbo].[Blogs] ADD [Rating] [int] NOT NULL DEFAULT 3
ALTER TABLE [dbo].[Posts] ADD CONSTRAINT [FK_dbo.Posts_dbo.Blogs_BlogId] FOREIGN KEY ([BlogId]) REFERENCES [dbo].[Blogs] ([BlogId]) ON DELETE CASCADE
[Inserting migration history record]
Running Seed method.
{% endhighlight %}

####資料遷移和使用SQL
-----

目前為止我們看過了關於資料遷移的操作，但我們並沒有改變或搬移任何資料，讓我們複習一下目前為止重要的指令:

{% highlight sh %}
Enable-Migrations
Add-Migration [name]
Update-Database -Verbose
{% endhighlight %}

有些情況下我們需要搬移資料。目前 Migration 機制還沒有支援資料搬移，但我們可以透過使用 SQL 指令來完成這個任務。
新增一個 Post.Abstract

{% highlight csharp %}
public string Abstract { get; set; }
{% endhighlight %}

然後我們想要預先為已存在的 Post 資料列填入 Abstract(摘要) 而且這些內容要從 Content 取得。
編輯Model之後執行:

{% highlight sh %}
Add-Migration AddPostAbstract 
{% endhighlight %}

執行完 Add-Migration 之後遷移檔中就幫我們設定了關於資料表結構變更的程式
但是我們想要預先將 Content 欄位中的前 100 個字元填入 Abstract
接著讓我們修改遷移檔

{% highlight csharp %}
namespace MigrationsDemo.Migrations
{
    using System;
    using System.Data.Entity.Migrations;
    
    public partial class AddPostAbstract : DbMigration
    {
        public override void Up()
        {
            AddColumn("dbo.Posts", "Abstract", c => c.String());
            Sql("UPDATE Posts SET Abstract = LEFT(Content, 100) WHERE Abstract IS NULL");
        }
        
        public override void Down()
        {
            DropColumn("dbo.Posts", "Abstract");
        }
    }
}
{% endhighlight %}


編輯完成之後讓我們再度執行 Update-Database

####遷移指定的版本(包含降版)
-----

到這一小節我們都是升級最新的 Migration 檔，但有可能有些時候你想要升級/降級到特定的遷移檔。假設我們想要改變資料庫到 AddBlogUrl 這個遷移檔我們可以使用 TargetMigration 切換到降版

執行

{% highlight sh %}
Update-Database –TargetMigration: AddBlogUrl
{% endhighlight %}

這個指令會執行程式中 Down() 的 Script 一路從 AddBlogAbstract 一直往下降版本，所以這邊要注意的是每一版和上一版之間的 Down() 如果有自訂的設定
也要有對應的還原。

####建立 SQL Script
-----

如果其他的開發者想要在他們的機器執行這些改變，他們只需要在版本控制系統取得每一次的改變(資料庫遷移檔)。每當我們更新他們就可以透過 Migration 檔案和 Update-Database 指令切換資料庫的結構。然而如果我們想要傳遞這些變更到測試伺服器並且模擬實際產品環境我們可能想要 SQL Script 如此我們就可以把工作交給DBA。

執行 Update-Database 不過這次我們帶入參數 -Script

{% highlight sh %}
Update-Database -Script
{% endhighlight %}

接著就會產生 SQL Script 而且沒有變更資料庫，我們也可以指定一個 source 和一個 target  來產生特定條件的 SQL。

我們想要產生從空白資料庫到最新版的 Script 時

{% highlight sh %}
Update-Database -Script -SourceMigration: $InitialDatabase -TargetMigration: AddPostAbstract 
{% endhighlight %}

註: 如果你不指定 target Migration 將會使用最新的遷移檔。如果你不指定 source Migration會使用目前資料庫的狀態。


####於程式啟動時自動更新資料結構
----

如果你想要發佈的應用程式，當程式執行時自動更新資料庫您可以透過註冊 MigrateDatabaseToLatestVersion 初始化物件來辦到。
一個資料庫初始化物件 (Database initializer) 只是用來確認資料庫目前是否設定正確。這些邏輯在第一次 Context 類別被使用的時候執行。
當我們更新 Programs.cs 如下:

註: 注意您需要加入 System.Data.Entity 因為要使用 Database.SetInitializer()。
當我們建立了一個初始化物件時我們需要指定 context type (BlogContext) 和 migration 的設定檔(Configuration)。
Configuration 類別會在我們使用 Enable-Migrations 時一起被加入 Migrations 目錄
程式碼如下:

{% highlight csharp %}
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace MigrationsDemo
{
    class Program
    {
        static void Main(string[] args)
        {
            Database.SetInitializer(new MigrateDatabaseToLatestVersion<BlogContext, Migrations.Configuration>);
            using (var db = new BlogContext())
            {
                db.Blogs.Add(new Blog { Name="Another Blog"});
                db.SaveChanges();

                foreach (var blog in db.Blogs)
                {
                    Console.WriteLine(blog.Name);
                }

                Console.WriteLine("Press any key to exit...");
                Console.ReadKey();
            }
        }
    }
}
{% endhighlight %}

現在每當您的應用程式執行的時候就會先確認資料庫如果有變動就會自動 Migration 。

在這個練習你看到了如何設定結構，修改和使用程式來變更您的資料庫。您也學習到如何取得
SQL Script和如何使用遷移檔。以及最後如何自動化變更資料庫。
