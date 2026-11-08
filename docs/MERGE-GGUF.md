# 合并 GGUF 分卷（GitHub Release 单个附件上限 2GB，故按卷发布）

每个模型被切成 2 个分卷：`*.gguf.part01`、`*.gguf.part02`。

## Windows（cmd / PowerShell）

```cmd
copy /b dawcoder-grpo-v3-Q4_K_M.gguf.part01 + dawcoder-grpo-v3-Q4_K_M.gguf.part02 dawcoder-grpo-v3-Q4_K_M.gguf
```

PowerShell：

```powershell
$out = [System.IO.File]::Create("dawcoder-grpo-v3-Q4_K_M.gguf")
foreach ($p in Get-ChildItem "dawcoder-grpo-v3-Q4_K_M.gguf.part*" | Sort-Object Name) {
    $in = [System.IO.File]::OpenRead($p.FullName)
    $in.CopyTo($out); $in.Close()
}
$out.Close()
```

## Linux / macOS

```bash
cat dawcoder-grpo-v3-Q4_K_M.gguf.part* > dawcoder-grpo-v3-Q4_K_M.gguf
```

## 校验

合并后核对 `SHA256SUMS.txt` 中对应整卷的 sha256：

```powershell
Get-FileHash .\dawcoder-grpo-v3-Q4_K_M.gguf -Algorithm SHA256
```
