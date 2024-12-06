# comfyui 节点开发 code test

按照要求提交适配 ComfyUI-Easy-Use 节点（ https://github.com/yolain/ComfyUI-Easy-Use ）的 Pull Request 到本仓库


## 背景知识

https://docs.comfy.org/essentials/custom_node_server_overview


## 目标

将comfyui执行过程中所需的模型文件提前下载，并建立加载映射关系，避免节点临时加载模型。

## 基本原理
公司内挂载了一块大共享盘，把 custom node 需要的模型，依赖等数据提交下载到里面了。
这样启动新节点时不用再下载，提升了启动速度，降低了存储成本。
comfyui主仓库：https://github.com/liblib-co-work/ComfyUI

为了实现这一目标，一般来讲需要修改comfyui主仓库和节点仓库代码的几个地方：
1. 修改 custom node 原代码，通过调用公司封装到 comfyUI 里的函数来从共享盘加载模型/依赖，不用额外下载
   ```python
   
    def get_juicefs_endpoint():
        """获取juicefs endpoint"""
        return "/ComfyUI/std-models"
    def get_juicefs_path(file_path):
        """获取juicefs文件路径"""
        return get_juicefs_endpoint() + "/" + file_path
    def get_juicefs_full_path_safemode(mappings, model_name: str):
        """根据mappings和model name获取juicefs的文件路径
        mappings：目标mapping
        model_name: mapping的key
        """
        model_path = mappings.get(model_name, "")
        if model_path:
            model_path = get_juicefs_path(mappings[model_name])
        else:
            raise ValueError(str(model_name) + " does not exist in mapping")
        return model_path
 
   ```
调用example（下图1间接调用，2直接调用了）：
      ![image](https://github.com/user-attachments/assets/0bd6997a-74e3-4c53-b079-8d10f5a91b2e)
      图1
      ![image](https://github.com/user-attachments/assets/b6f77037-1cc6-4414-abc3-ed86cd91b32d)
      图2


2. 修改 comfyUI 的配置（主要关注 configs/node_fields.py 中的 PULID_FLUX_MAPPINGS 和 NODE_FILE_FIELDS 两处修改即可）
      ![image](https://github.com/user-attachments/assets/71ee6ced-0f64-47fd-8a9b-04b4e9d86044)
      ![image](https://github.com/user-attachments/assets/ab03c303-a841-4932-9c92-886d0b287b68)

3. 如果源 node 需要从 github 下载代码，需要参考如下方式将代码提前下载好，并注释掉 requirement.txt 中对应的逻辑，改为 -e 从本地安装
      ![image](https://github.com/user-attachments/assets/0837bc5e-1918-4703-bb7a-fe5870349b69)


