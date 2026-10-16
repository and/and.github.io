---
layout: post
title: "See your Claude usage limits in the Claude Code footer"
date: 2026-10-16 09:30:00 +0530
categories: [utilities]
permalink: /utilities/claude-code-usage-footer/
excerpt: "A status line for Claude Code that always shows how full your context is and how much of your 5-hour and weekly plan limits you have used, with bars that turn yellow, orange and red as you get close."
---

I kept typing `/usage` in Claude Code to check how close I was to my plan limits. Now the
footer shows it all the time:

<style>
.ccf-term{background:#1c2029;color:#c9cdd6;border-radius:10px;padding:.9rem 1rem;font:13px/1.5 "JetBrains Mono",ui-monospace,Menlo,Consolas,monospace;overflow-x:auto;margin-top:1.25rem;margin-bottom:1.25rem}
.ccf-term span.ccf-seg{white-space:nowrap}
.ccf-term b{color:#fff}
.ccf-t{color:#444}.ccf-e{background:#444}
.ccf-g{color:#bcbcbc}.ccf-y{color:#ffd700}.ccf-o{color:#ff8700}.ccf-r{color:#ff0000}
.ccf-key{display:flex;flex-wrap:wrap;gap:.4rem 1.25rem;list-style:none;padding:0;margin-top:0;margin-bottom:1.5rem;font-size:.9rem}
.ccf-key li{display:flex;align-items:center;gap:.45rem;margin:0}
.ccf-key i{width:.8rem;height:.8rem;border-radius:2px;display:inline-block}
</style>
<div class="ccf-term" role="img" aria-label="Footer: Opus 5.5, Context 7%, Session 37% resets 11:30, Weekly 72% resets Sat 17:30"><span class="ccf-seg">Opus 5.5</span> <span class="ccf-seg">| Context: <span class="ccf-e ccf-g">▋</span><span class="ccf-t">█████████</span> <b>7%</b></span> <span class="ccf-seg">| Session: <span class="ccf-g">███</span><span class="ccf-e ccf-g">▋</span><span class="ccf-t">██████</span> <b>37%</b> (resets 11:30)</span> <span class="ccf-seg">| Weekly: <span class="ccf-y">███████</span><span class="ccf-e ccf-y">▏</span><span class="ccf-t">██</span> <b>72%</b> (resets Sat 17:30)</span></div>

- **Context** is how much of the current conversation's context window is used.
- **Session** is your rolling 5-hour plan limit. It counts every Claude Code window, claude.ai
  and your other devices, not just this conversation.
- **Weekly** is your 7-day plan limit, with the day and time it resets.

Each bar has 10 blocks of 10%. The filled part changes colour as you use more:

<ul class="ccf-key">
<li><i style="background:#bcbcbc"></i>Up to 50%</li>
<li><i style="background:#ffd700"></i>Over 50%</li>
<li><i style="background:#ff8700"></i>Over 75%</li>
<li><i style="background:#ff0000"></i>Over 85%</li>
</ul>

## How it works

Claude Code can run a script of your choice to draw a
[status line](https://code.claude.com/docs/en/statusline) above its footer. After every reply
it sends that script some JSON about the session. For Pro and Max subscribers, that JSON
includes `rate_limits.five_hour` and `rate_limits.seven_day`, each with a `used_percentage`
and a `resets_at` time. The script reads those numbers with `jq` and prints one line.

## Before you start

- You need a claude.ai **Pro or Max** plan, with Claude Code signed in to that account. With
  an API key, Session and Weekly don't appear.
- The script needs `jq`: `brew install jq` on macOS, `sudo apt install jq` on Debian and
  Ubuntu.

## Install

Pick one way. Both make the same changes, so you can install one way and uninstall the other.
Both back up any status line you already have, as `~/.claude/statusline.sh.bak` and
`~/.claude/statusline-setting.bak.json`. Installing again never overwrites that backup.

### Ask Claude Code

Paste this prompt into Claude Code. It shows you each change before making it, checks for
`jq`, and tests the footer at the end.

<details markdown="1">
<summary>Show the install prompt</summary>

~~~~text
Install a usage footer for my Claude Code status line. Show me each change before you make it.

1. If ~/.claude/statusline.sh exists and ~/.claude/statusline.sh.bak does not, copy statusline.sh to statusline.sh.bak.
2. If ~/.claude/statusline-setting.bak.json does not exist, save the current "statusLine" value from ~/.claude/settings.json into it as JSON. Write null if there is no statusLine or no settings.json.
3. Write the script below to ~/.claude/statusline.sh, exactly as given.
4. In ~/.claude/settings.json, set "statusLine" to {"type": "command", "command": "bash ~/.claude/statusline.sh"}. Create the file if it is missing. Keep every other setting.
5. Check that jq is installed. If it is not, tell me the command to install it on my system.
6. Test the script by piping this sample JSON into it, and show me the output:
{"model":{"display_name":"Opus 5.5"},"context_window":{"used_percentage":7},"rate_limits":{"five_hour":{"used_percentage":37},"seven_day":{"used_percentage":72}}}

```bash
#!/bin/bash
# Claude Code status line: model | context | 5h session limit | 7d weekly limit, each with a bar
input=$(cat)

B=$'\033[1m'; N=$'\033[22m'

# 10-block bar: filled part on a solid dark gray track, so it reads as progress.
# Fill is light gray, then yellow above 50%, orange above 75%, red above 85%.
# Leftover under 10% becomes a partial block (eighths) drawn over the track.
T=$'\033[38;5;238m'; R=$'\033[39;49m'
bar() {
  local p=$1 fill='' track='' edge='' i c
  [ "$p" -gt 100 ] && p=100
  if [ "$p" -gt 85 ]; then c=196; elif [ "$p" -gt 75 ]; then c=208; elif [ "$p" -gt 50 ]; then c=220; else c=250; fi
  local F=$'\033[38;5;'"${c}m" P=$'\033[38;5;'"${c};48;5;238m"
  local filled=$((p / 10)) part=$(( (p % 10) * 8 / 10 ))
  [ $part -eq 0 ] && [ $((p % 10)) -gt 0 ] && part=1
  local partials=('' '▏' '▎' '▍' '▌' '▋' '▊' '▉')
  for ((i = 0; i < filled; i++)); do fill="${fill}█"; done
  local rest=$((10 - filled))
  if [ $part -gt 0 ]; then edge="${P}${partials[$part]}${R}"; rest=$((rest - 1)); fi
  for ((i = 0; i < rest; i++)); do track="${track}█"; done
  printf '%s%s%s%s%s%s%s %s%s%%%s' "$F" "$fill" "$R" "$edge" "$T" "$track" "$R" "$B" "$p" "$N"
}

# Epoch seconds to local time; date -r is macOS, date -d @ is Linux
fmt() { date -r "$1" "$2" 2>/dev/null || date -d "@$1" "$2" 2>/dev/null; }

model=$(echo "$input" | jq -r '.model.display_name')
ctx=$(printf '%.0f' "$(echo "$input" | jq -r '.context_window.used_percentage // 0')")

# Rate limits (Claude.ai subscribers only; absent until first API response)
five=$(echo "$input" | jq -r '.rate_limits.five_hour.used_percentage // empty')
week=$(echo "$input" | jq -r '.rate_limits.seven_day.used_percentage // empty')
five_reset=$(echo "$input" | jq -r '.rate_limits.five_hour.resets_at // empty')
week_reset=$(echo "$input" | jq -r '.rate_limits.seven_day.resets_at // empty')

line="$model | Context: $(bar "$ctx")"
if [ -n "$five" ]; then
  line="$line | Session: $(bar "$(printf '%.0f' "$five")")"
  [ -n "$five_reset" ] && line="$line (resets $(fmt "$five_reset" +%H:%M))"
fi
if [ -n "$week" ]; then
  line="$line | Weekly: $(bar "$(printf '%.0f' "$week")")"
  [ -n "$week_reset" ] && line="$line (resets $(fmt "$week_reset" '+%a %H:%M'))"
fi

printf '%s' "$line"
```
~~~~

</details>

### Or paste a terminal command

This writes the script to `~/.claude/statusline.sh` and points `statusLine` in
`~/.claude/settings.json` at it. Your other settings stay as they are.

<details markdown="1">
<summary>Show the install command</summary>

```bash
mkdir -p ~/.claude
f=~/.claude/settings.json; [ -f "$f" ] || echo '{}' > "$f"
# Back up your current status line once (a second install will not overwrite the backup)
[ -f ~/.claude/statusline.sh ] && [ ! -f ~/.claude/statusline.sh.bak ] && cp ~/.claude/statusline.sh ~/.claude/statusline.sh.bak
[ -f ~/.claude/statusline-setting.bak.json ] || jq '.statusLine // null' "$f" > ~/.claude/statusline-setting.bak.json
cat > ~/.claude/statusline.sh <<'EOF'
#!/bin/bash
# Claude Code status line: model | context | 5h session limit | 7d weekly limit, each with a bar
input=$(cat)

B=$'\033[1m'; N=$'\033[22m'

# 10-block bar: filled part on a solid dark gray track, so it reads as progress.
# Fill is light gray, then yellow above 50%, orange above 75%, red above 85%.
# Leftover under 10% becomes a partial block (eighths) drawn over the track.
T=$'\033[38;5;238m'; R=$'\033[39;49m'
bar() {
  local p=$1 fill='' track='' edge='' i c
  [ "$p" -gt 100 ] && p=100
  if [ "$p" -gt 85 ]; then c=196; elif [ "$p" -gt 75 ]; then c=208; elif [ "$p" -gt 50 ]; then c=220; else c=250; fi
  local F=$'\033[38;5;'"${c}m" P=$'\033[38;5;'"${c};48;5;238m"
  local filled=$((p / 10)) part=$(( (p % 10) * 8 / 10 ))
  [ $part -eq 0 ] && [ $((p % 10)) -gt 0 ] && part=1
  local partials=('' '▏' '▎' '▍' '▌' '▋' '▊' '▉')
  for ((i = 0; i < filled; i++)); do fill="${fill}█"; done
  local rest=$((10 - filled))
  if [ $part -gt 0 ]; then edge="${P}${partials[$part]}${R}"; rest=$((rest - 1)); fi
  for ((i = 0; i < rest; i++)); do track="${track}█"; done
  printf '%s%s%s%s%s%s%s %s%s%%%s' "$F" "$fill" "$R" "$edge" "$T" "$track" "$R" "$B" "$p" "$N"
}

# Epoch seconds to local time; date -r is macOS, date -d @ is Linux
fmt() { date -r "$1" "$2" 2>/dev/null || date -d "@$1" "$2" 2>/dev/null; }

model=$(echo "$input" | jq -r '.model.display_name')
ctx=$(printf '%.0f' "$(echo "$input" | jq -r '.context_window.used_percentage // 0')")

# Rate limits (Claude.ai subscribers only; absent until first API response)
five=$(echo "$input" | jq -r '.rate_limits.five_hour.used_percentage // empty')
week=$(echo "$input" | jq -r '.rate_limits.seven_day.used_percentage // empty')
five_reset=$(echo "$input" | jq -r '.rate_limits.five_hour.resets_at // empty')
week_reset=$(echo "$input" | jq -r '.rate_limits.seven_day.resets_at // empty')

line="$model | Context: $(bar "$ctx")"
if [ -n "$five" ]; then
  line="$line | Session: $(bar "$(printf '%.0f' "$five")")"
  [ -n "$five_reset" ] && line="$line (resets $(fmt "$five_reset" +%H:%M))"
fi
if [ -n "$week" ]; then
  line="$line | Weekly: $(bar "$(printf '%.0f' "$week")")"
  [ -n "$week_reset" ] && line="$line (resets $(fmt "$week_reset" '+%a %H:%M'))"
fi

printf '%s' "$line"
EOF
jq '.statusLine = {"type":"command","command":"bash ~/.claude/statusline.sh"}' "$f" > "$f.tmp" && mv "$f.tmp" "$f"
```

</details>

Then send any message in Claude Code. The footer fills in after the first reply, because that
reply carries the usage numbers.

## Uninstall

This puts back the status line you had before, or removes it if you had none.

Paste this into Claude Code:

~~~~text
Remove the usage footer from my Claude Code status line and put back what I had before. Show me each change before you make it.

1. If ~/.claude/statusline.sh.bak exists, move it back to ~/.claude/statusline.sh. If it does not exist, delete ~/.claude/statusline.sh.
2. If ~/.claude/statusline-setting.bak.json exists, read its value. If the value is null, remove "statusLine" from ~/.claude/settings.json. Otherwise set "statusLine" to that value. Then delete statusline-setting.bak.json.
3. If ~/.claude/statusline-setting.bak.json does not exist, remove "statusLine" from ~/.claude/settings.json.
4. Keep every other setting in settings.json.
~~~~

Or run this in your terminal:

```bash
f=~/.claude/settings.json; b=~/.claude/statusline-setting.bak.json
# Put back your old script, or remove this one if you had none
if [ -f ~/.claude/statusline.sh.bak ]; then mv ~/.claude/statusline.sh.bak ~/.claude/statusline.sh; else rm -f ~/.claude/statusline.sh; fi
# Put back your old statusLine setting, or remove it if you had none
if [ -f "$b" ]; then
  jq --slurpfile p "$b" 'if $p[0] == null then del(.statusLine) else .statusLine = $p[0] end' "$f" > "$f.tmp" && mv "$f.tmp" "$f" && rm "$b"
else
  jq 'del(.statusLine)' "$f" > "$f.tmp" && mv "$f.tmp" "$f"
fi
```

## Good to know

- **The footer can trail `/usage` by a point or two.** It only gets new numbers when a reply
  arrives in that window, while your other windows keep using the same limit.
- **It works on macOS and Linux.** Reset times use your computer's time zone.
