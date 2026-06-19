# HTTP 模型接入指南（ML Backend）

平台「模型管理」**只对接外部 HTTP 推理服务**（对标 Label Studio 的 ML Backend）：
不上传模型权重文件，只登记一个服务端点 URL。推理时平台把数据 POST 给你的服务，
取回 JSON。模型推理与平台主进程彻底隔离，可用任意框架/语言实现、独立扩容、上 GPU。

## 整体流程

```
模型管理「接入模型服务」→ 填 http(s) 端点 → 登记(ModelRegistry.framework='http')
        │
        ├─ 在线试运行：POST /model-registry/{id}/invoke  → 转发到你的端点，看返回
        │
数据处理任务(processing_mode=model, model_id=该模型)
        │  ModelPipeline 逐条 → HttpAdapter POST {inputs, meta} → 你的端点
        │  你的返回 JSON 原样写回 dataset_items.media_metadata['model_processing'].output
        └─ 「一键沉淀为数据集」导出处理结果

标注任务自动审核(config.auto_review.model_id=该模型)
        └─ 逐条标注 POST 给你的端点；从返回里读 score/confidence 判定放行/驳回/留人工

标注任务模型预标注(POST /tasks/{id}/run-model-annotation)
        └─ 逐条数据 POST 给你的端点；从返回的 annotation 字段(平台原生 annotation_data)
           写进 annotations 表 → 标注员打开数据项时作为"起标草稿"展示、改完提交即覆盖
```

## 接口契约

### 请求（平台 → 你的服务）

`POST <你的端点>`，`Content-Type: application/json`：

```json
{
  "inputs": { "...": "..." },
  "meta":   { "...": "..." }
}
```

`inputs` 形态：
- **图像样本**：平台自动读图、base64 后随包发出：`{"image_b64": "<base64>", "filename": "xxx.jpg"}`。
- **非图像样本（音频/视频/点云/文档等）**：平台给出**可下载 URL** + 元数据：
  `{"file_url": "http://host/data/files/raw/...", "file_path": "...", "file_name": "...", "file_type": "..."}`，
  你的服务自行下载 `file_url` 处理。
  > `file_url` 要外部可达需配置 model-service 的环境变量 `PUBLIC_FILE_BASE_URL`（平台对外源，如
  > `http://your-host:8080`）；未配置时发相对路径，仅同源/同网段远程可用。文本类若已内联，
  > 字段也可能直接带 `text`。

`meta`：来自 `task.config.inference` 的业务参数。注意以下**传输参数会被平台消费掉、不会进 meta**：
`endpoint`（覆盖 URL）、`method`（默认 POST）、`headers`（附加请求头，如鉴权）、`timeout`（秒，默认 30）。

### 响应（你的服务 → 平台）

HTTP `200` + JSON。整段会被原样写入 `dataset_items.media_metadata['model_processing'].output`。
建议结构（自动审核会从中读取整体置信度，认 `score`/`confidence`/`prob`/`probability`）：

```json
{
  "result": [
    {"label": "cat", "score": 0.93, "bbox": [12, 30, 80, 120]}
  ],
  "score": 0.93
}
```

- 非 `200` 或非 JSON：该条记为失败（写 `model_processing_error`），不阻塞整体。
- `result`：用于「数据处理任务」落库（写 `media_metadata.model_processing.output`）与自动审核取分。
- `score`/`confidence`/`prob`/`probability`：自动审核读取的整体置信度。
- `annotation`：用于「模型预标注」，见下。

## 模型预标注（统一通路）

触发：`POST /tasks/{task_id}/run-model-annotation`（body 可带 `{"model_id": N}`，缺省读
`task.config.pre_annotation.model_id`）。平台逐条把数据 POST 给你，期望返回里带 **`annotation`**：

```json
{ "annotation": <该数据项的"平台原生 annotation_data">, "score": 0.9 }
```

关键点——**平台不翻译，原样写库**，所以 `annotation` 必须就是对应标注类型工作台保存的
`annotation_data` 结构（检测框/多边形/关键点/分类/NER span/… 各类型不同）。这套机制因此
对 30+ 标注类型一次性通吃，"按类型的知识"放在你的服务里。

写入行为：
- 写成 `annotations` 行，`annotator_id=NULL`（标记模型来源）、`is_latest`、带版本，可重复跑（旧的自动退役）。
- **不**改 `task_item_status`：数据项仍是"待人工"。标注员打开数据项时，若自己尚无标注，
  平台把模型预标注作为**起标草稿**展示；人工提交后，模型预标注退役、以人工为准。
- 取不到 `annotation`（也兼容 `annotations` / 顶层就是列表）的条目记为失败，不阻塞整体。

> 想知道某类型 `annotation_data` 长什么样：最省事是先人工标一条，调
> `GET /tasks/{id}/annotations/by-item/{item_id}` 拿到真实结构当模板。

### 各标注类型 annotation_data 模板

`annotation` 是一个**列表**，元素结构按类型如下（字段名平台做了多种兼容，给最常用的一种）：

| 标注类型 | 列表元素示例 |
|---|---|
| 图像检测框 | `{"type":"detection","label":"car","x":40,"y":30,"width":120,"height":90,"score":0.9}`（也认 `bbox:[x,y,w,h]` 或 `w/h`） |
| 图像分割多边形 | `{"type":"polygon","label":"road","points":[{"x":40,"y":30},{"x":160,"y":30},{"x":100,"y":120}]}` |
| 车道线 | `{"type":"polyline","label":"lane","points":[{"x":..,"y":..}, ...]}`（type 用 `polyline`/`lane`/`line`） |
| 圆/椭圆 | `{"type":"ellipse","label":"x","x":40,"y":30,"width":80,"height":80}` |
| 关键点/姿态 | `{"type":"keypoint","label":"person","bbox":[40,30,120,160],"keypoints":[{"x":100,"y":50,"v":2,"name":"nose"}, ...]}`（v=可见性0/1/2） |
| 图像/文本分类 | `{"type":"classification","label":"cat","score":0.9}` |
| 文本实体(NER) | `{"type":"ner","start":0,"end":3,"label":"PER","text":"张三"}`（按字符下标） |
| OCR | `{"type":"ocr","text":"识别文字","x":..,"y":..,"width":..,"height":..}` |
| 关系抽取 | `{"type":"relation","from":<实体id>,"to":<实体id>,"label":"属于"}` |

坐标单位为**原图像素**。其余类型（对话/QA/文本生成/RLHF/表格/知识图谱/音频片段/3D/视频等）
同理带各自 `type` 字段——不确定就用上面的"先标一条照抄"法。

> 可运行示例见 `data-annotation-model/examples/`（`ml_backend_example.py` 零依赖、含各类型模板；
> `ml_backend_fastapi.py` 贴近生产）。

## 并发

数据处理 / 模型预标注按**有界线程池并行**调用你的服务（远程 HTTP 是 IO 密集，DB 写入仍由平台主线程串行）。
并发度优先级：`task.config.inference.concurrency` > 环境变量 `MODEL_PIPELINE_CONCURRENCY` > 默认 4（限幅 1~32）。
请确保你的服务能承受相应并发；扛不住就调小，或在服务端限流。

## 鉴权 / 超时

在任务的 `config.inference` 里配置，会作为 `meta` 的传输参数透传：

```json
{
  "inference": {
    "headers": {"Authorization": "Bearer <token>"},
    "timeout": 60
  }
}
```

## 快速验证

1. 跑示例后端（零依赖）：
   ```bash
   python data-annotation-model/examples/ml_backend_example.py   # :9090
   ```
2. 「模型管理 → 接入模型服务」端点填 `http://<本机IP>:9090/predict`。
3. 点卡片「试运行」，inputs 填 `{"text":"hello"}`，应返回 `{"result":[...],"score":...}`。
4. 建一个「数据处理任务」选该模型，跑批处理，到数据项详情看 `media_metadata.model_processing`。
