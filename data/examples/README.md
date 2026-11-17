# JSONL examples

These are synthetic schema examples only. Replace `asset://...` references with access-controlled locations. Never commit confidential source documents, production logs, credentials, or licensed benchmark test data.

## Benchmark example

```json
{"id":"bench-0001","split":"dev","task_type":"single_doc_qa","source":{"name":"synthetic_contract_v1","license":"CC0-1.0","source_uri":"internal://synthetic/contract-17"},"doc_ids":["doc-17"],"assets":[{"uri":"asset://doc-17.pdf","sha256":"REPLACE_WITH_SHA256","media_type":"application/pdf","pages":2}],"question":"What is the notice period for termination?","target":{"answer":"30 days","answer_type":"text","abstain":false},"evidence":[{"doc_id":"doc-17","page":2,"bbox_1000":[104,311,864,392],"text_span":"Either party may terminate this Agreement with thirty (30) days' written notice.","table":null,"cells":[]}],"validation":{"answer_rule":"normalized_exact_match","evidence_must_entail_answer":true},"metadata":{"language":"en","document_type":"contract","difficulty":"easy","noise":[]}}
```

## SFT trajectory example

```json
{"id":"sft-0001","split":"train","task_type":"evidence_grounded_qa","doc_ids":["doc-17"],"messages":[{"role":"user","content":"Find the termination notice period and cite the page."},{"role":"assistant","content":"I will search the document for the termination clause.","rationale_type":"brief_decision_note"},{"role":"assistant","tool_call":{"name":"search_document","arguments":{"doc_id":"doc-17","query":"termination notice period"}}},{"role":"tool","name":"search_document","content":{"hits":[{"evidence_id":"ev-2-4","page":2,"text":"Either party may terminate this Agreement with thirty (30) days' written notice."}]}},{"role":"assistant","content":"The notice period is 30 days (page 2).","citations":["ev-2-4"]}],"target":{"answer":"30 days","evidence_ids":["ev-2-4"],"abstain":false},"metadata":{"generator":"human_verified","tool_schema_version":"1.0","license":"CC0-1.0"}}
```

## Agentic RL episode example

```json
{"episode_id":"rl-0001-rollout-02","task_id":"task-0001","split":"train","environment":{"name":"smartdoc_local_v1","snapshot":"sha256:REPLACE_WITH_ENV_HASH","seed":17},"task":{"request":"Compare the renewal period in both contracts and provide page citations.","doc_ids":["contract-a","contract-b"]},"trajectory":[{"step":0,"observation_ref":"obs://start","action":{"type":"tool_call","name":"search_document","arguments":{"doc_ids":["contract-a","contract-b"],"query":"renewal period"}},"observation_ref_next":"obs://search-1","labels":{"tool_choice":"correct","arguments":"valid","evidence":"partial","continue":"yes"},"reward":{"terminal":0.0,"process":0.2,"cost":-0.01}},{"step":1,"observation_ref":"obs://search-1","action":{"type":"final","answer":"Contract A renews annually (p. 4); Contract B renews every two years (p. 7).","citations":["contract-a:p4:ev3","contract-b:p7:ev1"]},"observation_ref_next":"obs://terminal","labels":{"answer":"correct","citations":"supported","stop":"correct"},"reward":{"terminal":1.0,"process":0.8,"cost":0.0}}],"outcome":{"success":true,"answer_valid":true,"citations_valid":true,"termination_reason":"completed"},"provenance":{"policy":"REPLACE_WITH_MODEL_ID","tool_schema_version":"1.0","prompt_version":"v1","license":"CC0-1.0"}}
```

The `rationale_type` field is for short, auditable decision notes. It is not a request to store or train on hidden chain-of-thought. Store tool observations as environment outputs and keep stable evidence IDs.
