graph TD
    %% 这是一个完整的Katrina业务逻辑的Mermaid蓝图
    %% 定义样式
    style Start fill:#2E8B57,stroke:#333,stroke-width:2px,color:#fff
    style End fill:#DC143C,stroke:#333,stroke-width:2px,color:#fff
    
    %% 主流程开始
    Start[对话开始] --> A;

    %% A区: Bot大脑 - 负责对话、决策与调度
    subgraph Coze Bot (智能调度中心)
        direction LR
        A(信息收集循环) -- 用户持续输入 --> A;
        A -- 信息达到触发条件 --> B{决策: 执行哪项任务?};
        
        B -- 触发[第一次推荐] --> C[调用: 初次推荐工作流];
        C -- 工作流JSON结果 --> F[处理工作流结果];
        
        B -- 触发[二次或被动推荐] --> D[调用: 精准推荐工作流];
        D -- 工作流JSON结果 --> F;

        B -- 信息不足 --> E[引导与追问];
        E --> A;
        
        F -- 结果为 SUCCESS --> G[格式化职位列表];
        G -- 发送后 --> I(引导上传简历);

        F -- 结果为 NOT_FOUND --> H[格式化'无匹配'话术];
        H --> I;
    end

    %% B区: Workflow - 负责结构化任务执行
    subgraph Workflow (结构化执行工具)
        direction TD
        WF_Start(入口: 接收Bot参数);
        WF_Start --> WF_DB[DB节点: 查询职位];
        WF_DB --> WF_Check{结果是否为空?};
        
        WF_Check -- 否 --> WF_Code[Code节点: 执行4x4规则];
        WF_Code --> WF_Format_Success[格式化成功结果JSON];
        WF_Format_Success --> WF_End(出口: 返回JSON给Bot);

        WF_Check -- 是 --> WF_Format_NotFound[格式化'未找到'结果JSON];
        WF_Format_NotFound --> WF_End;
    end

    %% 连接 Bot 和 Workflow
    C -- 携带参数: 公司/方向/级别 --> WF_Start;
    D -- 携带参数: 全部6项信息 --> WF_Start;
    WF_End -- 返回 {status, data} --> F;

    %% 最终流程
    I --> End[对话结束];
