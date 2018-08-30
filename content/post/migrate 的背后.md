---
title: "migrate 的背后"
date: 2018-08-30
tags: [laravel, migration]
draft: true
---
```
/**
   * Log that a migration was run.
   *
   * @param  string  $file
   * @param  int  $batch
   * @return void
   */
  public function log($file, $batch)
  {
      $record = ['migration' => $file, 'batch' => $batch];

      $this->table()->insert($record);
  }
```
执行一个 migration 都会在数据表 migrations 插入一条数据。

在执行 php artisan migrate 之后：  
laravel 会找到 migrations 目录下所有 migration 集合 A，
```
/**
   * Get all of the migration files in a given path.
   *
   * @param  string|array  $paths
   * @return array
   */
  public function getMigrationFiles($paths)
  {
      return Collection::make($paths)->flatMap(function ($path) {
          return $this->files->glob($path.'/*_*.php');
      })->filter()->sortBy(function ($file) {
          return $this->getMigrationName($file);
      })->values()->keyBy(function ($file) {
          return $this->getMigrationName($file);
      })->all();
  }
```
然后通过下面的函数找到已经执行过的 migration 的集合 B （在数据库 migrations 表中记录），

```
/**
 * Get the completed migrations.
 *
 * @return array
 */
public function getRan()
{
    return $this->table()
            ->orderBy('batch', 'asc')
            ->orderBy('migration', 'asc')
            ->pluck('migration')->all();
}

```

A 和 B 的差集 C，就是当前需要执行的 migration。
