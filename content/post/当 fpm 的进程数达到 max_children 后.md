---
title: "当 fpm 的进程数达到 max_children 后"
date: 2019-05-21
draft: false
tags: [php, fpm]
---
当 fpm 的进程数达到 max_children 后, fpm 只是打一下日志>>
```
if (wp->running_children >= wp->config->pm_max_children) {
    if (!wp->warn_max_children) {
        fpm_scoreboard_update(0, 0, 0, 0, 0, 1, 0, FPM_SCOREBOARD_ACTION_INC, wp->scoreboard);
        zlog(ZLOG_WARNING, "[pool %s] server reached pm.max_children setting (%d), consider raising it", wp->config->name, wp->config->pm_max_children);
        wp->warn_max_children = 1;
    }
    wp->idle_spawn_rate = 1;
    continue;
}
```

参考:  
https://github.com/owenliang/php-fpm-code-analysis