---

sidebar_position: 1

---

import InstallPulumi from "./_pulumi_provider_base.mdx"

import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";

# Pulumi

通过我们与[Pulumi](https://www.pulumi.com/) 的集成，您可以将基础设施的状态与 Port 中代表它们的实体结合起来。

通过使用 Port 的 Providers，您可以轻松地将 Port 与现有的 IaC 定义集成，Pulumi 提供的每个资源也可以使用相同的定义文件报告给软件目录。

<!-- :::info port pulumi provider
You can view the official registry page for our Pulumi provider [here](https://)
::: -->

## 💡 Pulumi Providers 常见用例

例如，我们的 Providers 可以轻松地将 IaC 定义中的数据直接填入软件目录: 

* 报告**云账户**；
* 报告**数据库**；
* 报告**lambdas**和**托管 Kubernetes 服务**(EKS、AKS、GKE 等)；
* 等等。

## 安装

<InstallPulumi/>

## Pulumi 定义结构

Port 的 Pulumi Providers 支持以下资源将数据摄取到目录中: 

### `实体

实体 "资源定义了一个基本实体: 

<Tabs groupId="entity-resource" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import pulumi
from port_pulumi import Entity,BlueprintPropertiesArgs

entity = Entity(
    "myEntity",
    title="My Entity",
    blueprint="myBlueprint",
    properties=BlueprintPropertiesArgs(),
    relations={}
)
```

</TabItem>

<TabItem value="typescript">

```typescript showLineNumbers
import * as pulumi from "@pulumi/pulumi";
import { Entity } from "@port-labs/port";

export const entity = new Entity("myEntity", {
  title: "My Entity",
  blueprint: "myBlueprint",
  properties: {},
  relations: {},
});
```

</TabItem>

<TabItem value="javascript">

```javascript showLineNumbers
"use strict";
const pulumi = require("@pulumi/pulumi");
const port = require("@port-labs/port");

const entity = new Entity("myEntity", {
  title: "My Entity",
  blueprint: "myBlueprint",
  properties: {},
  relations: {},
});

exports.title = entity.title;
```

</TabItem>
<TabItem value="go">

```go showLineNumbers
// A Go Pulumi program

package main

import (
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
    "github.com/port-labs/pulumi-port/sdk/go/port"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
        entity, err := port_pulumi.NewEntity(ctx, "myEntity", &port_pulumi.EntityArgs{
            Title:      pulumi.String("My Entity"),
            Blueprint:  pulumi.String("myBlueprint"),
            Properties: port.EntityPropertiesArgs{},
            Relations:  port.EntityRelations{},
        })
        if err != nil {
            return err
        }
        ctx.Export("entityId", entity.ID())
        return nil
    })
}
```

</TabItem>

</Tabs>

以下参数是**必需的**: 

* blueprint` - 据以创建此实体的蓝图的标识符；
* title: 实体的标题；
* 一个或多个[`properties`](#properties-schema) 模式定义。

还可以指定以下参数作为 `port_entity` 资源的一部分: 

* `identifier` - 实体的标识符；
    - 如果没有 Provider，则会自动生成一个标识符。
* `team` - 拥有该实体的团队；
* `teams` - 拥有该实体的团队的数组；
* `run_id` - 创建实体的操作的运行 ID。

### `属性`模式

属性 "模式为实体的一个属性指定一个值。

#### 定义

<Tabs groupId="properties" queryString="property" defaultValue="string" values={[
{label: "String", value: "string"},
{label: "Number", value: "number"},
{label: "Boolean", value: "boolean"},
{label: "Object", value: "object"},
{label: "Array", value: "array"},
{label: "URL", value: "url"},
{label: "Email", value: "email"},
{label: "User", value: "user"},
{label: "Team", value: "team"},
{label: "Datetime", value: "datetime"},
{label: "Timer", value: "timer"},
{label: "YAML", value: "yaml"}
]}>

<TabItem value="string">

<Tabs groupId="string-definition" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""
from port_pulumi import Entity,EntityPropertiesArgs

entity = Entity(
  "myEntity",
  identifier="myEntity",
  title="My Entity",
  blueprint="myBlueprint",
  # highlight-start
  properties=EntityPropertiesArgs(
    string_props={
      "myStringProp": "My string"
    }
  ),
  # highlight-end
  relations={}
)
```

</TabItem>

<TabItem value="typescript">

```typescript showLineNumbers
import * as pulumi from "@pulumi/pulumi";
import { Entity } from "@port-labs/port";

export const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    stringProps: {
      myStringProp: {
        title: "My string",
        required: false
      }
    }
  },
  // highlight-end
  relations: {},
});
```

</TabItem>

<TabItem value="javascript">

```javascript showLineNumbers
"use strict";
const pulumi = require("@pulumi/pulumi");
const port = require("@port-labs/port");

const entity = new port.Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    stringProps: {
      myStringProp: {
          title: "My string",
          required: false
      }
    }
  },
  // highlight-end
  relations: {},
});

exports.title = entity.title;
```

</TabItem>
<TabItem value="go">

```go showLineNumbers
package main

import (
    "github.com/port-labs/pulumi-port/sdk/go/port"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
    	entity, err := port.NewEntity(ctx, "entity", &port.EntityArgs{
    		Identifier: pulumi.String("myEntity"),
    		Title:      pulumi.String("My Entity"),
    		Blueprint:  pulumi.String("myBlueprint"),
    		// highlight-start
            Properties: port.EntityPropertiesArgs{
              StringProps: pulumi.StringMap{
                "myStringProp": pulumi.String("myStringValue"),
              },
            },
    		// highlight-end
    	})
    	ctx.Export("entity", entity.Title)
    	if err != nil {
    		return err
    	}
    	return nil
    })
}
```

</TabItem>

</Tabs>
</TabItem>
<TabItem value="number">

<Tabs groupId="number-definition" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import pulumi
from port_pulumi import Entity,EntityPropertiesArgs

entity = Entity(
    "myEntity",
    identifier="myEntity",
    title="My Entity",
    blueprint="myBlueprint",
    # highlight-start
    properties=EntityPropertiesArgs(
      number_props={
        "myNumberProp": 7,
      }
    ),
    # highlight-end
    relations=[])
```

</TabItem>

<TabItem value="typescript">

```typescript showLineNumbers
import * as pulumi from "@pulumi/pulumi";
import { Entity } from "@port-labs/port";

export const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    numberProps: {
      "myNumberProp": 7
    }
  },
  // highlight-end
  relations: {}
});
```

</TabItem>

<TabItem value="javascript">

```javascript showLineNumbers
"use strict";
const pulumi = require("@pulumi/pulumi");
const port = require("@port-labs/port");

const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    numberProps: {
      "myNumberProp": 7
    }
  },
  // highlight-end
  relations: {}
});

exports.title = entity.title;
```

</TabItem>
<TabItem value="go">

```go showLineNumbers
package main

import (
    "github.com/port-labs/pulumi-port/sdk/go/port"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
    	entity, err := port.NewEntity(ctx, "entity", &port.EntityArgs{
    		Identifier: pulumi.String("myEntity"),
    		Title:      pulumi.String("My Entity"),
    		Blueprint:  pulumi.String("myBlueprint"),
    		// highlight-start
            Properties: port.EntityPropertiesArgs{
              NumberProps: pulumi.Float64Map{
                "myNumberProp": pulumi.Float64(7),
              },
            },
    		// highlight-end
    	})
    	ctx.Export("entity", entity.Title)
    	if err != nil {
    		return err
    	}
    	return nil
    })
}
```

</TabItem>

</Tabs>

</TabItem>
<TabItem value="boolean">

<Tabs groupId="boolean-definition" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import pulumi
from port_pulumi import Entity,EntityPropertiesArgs

entity = Entity(
    "myEntity",
    identifier="myEntity",
    title="My Entity",
    blueprint="myBlueprint",
    # highlight-start
    properties=EntityPropertiesArgs(
      boolean_props={
        "myBooleanProp": True
      },
    ),
    # highlight-end
    relations={}
)
```

</TabItem>

<TabItem value="typescript">

```typescript showLineNumbers
import * as pulumi from "@pulumi/pulumi";
import { Entity } from "@port-labs/port";

export const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    booleanProps: {
      "myBooleanProp": true
    }
  },
  // highlight-end
  relations: {},
});
```

</TabItem>

<TabItem value="javascript">

```javascript showLineNumbers
"use strict";
const pulumi = require("@pulumi/pulumi");
const port = require("@port-labs/port");

const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    booleanProps: {
      "myBooleanProp": true
    }
  },
  // highlight-end
  relations: {},
});

exports.title = entity.title;
```

</TabItem>
<TabItem value="go">

```go showLineNumbers
package main

import (
    "github.com/port-labs/pulumi-port/sdk/go/port"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
    	entity, err := port.NewEntity(ctx, "entity", &port.EntityArgs{
    		Identifier: pulumi.String("myEntity"),
    		Title:      pulumi.String("My Entity"),
    		Blueprint:  pulumi.String("myBlueprint"),
    		// highlight-start
            Properties: port.EntityPropertiesArgs{
              BooleanProps: pulumi.BoolMap{
                "myBooleanProp": pulumi.Bool(true),
              },
            },
    		// highlight-end
    	})
    	ctx.Export("entity", entity.Title)
    	if err != nil {
    		return err
    	}
    	return nil
    })
}
```

</TabItem>

</Tabs>

</TabItem>
<TabItem value="object">

<Tabs groupId="object-definition" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import json
import pulumi
from port_pulumi import Entity,EntityPropertiesArgs

entity = Entity(
    "myEntity",
    identifier="myEntity",
    title="My Entity",
    blueprint="myBlueprint",
    # highlight-start
    properties=EntityPropertiesArgs(
      object_props={
        "myObjectProp":  json.dumps({"hello": "world"})
      }
    ),
    # highlight-end
    relations={}
)
```

</TabItem>

<TabItem value="typescript">

```typescript showLineNumbers
import * as pulumi from "@pulumi/pulumi";
import { Entity } from "@port-labs/port";

export const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    objectProps: {
      "myObjectProp": JSON.stringify({hello: "world"})
    }
  },
  // highlight-end
  relations: {},
});
```

</TabItem>

<TabItem value="javascript">

```javascript showLineNumbers
"use strict";
const pulumi = require("@pulumi/pulumi");
const port = require("@port-labs/port");

const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    objectProps: {
      "myObjectProp": JSON.stringify({hello: "world"})
    }
  },
  // highlight-end
  relations: {},
});

exports.title = entity.title;
```

</TabItem>

<TabItem value="go">

```go showLineNumbers
package main

import (
    "encoding/json"

    "github.com/port-labs/pulumi-port/sdk/go/port"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    // highlight-start
    obj := map[string]string{"hello": "world"}
    objStr, err := json.Marshal(obj)

    if err != nil {
    	panic(err)
    }
    // highlight-end

    pulumi.Run(func(ctx *pulumi.Context) error {
    	entity, err := port.NewEntity(ctx, "entity", &port.EntityArgs{
    		Identifier: pulumi.String("myEntity"),
    		Title:      pulumi.String("My Entity"),
    		Blueprint:  pulumi.String("myBlueprint"),
    		// highlight-start
            Properties: port.EntityPropertiesArgs{
              ObjectProps: pulumi.StringMap{
                "myObjectProperty": pulumi.String(objStr),
              },
            },
    		// highlight-end
    	})
    	ctx.Export("entity", entity.Title)
    	if err != nil {
    		return err
    	}
    	return nil
    })
}
```

</TabItem>

</Tabs>

</TabItem>

<TabItem value="array">

<Tabs groupId="array-definition" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import json
import pulumi
from port_pulumi import Entity,EntityPropertiesArgs

entity = Entity(
    "myEntity",
    identifier="myEntity",
    title="My Entity",
    blueprint="myBlueprint",
    # highlight-start
    properties=EntityPropertiesArgs(
      array_props={
        "myArrayProp": ["hello", "world"]
      }
    ),  
    # highlight-end
    relations={}
)
```

</TabItem>

<TabItem value="typescript">

```typescript showLineNumbers
import * as pulumi from "@pulumi/pulumi";
import { Entity } from "@port-labs/port";

export const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    arrayProps: {
      stringItems: {
        "myArrayProp": ["a", "b", "c"]
      }
    }
  },
  // highlight-end
  relations: {},
});
```

</TabItem>

<TabItem value="javascript">

```javascript showLineNumbers
"use strict";
const pulumi = require("@pulumi/pulumi");
const port = require("@port-labs/port");

const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    arrayProps: {
      stringItems: {
        "myArrayProp": ["a", "b", "c"]
      }
    }
  },
  // highlight-end
  relations: {},
});

exports.title = entity.title;
```

</TabItem>

<TabItem value="go">

```go showLineNumbers
package main

import (
    "github.com/port-labs/pulumi-port/sdk/go/port"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
    	entity, err := port.NewEntity(ctx, "entity", &port.EntityArgs{
    		Identifier: pulumi.String("myEntity"),
    		Title:      pulumi.String("My Entity"),
    		Blueprint:  pulumi.String("myBlueprint"),
    		// highlight-start
            Properties: port.EntityPropertiesArgs{
              ArrayProps: port.EntityPropertiesArrayPropsArgs{
                StringItems: pulumi.StringArrayMap{
                  "myArrayProp": pulumi.StringArray{
                    pulumi.String("hello"),
                    pulumi.String("world"),
                    pulumi.String("!"),
                  },
                },
              },
            },
    		// highlight-end
    	})
    	ctx.Export("entity", entity.Title)
    	if err != nil {
    		return err
    	}
    	return nil
    })
}
```

</TabItem>

</Tabs>

</TabItem>

<TabItem value="url">

<Tabs groupId="url-definition" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import json
import pulumi
from port_pulumi import Entity,EntityPropertiesArgs

entity = Entity(
    "myEntity",
    identifier="myEntity",
    title="My Entity",
    blueprint="myBlueprint",
    # highlight-start
    properties=EntityPropertiesArgs(
      string_props={
        "myUrlProp": "https://example.com"
      }
    ),
    # highlight-end
    relations={}
)
```

</TabItem>

<TabItem value="typescript">

```typescript showLineNumbers
import * as pulumi from "@pulumi/pulumi";
import { Entity } from "@port-labs/port";

export const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    stringProps: {
      myUrlProp: "https://example.com",
    }
  },
  // highlight-end
  relations: {},
});
```

</TabItem>

<TabItem value="javascript">

```javascript showLineNumbers
"use strict";
const pulumi = require("@pulumi/pulumi");
const port = require("@port-labs/port");

const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    stringProps: {
      myUrlProp: "https://example.com",
    }
  },
  // highlight-end
  relations: {},
});

exports.title = entity.title;
```

</TabItem>

<TabItem value="go">

```go showLineNumbers
package main

import (
    "github.com/port-labs/pulumi-port/sdk/go/port"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
    	entity, err := port.NewEntity(ctx, "entity", &port.EntityArgs{
    		Identifier: pulumi.String("myEntity"),
    		Title:      pulumi.String("My Entity"),
    		Blueprint:  pulumi.String("myBlueprint"),
    		// highlight-start
            Properties: port.EntityPropertiesArgs{
              StringProps: pulumi.StringMap{
                "myUrlProp": pulumi.String("https://example.com"),
              },
            },
    		// highlight-end
    	})
    	ctx.Export("entity", entity.Title)
    	if err != nil {
    		return err
    	}
    	return nil
    })
}
```

</TabItem>

</Tabs>

</TabItem>

<TabItem value="email">

<Tabs groupId="email-definition" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import json
import pulumi
from port_pulumi import Entity

entity = Entity(
    "myEntity",
    identifier="myEntity",
    title="My Entity",
    blueprint="myBlueprint",
    # highlight-start
    properties=EntityPropertiesArgs(
      string_props={
        "myEmailProp": "me@example.com"
      }
    ),
    # highlight-end
    relations={}
)
```

</TabItem>

<TabItem value="typescript">

```typescript showLineNumbers
import * as pulumi from "@pulumi/pulumi";
import { Entity } from "@port-labs/port";

export const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    stringProps: {
      myEmailProp: "me@example.com",
    }
  },
  // highlight-end
  relations: {},
});
```

</TabItem>

<TabItem value="javascript">

```javascript showLineNumbers
"use strict";
const pulumi = require("@pulumi/pulumi");
const port = require("@port-labs/port");

const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    stringProps: {
      myEmailProp: "me@example.com",
    }
  },
  // highlight-end
  relations: {},
});

exports.title = entity.title;
```

</TabItem>

<TabItem value="go">

```go showLineNumbers
package main

import (
    "github.com/port-labs/pulumi-port/sdk/go/port"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
    	entity, err := port.NewEntity(ctx, "entity", &port.EntityArgs{
    		Identifier: pulumi.String("myEntity"),
    		Title:      pulumi.String("My Entity"),
    		Blueprint:  pulumi.String("myBlueprint"),
    		// highlight-start
            Properties: port.EntityPropertiesArgs{
              StringProps: pulumi.StringMap{
                "myEmailProp": pulumi.String("me@example.com"),
              },
            },
    		// highlight-end
    	})
    	ctx.Export("entity", entity.Title)
    	if err != nil {
    		return err
    	}
    	return nil
    })
}
```

</TabItem>

</Tabs>

</TabItem>

<TabItem value="user">

<Tabs groupId="user-definition" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import json
import pulumi
from port_pulumi import Entity,EntityPropertiesArgs

entity = Entity(
    "myEntity",
    identifier="myEntity",
    title="My Entity",
    blueprint="myBlueprint",
    # highlight-start
    properties=EntityPropertiesArgs(
      string_props={
        "myUserProp": "argo-admin"
      }
    ),
    # highlight-end
    relations={}
)
```

</TabItem>

<TabItem value="typescript">

```typescript showLineNumbers
import * as pulumi from "@pulumi/pulumi";
import { Entity } from "@port-labs/port";

export const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    stringProps: {
      myUserProp: "argo-admin",
    }
  },
  // highlight-end
  relations: {},
});
```

</TabItem>

<TabItem value="javascript">

```javascript showLineNumbers
"use strict";
const pulumi = require("@pulumi/pulumi");
const port = require("@port-labs/port");

const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    stringProps: {
      myUserProp: "argo-admin",
    }
  },
  // highlight-end
  relations: {},
});

exports.title = entity.title;
```

</TabItem>

<TabItem value="go">

```go showLineNumbers
package main

import (
    "github.com/port-labs/pulumi-port/sdk/go/port"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
    	entity, err := port.NewEntity(ctx, "entity", &port.EntityArgs{
    		Identifier: pulumi.String("myEntity"),
    		Title:      pulumi.String("My Entity"),
    		Blueprint:  pulumi.String("myBlueprint"),
    		// highlight-start
            Properties: port.EntityPropertiesArgs{
              StringProps: pulumi.StringMap{
                "myUserProp": pulumi.String("argo-admin"),
              },
            },
    		// highlight-end
    	})
    	ctx.Export("entity", entity.Title)
    	if err != nil {
    		return err
    	}
    	return nil
    })
}
```

</TabItem>

</Tabs>

</TabItem>

<TabItem value="team">

<Tabs groupId="team-definition" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import json
import pulumi
from port_pulumi import Entity,EntityPropertiesArgs

entity = Entity(
    "myEntity",
    identifier="myEntity",
    title="My Entity",
    blueprint="myBlueprint",
    # highlight-start
    properties=EntityPropertiesArgs(
      string_props={
        "myTeamProp": "My string"
      }
    ),
    # highlight-end
    relations={}
)
```

</TabItem>

<TabItem value="typescript">

```typescript showLineNumbers
import * as pulumi from "@pulumi/pulumi";
import { Entity } from "@port-labs/port";

export const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    stringProps: {
      myTeamProp: "argo-admins",
    }
  },
  // highlight-end
  relations: {},
});
```

</TabItem>

<TabItem value="javascript">

```javascript showLineNumbers
"use strict";
const pulumi = require("@pulumi/pulumi");
const port = require("@port-labs/port");

const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    stringProps: {
      myTeamProp: "argo-admins",
    }
  },
  // highlight-end
  relations: {},
});

exports.title = entity.title;
```

</TabItem>

<TabItem value="go">

```go showLineNumbers
package main

import (
    "github.com/port-labs/pulumi-port/sdk/go/port"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
    	entity, err := port.NewEntity(ctx, "entity", &port.EntityArgs{
    		Identifier: pulumi.String("myEntity"),
    		Title:      pulumi.String("My Entity"),
    		Blueprint:  pulumi.String("myBlueprint"),
    		// highlight-start
            Properties: port.EntityPropertiesArgs{
              StringProps: pulumi.StringMap{
                "myTeamProp": pulumi.String("argo-admins"),
              },
            },
    		// highlight-end
    	})
    	ctx.Export("entity", entity.Title)
    	if err != nil {
    		return err
    	}
    	return nil
    })
}
```

</TabItem>

</Tabs>

</TabItem>

<TabItem value="datetime">

<Tabs groupId="datetime-definition" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import json
import pulumi
from port_pulumi import Entity

entity = Entity(
    "myEntity",
    identifier="myEntity",
    title="My Entity",
    blueprint="myBlueprint",
    # highlight-start
    properties=port.EntityPropertiesArgs(
      string_props={
        "myDatetimeProp": "2022-04-18T11:44:15.345Z"
      }
    ),
    # highlight-end
    relations={}
)
```

</TabItem>

<TabItem value="typescript">

```typescript showLineNumbers
import * as pulumi from "@pulumi/pulumi";
import { Entity } from "@port-labs/port";

export const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    stringProps: {
      myDatetimeProp: "2022-04-18T11:44:15.345Z",
    }
  },
  // highlight-end
  relations: {},
});
```

</TabItem>

<TabItem value="javascript">

```javascript showLineNumbers
"use strict";
const pulumi = require("@pulumi/pulumi");
const port = require("@port-labs/port");

const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    stringProps: {
      myDatetimeProp: "2022-04-18T11:44:15.345Z",
    }
  },
  // highlight-end
  relations: {},
});

exports.title = entity.title;
```

</TabItem>

<TabItem value="go">

```go showLineNumbers
package main

import (
    "github.com/port-labs/pulumi-port/sdk/go/port"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
    	entity, err := port.NewEntity(ctx, "entity", &port.EntityArgs{
    		Identifier: pulumi.String("myEntity"),
    		Title:      pulumi.String("My Entity"),
    		Blueprint:  pulumi.String("myBlueprint"),
    		// highlight-start
            Properties: port.EntityPropertiesArgs{
              StringProps: pulumi.StringMap{
                "myDatetimeProp": pulumi.String("2022-04-18T11:44:15.345Z"),
              },
            },
    		// highlight-end
    	})
    	ctx.Export("entity", entity.Title)
    	if err != nil {
    		return err
    	}
    	return nil
    })
}
```

</TabItem>

</Tabs>

</TabItem>
<TabItem value="timer">

<Tabs groupId="timer-definition" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import json
import pulumi
from port_pulumi import Entity,EntityPropertiesArgs

entity = Entity(
    "myEntity",
    identifier="myEntity",
    title="My Entity",
    blueprint="myBlueprint",
    # highlight-start
    properties=EntityPropertiesArgs(
      string_props={
        "myTimerProp": "2022-04-18T11:44:15.345Z"
      }
    ),
    # highlight-end
    relations={}
)
```

</TabItem>

<TabItem value="typescript">

```typescript showLineNumbers
import * as pulumi from "@pulumi/pulumi";
import { Entity } from "@port-labs/port";

export const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    stringProps: {
      myTimerProp: "2022-04-18T11:44:15.345Z",
    }
  },
  // highlight-end
  relations: {},
});
```

</TabItem>

<TabItem value="javascript">

```javascript showLineNumbers
"use strict";
const pulumi = require("@pulumi/pulumi");
const port = require("@port-labs/port");

const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    stringProps: {
      myTimerProp: "2022-04-18T11:44:15.345Z",
    }
  },
  // highlight-end
  relations: {},
});

exports.title = entity.title;
```

</TabItem>

<TabItem value="go">

```go showLineNumbers
package main

import (
    "github.com/port-labs/pulumi-port/sdk/go/port"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
    	entity, err := port.NewEntity(ctx, "entity", &port.EntityArgs{
    		Identifier: pulumi.String("myEntity"),
    		Title:      pulumi.String("My Entity"),
    		Blueprint:  pulumi.String("myBlueprint"),
    		// highlight-start
            Properties: port.EntityPropertiesArgs{
              StringProps: pulumi.StringMap{
                "myTimerProp": pulumi.String("2022-04-18T11:44:15.345Z"),
              },
            },
    		// highlight-end
    	})
    	ctx.Export("entity", entity.Title)
    	if err != nil {
    		return err
    	}
    	return nil
    })
}
```

</TabItem>

</Tabs>

</TabItem>

<TabItem value="yaml">

<Tabs groupId="yaml-definition" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import json
import pulumi
from port_pulumi import Entity,EntityPropertiesArgs

entity = Entity(
    "myEntity",
    identifier="myEntity",
    title="My Entity",
    blueprint="myBlueprint",
    # highlight-start
    properties=EntityPropertiesArgs(
      string_props={
        "myYAMLProp": "my: yaml"
      }
    ),
    # highlight-end
    relations={}
)
```

</TabItem>

<TabItem value="typescript">

```typescript showLineNumbers
import * as pulumi from "@pulumi/pulumi";
import { Entity } from "@port-labs/port";

export const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    stringProps: {
      myYAMLProp: "my: yaml",
    }
  },
  // highlight-end
  relations: [],
});
```

</TabItem>

<TabItem value="javascript">

```javascript showLineNumbers
"use strict";
const pulumi = require("@pulumi/pulumi");
const port = require("@port-labs/port");

const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  // highlight-start
  properties: {
    stringProps: {
      myYAMLProp: "my: yaml",
    }
  },
  // highlight-end
  relations: {},
});

exports.title = entity.title;
```

</TabItem>
<TabItem value="go">

```go showLineNumbers
package main

import (
    "github.com/port-labs/pulumi-port/sdk/go/port"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
    	entity, err := port.NewEntity(ctx, "entity", &port.EntityArgs{
    		Identifier: pulumi.String("myEntity"),
    		Title:      pulumi.String("My Entity"),
    		Blueprint:  pulumi.String("myBlueprint"),
    		// highlight-start
            Properties: port.EntityPropertiesArgs{
              StringProps: pulumi.StringMap{
                "myYAMLProp": pulumi.String("my: yaml"),
              },
            },
    		// highlight-end
    	})
    	ctx.Export("entity", entity.Title)
    	if err != nil {
    		return err
    	}
    	return nil
    })
}
```

</TabItem>

</Tabs>

</TabItem>

</Tabs>

以下参数是**必需的**: 

* `name` -[blueprint definition](../../../define-your-data-model/setup-blueprint/properties/properties.md#structure) 中的属性名称；
* `values` - 属性的值(用于非数组属性)；
* `items` - 属性值数组(用于数组属性)。

### `relations` 模式

关系 "模式将目标实体映射到源实体定义: 

<Tabs groupId="yaml-definition" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import json
import pulumi
from port_pulumi import Entity,EntityRelationsArgs

entity = Entity(
    "myEntity",
    identifier="myEntity",
    title="My Entity",
    blueprint="myBlueprint",
    properties={},
    # highlight-start
    relations=EntityRelationsArgs(
      many_relations={
        "myManyRelation": ["myTargetEntityIdentifier", "myTargetEntityIdentifier2"]
      },
      single_relations={
        "mySingleRelation": "myTargetEntityIdentifier"
      },
    ),
    # highlight-end
)
```

</TabItem>

<TabItem value="typescript">

```typescript showLineNumbers
import * as pulumi from "@pulumi/pulumi";
import { Entity } from "@port-labs/port";

export const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  properties: {},
  // highlight-start
  relations: {
    singleRelations: {
      myRelation: "myTargetEntityIdentifier"
    },
    manyRelations: {
      myRelation: ["myTargetEntityIdentifier", "myTargetEntityIdentifier2"]
    }
  },
  // highlight-end
});
```

</TabItem>

<TabItem value="javascript">

```javascript showLineNumbers
"use strict";
const pulumi = require("@pulumi/pulumi");
const port = require("@port-labs/port");

const entity = new Entity("myEntity", {
  identifier: "myEntity",
  title: "My Entity",
  blueprint: "myBlueprint",
  properties: {},
  // highlight-start
  relations: {
    singleRelations: {
      myRelation: "myTargetEntityIdentifier"
    },
    manyRelations: {
      myRelation: ["myTargetEntityIdentifier", "myTargetEntityIdentifier2"]
    }
  },
  // highlight-end
});

exports.title = entity.title;
```

</TabItem>

<TabItem value="go">

```go showLineNumbers
package main

import (
    "github.com/port-labs/pulumi-port/sdk/go/port"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
    	entity, err := port.NewEntity(ctx, "entity", &port.EntityArgs{
    		Identifier: pulumi.String("myEntity"),
    		Title:      pulumi.String("My Entity"),
    		Blueprint:  pulumi.String("myBlueprint"),
    		// highlight-start
            Relations: port.EntityRelationsArgs{
              SingleRelations: pulumi.StringMap{
                "mySingleRelation": pulumi.String("myTargetEntityIdentifier"),
              },
              ManyRelations: pulumi.StringArrayMap{
                "myManyRelation": pulumi.StringArray{
                  pulumi.String("myTargetEntityIdentifier"),
                  pulumi.String("myTargetEntityIdentifier2"),
                },
              },
            },	
    		// highlight-end
            Properties: port.EntityPropertiesArgs{
    			// ..properties
    		},
    	})
    	ctx.Export("entity", entity.Title)
    	if err != nil {
    		return err
    	}
    	return nil
    })
}
```

</TabItem>

</Tabs>

以下参数是**必需的**: 

* `name` -[relation](../../../define-your-data-model/relate-blueprints/relate-blueprints.md#structure-table) 在蓝图定义中的名称；
* `identifier` - 目标实体的标识符。

:::note 目前，只有使用 Port 的 Providers 才能创建具有 "many: false "关系的实体。

:::

## 使用 Providers 被引用数据

要使用 Provider 将数据引用到软件目录，您需要用首选语言创建一个 `port.Entity` 资源实例: 

<Tabs groupId="sync-data" queryString="current-scenario" defaultValue="create" values={[
{label: "Create", value: "create"},
{label: "Update", value: "update"},
{label: "Delete", value: "delete"},
]}>

<TabItem value="create">

要使用 Pulumi 创建实体，请用您喜欢的语言创建一个文件，并插入以下内容: 

<Tabs groupId="create-resource-examples" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import pulumi
from port_pulumi import Entity

entity = Entity(
    "myEntity",
    title="My Entity",
    blueprint="myBlueprint",
    properties={},
    relations={}
)
```

</TabItem>

<TabItem value="typescript">

```typescript showLineNumbers
import * as pulumi from "@pulumi/pulumi";
import { Entity } from "@port-labs/port";

export const entity = new Entity("myEntity", {
  title: "My Entity",
  blueprint: "myBlueprint",
  properties: {},
  relations: {},
});
```

</TabItem>

<TabItem value="javascript">

```javascript showLineNumbers
"use strict";
const pulumi = require("@pulumi/pulumi");
const port = require("@port-labs/port");

const entity = new Entity("myEntity", {
  title: "My Entity",
  blueprint: "myBlueprint",
  properties: {},
  relations: {},
});

exports.title = entity.title;
```

</TabItem>

<TabItem value="go">

```go showLineNumbers
// A Go Pulumi program

package main

import (
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
    "github.com/port-labs/pulumi-port/sdk/go/port"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
        entity, err := port_pulumi.NewEntity(ctx, "myEntity", &port_pulumi.EntityArgs{
            Title:      pulumi.String("My Entity"),
            Blueprint:  pulumi.String("myBlueprint"),
            Properties: port.EntityPropertiesArgs{},
            Relations:  port.EntityRelationsArgs{},
        })
        if err != nil {
            return err
        }
        ctx.Export("entityId", entity.ID())
        return nil
    })
}
```

</TabItem>

</Tabs>

然后运行以下命令应用更改并更新目录: 

```shell showLineNumbers
pulumi up -y
```

运行这些命令后，您将看到目录已更新为新实体。

</TabItem>

<TabItem value="update">

要使用 Pulumi 更新实体，请更新代码文件中现有的 `port.Entity` 资源，然后运行 `pulumi up -y`。

也可以使用 Port 的 Provider 开始管理现有实体，要开始管理现有实体，请在代码文件中添加新的 `port.Entity` 资源，并进行所需的更改

:::info 

关于向 Providers 添加现有实体的重要说明: 

* 指定实体的 "标识符 "非常重要，否则 Pulumi 将创建一个带有自动生成的标识符的新实体。
* Port 的 Pulumi Providers 使用[create/override](../../api/api.md?update-strategy=create-override#usage) 策略，这意味着对于现有实体，资源定义中未定义的任何属性都将被空值覆盖。

:::

</TabItem>

<TabItem value="delete">

要使用 Pulumi 删除实体，只需删除代码文件中定义的 `port.Entity` 资源，然后运行 `pulumi up -y`。

</TabItem>

</Tabs>