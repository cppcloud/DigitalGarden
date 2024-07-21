---
{"dg-publish":true,"tags":["tools"],"permalink":"/Tools/python包管理器uv/","dgPassFrontmatter":true}
---


## 安装miniconda3

**提供多个版本的Python给uv使用，而不是真的要用conda**

```bash
scoop install miniconda3
```

## 安装下uv：

```bash
scoop install uv
```

### 创建常用的 python 环境

uv的虚拟环境并不像conda一样能提供多版本的Python。可以借用conda的python。因此创建几个conda环境(为了给uv提供不同版本的python)

```bash
conda create -n py9 python=3.9
conda create -n py10 python=3.10
conda create -n py11 python=3.11
conda create -n py12 python=3.12
```

## 设置取消自动激活conda环境

因为这会与uv虚拟环境冲突：

```bash
conda config --set auto_activate_base false
```

## 替换conda

使用conda，一般其流程是
```bash
conda create -n gpt python=3.10
conda activate gpt
python -m pip install -r requirements.txt
python main.py
```

使用 `uv` 的话，其流程就是

```bash
conda activate py10
uv venv --seed -p 3.10
conda deactivate
uv pip install -r requirements.txt
source ./.venv/bin/activate
python main.py
```