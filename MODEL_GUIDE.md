# claw-eval 测试指南

## 0. 先确认模型接口能通

仓库根目录的 `models.py` 默认按 `models.md` 当前全部 4 个模型做 smoke test：

```bash
python ../models.py smoke
```

> 备注：`python ../models.py smoke` 现在就会默认测完 `models.md` 当前 4 条配置。

## 1. 安装依赖

```bash
uv venv --python 3.11
source .venv/bin/activate
uv pip install -e .
```

如果需要沙箱任务：

```bash
uv pip install -e '.[sandbox,mock,dev]'
bash scripts/test_sandbox.sh
```

## 2. 单模型先跑一个小样

`claw-eval` 支持直接用命令行覆盖 `config.yaml` 里的模型参数，所以不用反复手改配置文件。

示例（把下面 3 个值替换成 `python ../models.py list` 打印出的内容）：

```bash
uv run claw-eval batch \
  --config config.yaml \
  --model 'Qwen3.5-35B-A3B' \
  --base-url 'http://103.237.28.246:2222/v1' \
  --api-key 'EMPTY' \
  --sandbox \
  --trials 1 \
  --parallel 4
```

建议先把 `--trials 1 --parallel 4` 跑通，再升到正式配置。

## 3. 正式跑法

README 里的正式建议是 3 次 trial：

```bash
uv run claw-eval batch \
  --config config.yaml \
  --model '<MODEL_NAME>' \
  --base-url '<BASE_URL>' \
  --api-key '<API_KEY>' \
  --sandbox \
  --trials 3 \
  --parallel 16
```

## 4. 四个模型分别跑

推荐按这个顺序：

1. `python ../models.py smoke`
2. 记下 4 个模型的 `model_name/base_url/api_key`
3. 每个模型各执行一次上面的 `uv run claw-eval batch ...`
4. 结果看 `traces/` 和最终终端 summary

## 5. 常见注意点

- `base_url` 这里填的是 `.../v1`，不要自己再拼 `/chat/completions`。
- judge 仍然走 `config.yaml` 里的配置；如果你只想先测主模型链路，可先看是否需要关闭 judge。
- 如果是 API 抖动导致 trial 不完整，README 说明需要补齐到 3 条有效轨迹。
