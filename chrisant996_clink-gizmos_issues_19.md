# [[Feature] Add ripgrep fzf integration in `fzf.lua`](https://github.com/chrisant996/clink-gizmos/issues/19)

## Description

I recently started using Linux. On Arch, I used this function (https://junegunn.github.io/fzf/tips/ripgrep-integration/, or https://github.com/junegunn/fzf/issues/2789) to do `ripgrep+fzf`. It's very useful and powerful!

I still like Cmder and clink. I tried to modify [fzf.lua](https://github.com/chrisant996/clink-fzf/blob/main/fzf.lua) with the help of GPT-4, then test runing it with clink.

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
	print("Current working directory: " .. cwd)

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

	print("Running command: 2>nul " .. cmd)

	local r = io.popen("2>nul " .. cmd)
	if not r then
		rl_buffer:ding()
		return
	end

	-- Read all selected lines (multi-select supported).
	local selected = {}
	for line in r:lines() do
		-- Remove trailing newline characters and whitespace
		line = line:gsub("[\r\n]+", "")
		if #line > 0 then
			table.insert(selected, line)
		end
	end
	r:close()

	if #selected == 0 then
		-- No selection made; just refresh prompt line.
		rl_buffer:refreshline()
		return
	end

	rl_buffer:beginundogroup()
	rl_buffer:remove(0, -1)
	rl_buffer:endundogroup()
	rl_buffer:refreshline()

	-- Now open nvim for each selected file.
	-- Extract filename from the selected lines, since each line includes `file:line:col:...`
	-- We want just the file path before first colon (usually)
	local files = {}
	for _, line in ipairs(selected) do
		local file = line:match("^(.-):%d+:")
		if file and #file > 0 then
			table.insert(files, file)
		else
			-- Fallback: use entire line, or ignore invalid lines
			table.insert(files, line)
		end
	end

	if #files > 0 then
		-- Build the command to launch nvim with all selected files.
		local nvim_cmd = "nvim"
		for _, file in ipairs(files) do
			nvim_cmd = nvim_cmd .. " " .. string.format('"%s"', file)
		end

		-- OPTIONAL: if you want to open at a specific line from rg output,
		-- you could parse the line number from the selected line and append +{line}.

		-- Run the editor **after** fzf finishes.
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

<img width="1071" height="612" alt="Image" src="https://github.com/user-attachments/assets/c93565e3-11c4-4597-9e54-f0c7dc95775c" />

It seems to work, but the code by GPT-4 have some problems, such as: some search results that are not be positioned and highlighted, just show the first few lines absolutely.

Really hope can do it on Windows 10!

## Comments

### Comment 1 by chrisant996

Can you give a short TLDR description of what you're asking for? 

The phrase "ripgrep+fzf" isn't enough for me to reverse engineer what that means. 

I started reading the link you shared, but it's very long and was not getting to the point:  what does it actually do, and how does one invoke it?

I might be willing to work on it, but I don't want to have to do a bunch of research to figure out what the request is before I can figure out whether it's something I might invest time in. 

### Comment 2 by scillidan

Thanks for your reply!

I'd be gald to re-describe it more clearly:

I want an interactive text searching tool/function in [fzf.lua](https://github.com/chrisant996/clink-gizmos/blob/main/fzf.lua). It mainly requires `fzf`, `ripgrep`, `bat`. It basically reproduces [this function](https://junegunn.github.io/fzf/tips/ripgrep-integration/#wrap-up) but need works on Windows 10 with clink.

It will actually do:

1. Use a keybind liked `Alt+S` to start `fzf` search with preview window.
2. Then user can type a query to do recursive text search with `rg` (ripgrep) in all files (contain hidden files) on current working directory.
3. In real time, `fzf` will do interactive fuzzy filter for the found files. And user can see the matched lines from files in preview window. The preview will have syntax-highlight and matched-lines hightlight.
4. And user can select one or multiple results, then open them in with `EDITOR` variable directly at the matching lines and character position if editor supported.
5. In same time to do opening in editor, it will exit from this text-search fuctnion.
6. User also can used keybind liked `Ctrl+c` to exit from function.

I record a video that I used this tool on Linux:

https://github.com/user-attachments/assets/51ae7a4b-191a-4561-a0f5-04dd89c79e09


This is actually function's writing in this video (including repairs to open multiple selection files in `nvim`, `subl` and some keybind modifications). Just as a reference:

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

### Comment 3 by chrisant996

> I record a video that I used this tool on Linux:

In the video, completion is not used.  So fzf.lua wouldn't be involved at all for what's shown in the video.

But maybe something isn't clear yet:
- Are you saying you want exactly what's in the video, except that you want to press <kbd>Alt</kbd>-<kbs>S</kbd> instead of running `rfs`?
- Or are you saying you want the type of thing from the video, but you want a _new and different usage pattern_ that would integrate the capability from `rfs` into actual completion so that you would instead type `somecommand `<kbd>Alt</kbd>-<kbd>S</kbd> and it would launch fzf to do completion -- but always only for files from the current directory -- and after marking one or more files in fzf you'd press <kbd>Enter</kbd> and then the marked file names would be inserted into the CMD input line and you could then press <kbd>Enter</kbd> in CMD to run whatever input line you constructed?  (i.e. doing actual completion, instead of being limited to only directly launching `nvim`)

If you want the former, i.e. exactly what's in the video, then fzf.lua isn't relevant, and the only way Clink would be relevant is because you said you want to make a key binding to invoke it instead of typing a command.

If you want the latter, then still the main thing is just coming up with the customizations for `fzf` itself.  And then a little tweaking in fzf.lua to just connect all that into completion.

### Comment 4 by chrisant996

Oh, and watch out:  when converting from bash to CMD, care must be taken to prevent what's known as an "injection attack":

The bash integration scriptlet just passes the user's input directly to `rg`.  That's ok in bash because bash automatically escapes the argument so that the argument is guaranteed to be received by `rg` and _not_ processed by the shell.  E.g. typing `hello & del c:\*` ends up with `rg` receiving a string `hello & del c:\*`, instead of the shell parsing the command line as two commands i.e. `rg hello` & `del c:\*`.  Clearly if the `del c:\*` part got run by a shell, that would be bad.

CMD does not do automatic escaping.  So if you were to literally just invoke `rg {user_input}` and the user input were `hello & del c:\*` then you'd have a problem.  Because the command line would be `rg hello & del c:\*` which CMD would parse into two separate commands `rg hello` & `del c:\*` and execute both of them.  Quoting isn't enough, e.g. `rg "{user_input}"` can still run into problems in various cases (especially if the user input itself contains a `"` character).

IIRC there's a function in fzf.lua that handles safely escaping the user input, and I think it has to simply drop a couple of specific characters in order to guarantee safety.  BUT that function can't be used here, because the command line for running `rg` is constructed from within `fzf`, not by the fzf.lua script.

So, making a safe and correct form of fzf configuration for this on Windows has additional "gotcha" issues that need to be solved.

**UPDATE:** oh actually the injection attack problem may be present even when using bash -- because bash is not what's passing the argument, instead fzf itself is constructing the string.  But maybe fzf applies bash-like escaping when inserting the `{q}`?  If so, then it can't do that on Windows (because Windows command line syntax is different from Unix/Linux and doesn't have the same kind of escaping mechanism).

### Comment 5 by chrisant996

And the script that AI generated is weird:  it integrates into completion for launching it, but it does not perform completion and forces directly launching `nvim`, and it erases the input line.  If completion isn't desired, then the script is not even the right approach (never mind any coding issues in the script).

Either AI is going off in a wrong direction from the start and that script shouldn't be used at all, or if completion truly is desired (which the fzf sample code does not do on Linux) then the implementation in the script is very incorrect.

I don't know which kind of functionally you want, though.

### Comment 6 by chrisant996

Clink does not seem to be involved.  So this is probably off topic in this repo.

<br/>

Anyway:

## Example Command Line

Here is a command line that seems to do essentially what you've described.  I basically just used literally what's at [Using fzf as interactive Ripgrep launcher](https://github.com/junegunn/fzf/blob/master/ADVANCED.md#using-fzf-as-interactive-ripgrep-launcher) from the fzf documentation, with very minor tweaks for Windows.

For readability, in the example below I've inserted line breaks and hanging indents, but to actually execute the command line it should all be a single line with no line breaks.

```cmd
fzf --ansi --disabled --delimiter : 
        --bind "change:reload:rg --column --line-number --no-heading --color=always --smart-case {q} || call;" 
        --bind "enter:become(nvim {1} +{2})" 
        --preview "bat --color=always {1} --highlight-line {2}" 
        --preview-window "up,60%,border-bottom,+{2}+3/3,~3" 
```

> [!CAUTION]
> It does not seem to suffer from the injection attack possibility, at least not with simple attempts.
>
> But I didn't try complex sequences of input, so there still might be an injection attack possible.  And when I say "injection attack" I don't mean that it would be an "attack" per se, I'm just using the name of the pattern where part of input text gets misinterpreted as command instructions.
>
> And if something you type accidentally deletes files or something, then it would _feel_ like an "attack" even if it was accidentally self-inflicted. 😉

## Example Script

You would want it in a reusable script, so something more like this:

**rfs.cmd**
```cmd
rem --------------------------------------------------------------------------
rem Sample script showing how to integrate ripgrep and fzf on Windows.
rem Ported from https://github.com/junegunn/fzf/blob/master/ADVANCED.md#using-fzf-as-interactive-ripgrep-launcher
rem --------------------------------------------------------------------------

rem -- Make sure local environment variables do not pollute the shell session.
setlocal

rem -- Use %EDITOR% if defined, otherwise fall back to notepad.exe.
if defined EDITOR set EDITEXE=%EDITOR%
if not defined EDITOR set EDITEXE=notepad.exe

rem -- Flags for invoking rg.exe and bat.exe.
set RGEXE=rg.exe --column --line-number --no-heading --color=always --smart-case
set BATEXE=bat.exe --color=always

rem -- The configuration for launching fzf.
set FLAGS=--ansi --disabled --delimiter :
set BIND_START=--bind "start:reload:%RGEXE% {q}"
set BIND_RELOAD=--bind "change:reload:%RGEXE% {q} || call;"
set BIND_ENTER=--bind "enter:become(%EDITEXE% {1} +{2})"
set PREVIEW=--preview "%BATEXE% {1} --highlight-line {2}"
set PREVIEW_WINDOW=--preview-window "up,60%%,border-bottom,+{2}+3/3,~3"
set QUERY=--query "%*"

rem -- Launch fzf.
fzf.exe %FLAGS% %BIND_START% %BIND_RELOAD% %BIND_ENTER% %PREVIEW% %PREVIEW_WINDOW% %QUERY%
```

Running `rfs` starts with no initial query.
Running `rfs blah` starts with "blah" as the initial query.

## What Problems?

> It seems to work, but the code by GPT-4 have some problems, such as some search results that are not be positioned and highlighted, just show the first few lines absolutely.

You would need to show examples of problems that you encountered.

## Is That What You Meant?

Is that what you meant?
If yes, and you just want to bind that to <kbd>Alt</kbd>-<kbd>s</kbd> in Clink, then you would just [add a key binding in your .inputrc file](https://chrisant996.github.io/clink/clink.html#keybindings).

**.inputrc**
```inputrc
# Bind ALT-s to insert 'rfs ' at the beginning of the input line and then run the input line.
# The '\e[H' is the VT sequence for the HOME key.
"\es": "\e[Hrfs \n"
```

## Or Maybe That Isn't What You Meant?

Or did you mean you wanted something different from what the fzf docs describe?

Maybe you wanted a new thing which the fzf docs do not cover?
Maybe you wanted actual completion, so that you could do the following?

1. At the CMD input line, type `echo blah`<kbd>Alt</kbd>-<kbd>s</kbd>
2. And have fzf start and run ripgrep with "blah" over the files in the current directory
3. And if you select entries and press <kbd>Enter</kbd> then actual completion would occur
4. Actual completion would replace the input line with `echo file1 file2 etc` _without executing the input line yet_, i.e. you could continue editing the input line.

If that's what you want, then that doesn't seem to be covered in the fzf docs and doesn't seem to be provided on Unix/Linux.

That's certainly possible to hook up, but since it hasn't been described by you or the fzf docs, I assume that isn't what you meant.

And if that's indeed not what you meant, then the question is unrelated to fzf.lua, and mostly unrelated to Clink, except for how to bind a key.

### Comment 7 by chrisant996

> [!NOTE]
> I edited the previous reply to change from `|| rem` to `|| call;` to clear the errorlevel.  Apparently in bash `:` has a side effect of clearing the exit code, but in CMD `rem` or `::` do not clear the exit code.

### Comment 8 by chrisant996

And the `+{2}` stuff seems to be argument syntax specific to `nvim` for jumping to a specified line number, so that stuff isn't applicable for other programs.

To make it work how you want with the specific programs you want will require various customizations; there isn't a generalized way to tell different programs to open a file and jump to a specific line number.

### Comment 9 by scillidan

Thank you very much for your detailed explanation and guidance!

I need to read through it carefully later to fully understand. But I first tried `rfs.cmd` and found it works perfectly.

Previously, I attempted many times to have GPT-4.0 write a `.bat` script but failed repeatedly. Later, I thought that `fzf.lua` could nicely integrate some fzf features, so I roughly assumed it could achieve this. But used `rfs.cmd` is more better.

I will try using WezTerm’s configuration to bind a key to run `rfs.cmd`. My problem has been perfectly solved.

Thank you for spending so much time! Wishing you a very pleasant day🌞!
