## 解决3090服务器无法编译 sparse-DETR、 Deformable-DETR算子：
将 
```
export TORCH_CUDA_ARCH_LIST="8.0"
```
插入到 ~/.bashrc 中，位置任意，结果如下所示
```
# sources /etc/bash.bashrc).
#if [ -f /etc/bash_completion ] && ! shopt -oq posix; then
#    . /etc/bash_completion
#fi

export TORCH_CUDA_ARCH_LIST="8.0"

# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$('/opt/conda/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/opt/conda/etc/profile.d/conda.sh" ]; then
        . "/opt/conda/etc/profile.d/conda.sh"
    else
        export PATH="/opt/conda/bin:$PATH"
    fi
fi
unset __conda_setup
# <<< conda initialize <<<
```
此时再运行编译的 setup.py 文件发现可以正常编译算子。
