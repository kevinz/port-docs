---
sidebar_position: 9
title: Meta 💲

---

# 💲 元属性

元属性是默认存在于 Port 中每个实体上的属性。

元属性总是在其前面使用美元符号 (`$`)来引用，这样可以更容易地分辨一个属性是用户定义的还是元属性。

下表列出了所有可用的元属性: 


| Meta-property | Description | Notes |
| ------------- | ----------- | ----- |

| `$identifier` | **Unique** Entity identifier, used for API calls, programmatic access and distinguishing between different entities | 
| `$title` | A human-readable name for the entity | |
| `$team`       | The team this entity belongs to| |
| `$icon`       | The entity's icon | |
| `$createdAt`  | The entity's creation time | Value is set upon creation and never changes |
| `$updatedAt`  | The entity's last update time | Value is updated automatically |
| `$createdBy`  | The user who created the entity |  Value is set upon creation and never changes |
| `$updatedBy`  | The user who last updated the entity |   Value is updated automatically |
| `$blueprint`  | The entity's blueprint | |