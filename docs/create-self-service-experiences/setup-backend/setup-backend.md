---
title: 设置后端

---

import DocCardList from '@theme/DocCardList';

# 设置后端

<center>

<iframe width="60%" height="400" src="https://www.youtube.com/embed/cU7W3xYbsEw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen allow="fullscreen;"></iframe>

</center>

Port 的自助操作支持各种后端，用户调用操作时可触发这些后端。

下面是基本的后端模型: 

![self-service action backend diagram](../../../static/img/self-service-actions/setup-backend/backend-flow-diagram.png)

Port 后端集成包括以下步骤: 

1. **在 Port** 中触发操作 - 触发器可以是与自助操作用户界面交互的用户，也可以是通过应用程序接口触发操作的自动化程序；
2. **Port 生成事件有效载荷** - 有效载荷包括有关调用操作和用户输入的元数据；
3. **将有效载荷发送到后端**--后端可以是一个 URL、一个专用的 Kafka 主题或一个 CI/CD 工作流和 Pipelines；
4. **您的后端接收有效载荷并处理请求**--根据操作，后端可能会打开一个 PR、创建一个新的云资源、提供一个新的环境等；
5. **您的后端更新 Port 的执行状态** - 您可以通过添加日志、附加到其他工作流或管道的链接来丰富 Port 中的操作运行对象，以帮助完成请求，并在操作完成后添加最终的成功/失败状态。

## 支持的后端

<DocCardList/>