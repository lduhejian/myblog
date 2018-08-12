---
title: "哪些 migration 会执行？"
date: 2018-08-30
tags: [laravel, migration]
draft: false
---
#### php artisan migrate 后会执行以下步骤

* laravel 会找到 migrations 目录下所有 migration 集合 A  

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

* 然后通过下面的函数找到已经执行过的 migration 的集合 B （在数据库 migrations 表中记录）

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

* A 和 B 的差集 C，就是当前需要执行的 migration
