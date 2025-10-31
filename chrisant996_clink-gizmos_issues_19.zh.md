# [[功能] 在 `fzf.lua` 中添加 ripgrep fzf 集成](https://github.com/chrisant996/clink-gizmos/issues/19)

## 描述

我最近开始使用 Linux。在 Arch 上，我使用了这个功能（https://junegunn.github.io/fzf/tips/ripgrep-integration/，或 https://github.com/junegunn/fzf/issues/2789）来做 `ripgrep+fzf`。这非常有用且强大！

我仍然喜欢 Cmder 和 clink。我尝试在 GPT-4 的帮助下修改了 [fzf.lua](https://github.com/chrisant996/clink-fzf/blob/main/fzf.lua)，然后使用 clink 进行了测试运行。

```lua
-- luacheck: globals fzf_live_grep
function fzf_live_grep(rl_buffer, line_state)
	local function get_cwd()
		local p = io.popen("cd")
		if not p then
			return "."
		end
		local cwd = p:read("*l")
		p:close()
		return cwd or "."
	end

	local cwd = get_cwd()
	print("当前工作目录: " .. cwd)

	local rg = "rg.exe --column --line-number --no-heading --color=always --smart-case {q} || :"
	local preview = "bat.exe --style=numbers --color=always --highlight-line {2} {1}"

	local cmd = string.format(
		"fzf.exe --ansi --disabled --multi "
			.. '--bind "start:reload:%s" '
			.. '--bind "change:reload:%s" '
			.. '--bind "enter:accept" '
			.. "--delimiter : "
			.. '--preview "%s" '
			.. "--preview-window=up:60%%:wrap",
		rg,
		rg,
		preview
	)

	local input_query = rl_buffer:getbuffer()
	if input_query and #input_query > 0 then
		cmd = cmd .. ' --query "' .. input_query:gsub('"', '\\"') .. '"'
	end

	print("正在运行的命令: 2>nul " .. cmd)

	local r = io.popen("2>nul " .. cmd)
	if not r then
		rl_buffer:ding()
		return
	end

	-- 读取所有选中的行（支持多重选择）。
	local selected = {}
	for line in r:lines() do
		-- 移除尾部新行字符和空白
		line = line:gsub("[\r\n]+", "")
		if #line > 0 then
			table.insert(selected, line)
		end
	end
	r:close()

	if #selected == 0 then
		-- 没有选择，刷新提示行。
		rl_buffer:refreshline()
		return
	end

	rl_buffer:beginundogroup()
	rl_buffer:remove(0, -1)
	rl_buffer:endundogroup()
	rl_buffer:refreshline()

	-- 现在为每个选中的文件打开 nvim。
	-- 从选中的行中提取文件名，因为每行都包含 `file:line:col:...`
	-- 我们通常想要的是第一个冒号之前的文件路径
	local files = {}
	for _, line in ipairs(selected) do
		local file = line:match("^(.-):%d+:")
		if file and #file > 0 then
			table.insert(files, file)
		else
			-- 后备方案：使用整行，或忽略无效行
			table.insert(files, line)
		end
	end

	if #files > 0 then
		-- 构建命令以使用所有选中的文件启动 nvim。
		local nvim_cmd = "nvim"
		for _, file in ipairs(files) do
			nvim_cmd = nvim_cmd .. " " .. string.format('"%s"', file)
		end

		-- 可选：如果您希望从 rg 输出中在特定行打开，
		-- 您可以解析选中行的行号并附加 +{line}。

		-- 在 fzf 完成后运行编辑器。
		os.execute(nvim_cmd)
	end
end
```
```lua
local function apply_default_bindings()
	if settings.get("fzf.default_bindings") then
                ...
		-- Alt+S
		rl.setbinding([["\M-s"]], [["luafunc:fzf_live_grep"]])
	end
end
```

<img width="1071" height="612" alt="图像" src="https://github.com/user-attachments/assets/c93565e3-11c4-4597-9e54-f0c7dc95775c" />

看起来它可以工作，但 GPT-4 编写的代码存在一些问题，例如：某些搜索结果没有被定位和高亮，仅显示了前几行。

真的希望能在 Windows 10 上做到这一点！

## 评论

### 评论 1 由 chrisant996

你可以给我一个简短的 TLDR 描述你在请求什么吗？

“ripgrep+fzf”这个短语对我来说不够清楚，很难逆向工程出它的含义。

我开始阅读你分享的链接，但它非常长，没能切中要点：它究竟做了什么，如何调用？

我可能愿意对此进行处理，但在我能够判断这是否是我投入时间精力的事情之前，我不想进行大量的研究来弄清楚请求是什么。

### 评论 2 由 scillidan

感谢你的回复！

我很高兴能更清楚地重新描述它：

我想要在 [fzf.lua](https://github.com/chrisant996/clink-gizmos/blob/main/fzf.lua) 中实现一个互动文本搜索工具/函数。它主要需要 `fzf`、`ripgrep`、`bat`。它基本上再现了 [这个功能](https://junegunn.github.io/fzf/tips/ripgrep-integration/#wrap-up)，但需要在 Windows 10 上与 clink 一起工作。

它实际上将执行：

1. 使用类似 `Alt+S` 的键绑定启动带预览窗口的 `fzf` 搜索。
2. 用户可以输入查询，在当前工作目录中（包含隐藏文件）使用 `rg`（ripgrep）进行递归文本搜索。
3. `fzf` 将实时对找到的文件进行互动模糊过滤，用户可以在预览窗口中查看文件的匹配行。预览将具有语法高亮和匹配行高亮。
4. 用户可以选择一个或多个结果，然后直接在匹配行和字符位置（如果编辑器支持）中使用 `EDITOR` 变量打开它们。
5. 在同一时间打开编辑器时，它将退出此文本搜索功能。
6. 用户还可以使用类似 `Ctrl+c` 的键绑定退出功能。

我录制了一段视频，展示了我在 Linux 上使用此工具的情况：

https://github.com/user-attachments/assets/51ae7a4b-191a-4561-a0f5-04dd89c79e09

这实际上是在此视频中写的功能（包括修复以在 `nvim`、`subl` 中打开多个选中文件以及一些键绑定修改）。仅供参考：

```bash
rfs() {
  local RELOAD='reload:rg --column --color=always --smart-case {q} || :'
  local OPENER='if [[ $FZF_SELECT_COUNT -eq 0 ]]; then
                nvim {1} +{2}
             else
                local cmds=()
                while read -r line; do
                  local file=$(echo "$line" | cut -d: -f1)
                  local lineno=$(echo "$line" | cut -d: -f2)
                  cmds+=( "+call cursor($lineno,1)" "$file" )
                done < <(cat {+f})
                nvim "${cmds[@]}"
             fi'
  local OPENER_SUBL='if [[ $FZF_SELECT_COUNT -eq 0 ]]; then
                    subl {1}:{2}
                  else
                    params=()
                    while read -r line; do
                      file=$(echo "$line" | cut -d: -f1)
                      line_no=$(echo "$line" | cut -d: -f2)
                      col_no=$(echo "$line" | cut -d: -f3)
                      params+=( "${file}:${line_no}:${col_no}" )
                    done < <(cat {+f})
                    subl "${params[@]}"
                  fi'
  fzf --disabled --ansi --multi \
      --bind "start:$RELOAD" --bind "change:$RELOAD" \
      --bind "enter:become:$OPENER" \
      --bind "ctrl-o:execute:$OPENER" \
      --bind "ctrl-e:execute:$OPENER_SUBL" \
      --bind 'ctrl-u:preview-up,ctrl-d:preview-down,alt-a:select-all,alt-d:deselect-all,ctrl-/:toggle-preview' \
      --delimiter : \
      --preview 'bat -n --theme=base16 --color=always --highlight-line {2} {1}' \
      --preview-window '~1,+{2}+4/3,<80(up)' \
      --query "$*"
}
```

### 评论 3 由 chrisant996

> 我录制了一段视频，展示了我在 Linux 上使用此工具的情况：

在视频中，没有使用补全。因此 fzf.lua 并不会涉及到视频中显示的内容。

但是，也许一些地方还不太清楚：
- 你是说你想要的视频中的内容，只是想按 <kbd>Alt</kbd>-<kbd>S</kbd> 而不是运行 `rfs`？
- 还是说你希望视频中的那种东西，但你想要一个_新的和不同的使用模式_，将 `rfs` 的功能集成到实际的补全中，因此你将输入 `somecommand `<kbd>Alt</kbd>-<kbd>S</kbd>，它会启动 fzf 进行补全——但始终仅为当前目录中的文件——并在标记一个或多个文件后按 <kbd>Enter</kbd>，然后标记的文件名将被插入到 CMD 输入行中，你可以按 <kbd>Enter</kbd> 在 CMD 中运行你构造的任何输入行？（即进行实际的补全，而不是仅限于直接启动 `nvim`）

如果你想要前者，即视频中的确切内容，那么 fzf.lua 并不相关，而 Clink 相关的唯一方式是因为你说你想将其绑定到键，而不是输入命令。

如果你想要后者，那么主要问题只是为此自定义 `fzf`。然后在 fzf.lua 中进行一些小的调整，将所有内容连接到补全中。

### 评论 4 由 chrisant996

哦，另外请注意：在从 bash 转到 CMD 时，必须小心避免被称为“注入攻击”的问题：

bash 集成脚本会直接将用户的输入传递给 `rg`。这在 bash 中是可以的，因为 bash 会自动转义参数，以确保该参数被传递给 `rg` 而不被 shell 处理。例如，输入 `hello & del c:\*` 最终将得到一个字符串 `hello & del c:\*` 被 `rg` 接收，而不是将命令行解析为两个命令，即 `rg hello` 和 `del c:\*` 并同时执行。显然，如果 `del c:\*` 的部分被 shell 运行，那将是糟糕的。

CMD 不会自动转义。因此，如果你简单地调用 `rg {user_input}` 而用户输入是 `hello & del c:\*`，那么你就会遇到问题。因为命令行将是 `rg hello & del c:\*`，而 CMD 将把其解析为两个独立的命令 `rg hello` 和 `del c:\*` 并执行它们。引用并不足够，例如 `rg "{user_input}"` 在各种情况下仍然可能出现问题（尤其是当用户输入本身包含 `"` 字符时）。

我记得在 fzf.lua 中有一个函数可以安全转义用户输入，我认为它必须简单地删除几个特定字符以保证安全。但是这个函数不能在这里使用，因为运行 `rg` 的命令行是在 fzf 内部构建的，而不是由 fzf.lua 脚本构造的。

因此，在 Windows 上为此建立一个安全正确的 fzf 配置还有其他一些问题需要解决。

**更新：**哦，其实在使用 bash 时也可能存在注入攻击的问题——因为bash并不是传递参数，而是 fzf 本身正在构建该字符串。不过，也许 fzf 在插入 `{q}` 时会应用 bash-like 的转义？如果是这样，那么它无法在 Windows 上做到，因为 Windows 命令行语法不同于 Unix/Linux，没有相同的转义机制。

### 评论 5 由 chrisant996

而 AI 生成的脚本则是奇怪的：它将其集成到补全中以启动它，但不进行实际补全，并强制直接启动 `nvim`，并擦除了输入行。如果不需要补全，那么该脚本甚至不是正确的解决方案（更不用说脚本中的任何编码问题）。

要么 AI 从一开始就走错了方向，根本不应该使用该脚本，要么如果确实希望有补全（而 fzf 示例代码在 Linux 中并没有做到这一点），那么脚本中的实现非常不正确。

不过，我不知道你想要哪种功能。

### 评论 6 由 chrisant996

Clink 似乎没有参与。所以这可能是与此库无关的主题。

<br/>

无论如何：

## 示例命令行

下面的命令行似乎执行了您所描述的内容。我基本上只是使用了 [使用 fzf 作为互动 ripgrep 启动器](https://github.com/junegunn/fzf/blob/master/ADVANCED.md#using-fzf-as-interactive-ripgrep-launcher) 中的内容，进行了一些非常小的调整以适应 Windows。

为了可读性，在下面的示例中，我插入了换行符和悬挂缩进，但要实际执行命令行，应该将其作为一行执行，没有换行。

```cmd
fzf --ansi --disabled --delimiter : 
        --bind "change:reload:rg --column --line-number --no-heading --color=always --smart-case {q} || call;" 
        --bind "enter:become(nvim {1} +{2})" 
        --preview "bat --color=always {1} --highlight-line {2}" 
        --preview-window "up,60%,border-bottom,+{2}+3/3,~3" 
```

> [!注意]
> 它似乎并没有遭受注入攻击的可能性，至少在简单尝试中是如此。
>
> 但我没有尝试复杂的输入序列，因此仍然可能存在注入攻击问题。当我说“注入攻击”时，我并不是说它会成为“一次攻击”，我只是使用这个模式的名称，其中输入文本的一部分被误解为命令指令。
>
> 如果你输入的内容意外删除文件，甚至会让人感觉像是“攻击”，即使这只是意外自我造成的。😉

## 示例脚本

你会希望它处于可重用的脚本中，因此更像这样：

**rfs.cmd**
```cmd
rem --------------------------------------------------------------------------
rem 示例脚本展示如何在 Windows 上集成 ripgrep 和 fzf。
rem 源自 https://github.com/junegunn/fzf/blob/master/ADVANCED.md#using-fzf-as-interactive-ripgrep-launcher
rem --------------------------------------------------------------------------

rem -- 确保本地环境变量不会污染 shell 会话。
setlocal

rem -- 如果定义了 %EDITOR%，则使用 %EDITOR%，否则回退到 notepad.exe。
if defined EDITOR set EDITEXE=%EDITOR%
if not defined EDITOR set EDITEXE=notepad.exe

rem -- 调用 rg.exe 和 bat.exe 的标记。
set RGEXE=rg.exe --column --line-number --no-heading --color=always --smart-case
set BATEXE=bat.exe --color=always

rem -- 启动 fzf 的配置。
set FLAGS=--ansi --disabled --delimiter :
set BIND_START=--bind "start:reload:%RGEXE% {q}"
set BIND_RELOAD=--bind "change:reload:%RGEXE% {q} || call;"
set BIND_ENTER=--bind "enter:become(%EDITEXE% {1} +{2})"
set PREVIEW=--preview "%BATEXE% {1} --highlight-line {2}"
set PREVIEW_WINDOW=--preview-window "up,60%%,border-bottom,+{2}+3/3,~3"
set QUERY=--query "%*"

rem -- 启动 fzf。
fzf.exe %FLAGS% %BIND_START% %BIND_RELOAD% %BIND_ENTER% %PREVIEW% %PREVIEW_WINDOW% %QUERY%
```

运行 `rfs` 不带初始查询。
运行 `rfs blah` 时将“blah”作为初始查询启动。

## 有什么问题？

> 看起来可以工作，但 GPT-4 编写的代码存在一些问题，例如某些搜索结果没有被定位和高亮，仅显示了前几行。

你需要提供你遇到的问题示例。

## 你是想要的那样吗？

如果是，你只需将其绑定到 <kbd>Alt</kbd>-<kbd>s</kbd> 在 Clink 中，然后你只需在你的 .inputrc 文件中添加一个键绑定。

**.inputrc**
```inputrc
# 将 ALT-s 绑定以在输入行开头插入 'rfs '，然后运行输入行。
# '\e[H' 是 HOME 键的 VT 序列。
"\es": "\e[Hrfs \n"
```

## 还是说这不是你想要的？

或者你是想要一些不同于 fzf 文档描述的内容？
也许你想要一个 fzf 文档没有覆盖的新东西？
也许你希望实际的补全，以便你可以执行以下操作：

1. 在 CMD 输入行中输入 `echo blah`<kbd>Alt</kbd>-<kbd>S</kbd>
2. 然后 fzf 启动并在当前目录中用“blah”运行 ripgrep
3. 如果你选择条目并按 <kbd>Enter</kbd>，则实际补全将发生
4. 实际补全将用 `echo file1 file2 etc` 替换输入行，而不是立即执行输入行，即你可以继续编辑输入行。

如果你想要这个，那么在 fzf 文档中似乎没有涵盖，而且在 Unix/Linux 中似乎提供不了。
这 certainly 可连接，但由于你未描述或 fzf 文档也没有描述，因此我假设这不是你想要的。

如果这确实不是你想要的，那么问题就与 fzf.lua 无关，与 Clink 的关系也仅限于如何绑定键。

### 评论 7 由 chrisant996

> [!注意]
> 我编辑了之前的回复，将 `|| rem` 改为 `|| call;` 以清除 errorlevel。显然在 bash 中 `:` 有一个副作用，可以清除退出码，但在 CMD 中，`rem` 或 `::` 不会清除退出码。

### 评论 8 由 chrisant996

而且 `+{2}` 之类的东西似乎是特定于 `nvim` 的参数语法，用于跳转到指定行号，因此这些东西不适用于其他程序。

要使其按您希望的方式工作以及适用于您想要的特定程序，将需要各种自定义；没有通用的方法可以告诉不同程序打开文件并跳转到特定行号。

### 评论 9 由 scillidan

非常感谢你详尽的解释和指导！

我需要稍后仔细阅读以完全理解。但我首先尝试了 `rfs.cmd` 发现它工作得很好。

之前我多次尝试让 GPT-4 编写 `.bat` 脚本，但反复失败。后来，我想 fzf.lua 可以很好地集成一些 fzf 功能，因此我粗略假设它可以实现这一点。但使用 `rfs.cmd` 更好。

我将尝试使用 WezTerm 的配置将一个键绑定到运行 `rfs.cmd`。我的问题已经完全解决。

感谢您花了这么多时间！祝您有愉快的一天🌞！

[GPT-4o mini]
