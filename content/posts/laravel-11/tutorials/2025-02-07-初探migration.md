---
title: "初探 migration"
date: 2025-02-07 11:30:00 +0800
categories: 
  - laravel
tags:
  - laravel
---

本文件的範例來自 Udemy 課程

Master Laravel 11 & PHP: From Beginner to Advanced: \#3.17 ~ #3.18

## 建立 Model

使用 `artisan make:model` 指令建立名稱為 Task 的 Model

```bash
# 指令: artisan make:model [options ...] <ModelName>
php artisan make:model Task -m

# -m 代表同時建立 migration 檔案

# 產生以下檔案： 
# app/Models/Task.php
# database/migrations/YYYY_MM_DD_XXXXXX_create_tasks_table.php
```

{{< alert type="notice" >}}
\<ModelName> 一般使用單數名詞，laravel 產生 migration 會自動替資料表名稱加上 "s"。  
例如本例會建立 Tasks 資料表。
{{< /alert >}}

### 建立資料表欄位

編輯產生的 `database/migrations/..._create_tasks_table.php`

```php
/**
 * Run the migrations.
 */
public function up(): void
{
    Schema::create('tasks', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->text('description');
        $table->text('long_description')->nullable();
        $table->boolean('completed')->default(false);
        $table->timestamps();
    });
}

/**
 * Reverse the migrations.
 */
public function down(): void
{
    Schema::dropIfExists('tasks');
}
```

`up()` 方法內定義此 migration 上行的內容，例如建立相關欄位。  
`down()` 方法內定義 migration 下行的內容(相當於上一步)的內容，例如刪除資料表。

完成編輯後，執行 `artisan migrate` 即可運行 migration 上行(up)。

```bash
php artisan migrate
# 運行尚未執行過的 migration 檔案的 up()
# 本例將會在資料庫新增資料表 tasks。
```

{{< alert type="info" >}}
資料庫中的 migrations 資料表即是用來記錄/更新每次運行的 migration 步驟。
{{< /alert >}}

### 補充：回復上一個 migration

若要回復上一個 migration，則使用 `artisan migrate:rollback` 來運行上一次 migration 的下行(down)。

```bash
php artisan migrate:rollback
# 運行上一次 migration 檔案的 down()
# 本例將會刪除資料庫 tasks 資料表
```

{{< alert type="danger" title="Danger" >}}
**rollback 的動作將會刪除資料表或移除欄位。**

**執行此指令前請務必清楚自己要做什麼，並且做好資料庫備份。**
{{< /alert >}}

## 使用 Factory 和 Seed 建立測試用的假資料

在 development 中，若需要建立測試的假資料，可以適用 Factory 搭配 Seed 來達成。  
例如在本例中建立 UserFactory 和 TaskFactory，分別替 users 和 tasks 資料表建立假資料。

laravel 專案預設已內建 UserFactory，所以這裡展示如何建立 TaskFactory。

### 建立 Factory

`artisan make:factory` 指令用來建立 Factory 檔案

```bash
# 指令: artisan make:factory [options ...] <FactoryName>
php artisan make:factory TaskFactory --model=Task

# --model 用來指定 Factroy 對應的 Model 名稱。在本例中為 Task。

# 產生以下檔案：
# database/factories/TaskFactory.php
```

編輯 TaskFactory.php

```php
/**
 * Define the model's default state.
 *
 * @return array<string, mixed>
 */
public function definition(): array
{
    return [
        'title' => fake()->sentence,
        'description' => fake()->paragraph,
        'long_description' => fake()->paragraph(7, true),
        'completed' => fake()->boolean
    ];
}
```

在 `definition()` 方法內定義每個欄位對應的假資料類型。詳細的假資料 `fake()` 項目請參考官方文件。

若 Model 要使用 Factory 來產生資料，則必須加入 `use HasFactory` 聲明。  
編輯 `app/Models/Task.php`

```php
class Task extends Model
{
    use HasFactory;
}
```

### 編輯 seeder 檔案

seeder 檔案用來協助 Factory 檔案產生假資料。

編輯 `database/seenders/DatabaseSeeder.php`

```php
/**
 * Seed the application's database.
 */
public function run(): void
{
    User::factory(10)->create();
    Task::factory(20)->create();
}
```

在 `run()` 方法內呼叫 Model 的 factory(n) 來產生 n 筆假資料。`factory()->create()` 預設會根據 Factory 的 `definition()` 內容產生假資料。

指令 `artisan db:seed` 可根據 DatabaseSeeder.php 的定義產生假資料。

```php
php artisan db:seed

# 根據上述設定將產生
# users 10 筆
# tasks 20 筆
```

每次執行 artisan db:seed 都新增資料，若是要將資料庫清空，則可以使用：

`artisan migrate:refresh --seed`

{{< alert type="danger" title="Danger" >}}
migrate:refresh 會執行所有 migration 的 down()，讓資料庫回復到最原始的狀態，然後重新執行 migration 的 up()。  
也就是說**所有的資料將會被清空，因此請僅僅在 development 中使用。**

**執行此指令前請務必清楚自己要做什麼，並且做好資料庫備份。**
{{< /alert >}}

#### 補充：產生不同狀態的假資料

若要產生不同狀態的假資料，例如 `definition()` 定義的某個特定欄位的狀態，想要產生不一樣的狀態來測試其他的可能路徑，則可在 Factory 內加入自訂義的方法來覆蓋。

例如 `database/factories/UserFacotry.php`

```php
/**
 * Define the model's default state.
 *
 * @return array<string, mixed>
 */
public function definition(): array
{
    return [
        'name' => fake()->name(),
        'email' => fake()->unique()->safeEmail(),
        'email_verified_at' => now(),
        'password' => static::$password ??= Hash::make('password'),
        'remember_token' => Str::random(10),
    ];
}

/**
 * Indicate that the model's email address should be unverified.
 */
public function unverified(): static
{
    return $this->state(fn (array $attributes) => [
        'email_verified_at' => null,
    ]);
}
```

以上建立一個 `unverified()` 方法，並設定 email_verified_at 為 null，則可在 seeder 內使用 `User::factory()->unverified()->create()` 來產生 email_verfieid_at 為 null 的假資料。

例如

```php
public function run(): void
{
    User::factory(10)->create();
    User::factory(2)->unverified()->create();
    Task::factory(20)->create();
}
```

以上將會產生 10 筆 email_verified_at 為現在時間以及 2 筆 email_verified_at 為 null 共 12 筆假資料。
