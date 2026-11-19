# 生图做目标图 / moodboard（可选）

Phase 1 可选项：生图做目标图或 moodboard 当质量锚点。火山 Seedream，需 `ARK_API_KEY` 环境变量。URL 24 小时失效，**必须当场下载**：

```bash
python3 - <<'EOF'
import json, os, urllib.request
# 先替换两个占位符
PROMPT = '<目标图/moodboard 的描述>'
OUT_PATH = '<保存路径>.png'
key = os.environ.get('ARK_API_KEY')
if not key:
    raise SystemExit('需先设置 ARK_API_KEY 环境变量')
req = urllib.request.Request(
    'https://ark.cn-beijing.volces.com/api/v3/images/generations',
    data=json.dumps({'model': 'doubao-seedream-5-5-261028',
                     'prompt': PROMPT, 'size': '1024x1024'}).encode(),
    headers={'Authorization': f'Bearer {key}', 'Content-Type': 'application/json'})
resp = json.load(urllib.request.urlopen(req, timeout=180))
open(OUT_PATH, 'wb').write(urllib.request.urlopen(resp['data'][0]['url'], timeout=120).read())
print('saved', OUT_PATH)
EOF
```

不需要时可整段跳过，流程不依赖此步。
