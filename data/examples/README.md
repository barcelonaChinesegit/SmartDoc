# JSONL 数据样例

> 历史资料：当前范围已收敛为科学文档本地问答，详见 /data/czj/DocAgent/docs/科学文档项目范围.md。非科学文档、编程与未实现的工具／平台设想不作为现行任务。原文保留供追溯，不能据此启动旧配置或声称已实现。

以下内容仅用于展示数据结构，文档内容为合成示例。实际使用时，将 `asset://...` 替换为有访问控制的文件位置。不要把保密文档、生产日志、凭据或受许可限制的基准测试数据提交到仓库。

## 基准评测样例

```json
{"id":"bench-0001","split":"dev","task_type":"single_doc_qa","source":{"name":"synthetic_contract_v1","license":"CC0-1.0","source_uri":"internal://synthetic/contract-17"},"doc_ids":["doc-17"],"assets":[{"uri":"asset://doc-17.pdf","sha256":"填写文件SHA256","media_type":"application/pdf","pages":2}],"question":"合同解除需要提前多少天通知？","target":{"answer":"30天","answer_type":"text","abstain":false},"evidence":[{"doc_id":"doc-17","page":2,"bbox_1000":[104,311,864,392],"text_span":"任何一方均可提前三十（30）天书面通知解除本协议。","table":null,"cells":[]}],"validation":{"answer_rule":"规范化后精确匹配","evidence_must_entail_answer":true},"metadata":{"language":"zh","document_type":"合同","difficulty":"easy","noise":[]}}
```

## SFT 示范轨迹样例

```json
{"id":"sft-0001","split":"train","task_type":"evidence_grounded_qa","doc_ids":["doc-17"],"messages":[{"role":"user","content":"查找合同解除的提前通知期限，并标出页码。"},{"role":"assistant","content":"我会检索合同中的解除条款。","rationale_type":"简短决策说明"},{"role":"assistant","tool_call":{"name":"search_document","arguments":{"doc_id":"doc-17","query":"解除 通知期限"}}},{"role":"tool","name":"search_document","content":{"hits":[{"evidence_id":"ev-2-4","page":2,"text":"任何一方均可提前三十（30）天书面通知解除本协议。"}]}},{"role":"assistant","content":"提前通知期限为30天，依据位于第2页。","citations":["ev-2-4"]}],"target":{"answer":"30天","evidence_ids":["ev-2-4"],"abstain":false},"metadata":{"generator":"人工核验","tool_schema_version":"1.0","license":"CC0-1.0"}}
```

## 智能体强化学习 episode 样例

```json
{"episode_id":"rl-0001-rollout-02","task_id":"task-0001","split":"train","environment":{"name":"smartdoc_local_v1","snapshot":"sha256:填写环境快照哈希","seed":17},"task":{"request":"比较两份合同的续约期限，并注明证据页码。","doc_ids":["contract-a","contract-b"]},"trajectory":[{"step":0,"observation_ref":"obs://start","action":{"type":"tool_call","name":"search_document","arguments":{"doc_ids":["contract-a","contract-b"],"query":"续约期限"}},"observation_ref_next":"obs://search-1","labels":{"tool_choice":"正确","arguments":"有效","evidence":"部分命中","continue":"继续"},"reward":{"terminal":0.0,"process":0.2,"cost":-0.01}},{"step":1,"observation_ref":"obs://search-1","action":{"type":"final","answer":"合同A每年续约一次（第4页）；合同B每两年续约一次（第7页）。","citations":["contract-a:p4:ev3","contract-b:p7:ev1"]},"observation_ref_next":"obs://terminal","labels":{"answer":"正确","citations":"证据支持","stop":"正确停止"},"reward":{"terminal":1.0,"process":0.8,"cost":0.0}}],"outcome":{"success":true,"answer_valid":true,"citations_valid":true,"termination_reason":"任务完成"},"provenance":{"policy":"填写模型编号","tool_schema_version":"1.0","prompt_version":"v1","license":"CC0-1.0"}}
```

`rationale_type` 用于标记简短、可核验的决策说明，不代表要保存或训练隐藏思维链。工具观察应保存真实环境返回值，证据 ID 应保持稳定。
