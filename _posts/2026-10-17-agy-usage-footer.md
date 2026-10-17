---
layout: post
title: "See your Antigravity usage limits in the agy footer"
date: 2026-10-17 21:50:00 +0530
categories: [utilities]
permalink: /utilities/agy-usage-footer/
excerpt: "A status line for the Antigravity CLI (agy) that always shows how full your context is and how much of your 5-hour and weekly quota you have used for the active model group, with bars that turn yellow, orange and red as you get close."
---

I kept typing `/usage` in the Antigravity CLI (`agy`) to check how close I was to my 5-hour and weekly model limits. Now the footer shows it all the time:

<style>
.ccf-term{background:#1c2029;color:#c9cdd6;border-radius:10px;padding:.9rem 1rem;font:13px/1.5 "JetBrains Mono",ui-monospace,Menlo,Consolas,monospace;overflow-x:auto;margin-top:1.25rem;margin-bottom:1.25rem}
.ccf-term span.ccf-seg{white-space:nowrap}
.ccf-term b{color:#fff}
.ccf-d{color:#777e8b}
.ccf-t{color:#444}.ccf-e{background:#444}
.ccf-g{color:#bcbcbc}.ccf-y{color:#ffd700}.ccf-o{color:#ff8700}.ccf-r{color:#ff0000}
.ccf-key{display:flex;flex-wrap:wrap;gap:.4rem 1.25rem;list-style:none;padding:0;margin-top:0;margin-bottom:1.5rem;font-size:.9rem}
.ccf-key li{display:flex;align-items:center;gap:.45rem;margin:0}
.ccf-key i{width:.8rem;height:.8rem;border-radius:2px;display:inline-block}
/* The prompts are prose: wrap them instead of scrolling sideways */
.post-content div.language-text pre{white-space:pre-wrap;overflow-wrap:anywhere}
.ccf-copybar{display:flex;margin-top:.75rem;margin-bottom:.5rem}
.ccf-copy{font:inherit;font-size:.9rem;font-weight:600;padding:.4rem .9rem;border-radius:8px;border:1px solid currentColor;background:transparent;color:inherit;cursor:pointer}
.ccf-copy:hover{background:rgba(127,127,127,.12)}
.ccf-copy:focus-visible{outline:2px solid currentColor;outline-offset:2px}
</style>
<div class="ccf-term" role="img" aria-label="Footer: Claude Opus 5.5 (Medium), Ctx 10%, 5h 74% resets in 4h41m, Week 37% resets in 6d23h"><span class="ccf-seg">Claude Opus 5.5 (Medium)</span> <span class="ccf-seg"><span class="ccf-d">|</span> Ctx <span class="ccf-g">█</span><span class="ccf-t">█████████</span> <b>10%</b></span> <span class="ccf-seg"><span class="ccf-d">|</span> 5h <span class="ccf-o">███████</span><span class="ccf-e ccf-o">▍</span><span class="ccf-t">██</span> <b>74%</b> <span class="ccf-d">↻4h41m</span></span> <span class="ccf-seg"><span class="ccf-d">|</span> Week <span class="ccf-g">███</span><span class="ccf-e ccf-g">▋</span><span class="ccf-t">██████</span> <b>37%</b> <span class="ccf-d">↻6d23h</span></span></div>

- **Ctx** is how much of the current conversation's context window is used.
- **5h** is the rolling 5-hour quota used for your active model's group, with a countdown (`↻4h41m`) until it refreshes.
- **Week** is the 7-day quota used for your active model's group, with a countdown (`↻6d23h`) until it refreshes.

When you switch models with `/model`, the footer automatically switches to the matching quota pool:
- **Claude & GPT models** show `3p-5h` and `3p-weekly`.
- **Gemini 4 Argon** shows `gemini-5h` and `gemini-argon-wtu-weekly`.
- **Other Gemini models** show `gemini-5h` and `gemini-weekly`.

Each bar has 10 blocks of 10%. The filled part changes colour as you use more:

<ul class="ccf-key">
<li><i style="background:#bcbcbc"></i>Up to 50%</li>
<li><i style="background:#ffd700"></i>Over 50%</li>
<li><i style="background:#ff8700"></i>Over 70%</li>
<li><i style="background:#ff0000"></i>Over 80%</li>
</ul>

## How it works

`agy` has a built-in `/statusline` command that can run a shell script of your choice to render the footer. On every refresh it pipes a JSON blob to that script on stdin containing `.model.display_name`, `.context_window.used_percentage`, `.terminal_width`, and `.quota` buckets (each with `remaining_fraction` and `reset_in_seconds`).

Because `agy` reports `remaining_fraction` (from `0` to `1`), the script converts that to percentage used (`(1 - remaining_fraction) * 100`) so it reads the same way as the context bar: fuller means closer to the limit. If your terminal window is too narrow for the full bars, it falls back to compact percentages (`5h 74% | Week 37%`) so the line never wraps or clips.

## Before you start

- The script needs `jq`: `brew install jq` on macOS, `sudo apt install jq` on Debian and Ubuntu.

## Install

Pick one way. Both write the script to `~/.config/agy/statusline.sh` and back up any existing `~/.config/agy/statusline.sh` to `~/.config/agy/statusline.sh.bak` once so a second install never overwrites your backup.

### Ask `agy`

Paste this prompt into `agy`, then run `/statusline ~/.config/agy/statusline.sh` when it finishes:

<details markdown="1">
<summary>Show the install prompt</summary>

~~~~text
Install a usage footer for my Antigravity CLI (agy) status line. Show me each change before you make it.

1. If ~/.config/agy/statusline.sh exists and ~/.config/agy/statusline.sh.bak does not, copy statusline.sh to statusline.sh.bak.
2. Write the script below to ~/.config/agy/statusline.sh and make it executable (chmod +x).
3. Check that jq is installed. If it is not, tell me the command to install it on my system.
4. Test the script by piping this sample JSON into it, and show me the output:
{"model":{"display_name":"Claude Opus 5.5 (Medium)"},"context_window":{"used_percentage":10},"terminal_width":160,"quota":{"3p-5h":{"remaining_fraction":0.26,"reset_in_seconds":16860},"3p-weekly":{"remaining_fraction":0.63,"reset_in_seconds":601200}}}
5. Remind me to run `/statusline ~/.config/agy/statusline.sh` inside agy to activate it.

```bash
#!/bin/bash
# Antigravity CLI (agy) footer: model | context | quota for the active model's group.
# Setup inside agy:  /statusline ~/.config/agy/statusline.sh
# Requires: jq
input=$(cat)

B=$'\033[1m'; N=$'\033[22m'; D=$'\033[2m'
T=$'\033[38;5;238m'; R=$'\033[0m'

# 10-block bar (light gray -> yellow >50% -> orange >70% -> red >80%).
bar() {
  local p=$1 fill='' track='' edge='' i c
  [ "$p" -gt 100 ] && p=100
  [ "$p" -lt 0 ] && p=0
  if [ "$p" -gt 80 ]; then c=196; elif [ "$p" -gt 70 ]; then c=208; elif [ "$p" -gt 50 ]; then c=220; else c=250; fi
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

# Seconds -> compact duration: 45m, 4h43m, 6d23h
dur() {
  local s=${1:-0} d h m
  d=$((s / 86400)); h=$(((s % 86400) / 3600)); m=$(((s % 3600) / 60))
  if [ $d -gt 0 ]; then printf '%dd%dh' $d $h
  elif [ $h -gt 0 ]; then printf '%dh%02dm' $h $m
  else printf '%dm' $m; fi
}

model=$(echo "$input" | jq -r '.model.display_name // .model.id // "agy"')
ctx=$(printf '%.0f' "$(echo "$input" | jq -r '.context_window.used_percentage // 0')")
width=$(echo "$input" | jq -r '.terminal_width // empty')
[ -z "$width" ] && width=$(tput cols 2>/dev/null || echo 200)

# Pick quota buckets for the active model's group (as shown in /usage).
case "$model" in
  *Claude*|*claude*|*GPT*|*gpt*) keys="3p-5h:5h 3p-weekly:Week" ;;
  *Argon*|*argon*)               keys="gemini-5h:5h gemini-argon-wtu-weekly:Week" ;;
  *)                             keys="gemini-5h:5h gemini-weekly:Week" ;;
esac

line="$model ${D}|${R} Ctx $(bar "$ctx")"

for pair in $keys; do
  key=${pair%%:*}; label=${pair#*:}
  read -r frac secs <<< "$(echo "$input" | jq -r --arg k "$key" \
    '.quota[$k] | if . then "\(.remaining_fraction) \(.reset_in_seconds // 0)" else empty end')"
  [ -z "$frac" ] && continue
  used=$(awk -v f="$frac" 'BEGIN { printf "%.0f", (1 - f) * 100 }')
  line="$line ${D}|${R} $label $(bar "$used") ${D}↻$(dur "$secs")${R}"
done

# Trim to terminal width (ANSI-aware) so the footer never wraps or clips mid-glyph.
visible=$(printf '%s' "$line" | sed $'s/\033\\[[0-9;]*m//g')
if [ ${#visible} -gt "$width" ]; then
  line="$model ${D}|${R} Ctx $(bar "$ctx")"
  for pair in $keys; do
    key=${pair%%:*}; label=${pair#*:}
    frac=$(echo "$input" | jq -r --arg k "$key" '.quota[$k].remaining_fraction // empty')
    [ -z "$frac" ] && continue
    used=$(awk -v f="$frac" 'BEGIN { printf "%.0f", (1 - f) * 100 }')
    line="$line ${D}|${R} $label ${B}${used}%${N}"
  done
fi

printf '%s' "$line"
```
~~~~

</details>

### Or paste a terminal command

This writes the script to `~/.config/agy/statusline.sh` and updates `~/.gemini/antigravity-cli/settings.json` directly:

<details markdown="1">
<summary>Show the install command</summary>

```bash
mkdir -p ~/.config/agy ~/.gemini/antigravity-cli
f=~/.gemini/antigravity-cli/settings.json; [ -f "$f" ] || echo '{}' > "$f"
[ -f ~/.config/agy/statusline.sh ] && [ ! -f ~/.config/agy/statusline.sh.bak ] && cp ~/.config/agy/statusline.sh ~/.config/agy/statusline.sh.bak
[ -f ~/.config/agy/statusline-setting.bak.json ] || jq '.statusLine // null' "$f" > ~/.config/agy/statusline-setting.bak.json
cat > ~/.config/agy/statusline.sh <<'EOF'
#!/bin/bash
# Antigravity CLI (agy) footer: model | context | quota for the active model's group.
# Setup inside agy:  /statusline ~/.config/agy/statusline.sh
# Requires: jq
input=$(cat)

B=$'\033[1m'; N=$'\033[22m'; D=$'\033[2m'
T=$'\033[38;5;238m'; R=$'\033[0m'

# 10-block bar (light gray -> yellow >50% -> orange >70% -> red >80%).
bar() {
  local p=$1 fill='' track='' edge='' i c
  [ "$p" -gt 100 ] && p=100
  [ "$p" -lt 0 ] && p=0
  if [ "$p" -gt 80 ]; then c=196; elif [ "$p" -gt 70 ]; then c=208; elif [ "$p" -gt 50 ]; then c=220; else c=250; fi
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

# Seconds -> compact duration: 45m, 4h43m, 6d23h
dur() {
  local s=${1:-0} d h m
  d=$((s / 86400)); h=$(((s % 86400) / 3600)); m=$(((s % 3600) / 60))
  if [ $d -gt 0 ]; then printf '%dd%dh' $d $h
  elif [ $h -gt 0 ]; then printf '%dh%02dm' $h $m
  else printf '%dm' $m; fi
}

model=$(echo "$input" | jq -r '.model.display_name // .model.id // "agy"')
ctx=$(printf '%.0f' "$(echo "$input" | jq -r '.context_window.used_percentage // 0')")
width=$(echo "$input" | jq -r '.terminal_width // empty')
[ -z "$width" ] && width=$(tput cols 2>/dev/null || echo 200)

# Pick quota buckets for the active model's group (as shown in /usage).
case "$model" in
  *Claude*|*claude*|*GPT*|*gpt*) keys="3p-5h:5h 3p-weekly:Week" ;;
  *Argon*|*argon*)               keys="gemini-5h:5h gemini-argon-wtu-weekly:Week" ;;
  *)                             keys="gemini-5h:5h gemini-weekly:Week" ;;
esac

line="$model ${D}|${R} Ctx $(bar "$ctx")"

for pair in $keys; do
  key=${pair%%:*}; label=${pair#*:}
  read -r frac secs <<< "$(echo "$input" | jq -r --arg k "$key" \
    '.quota[$k] | if . then "\(.remaining_fraction) \(.reset_in_seconds // 0)" else empty end')"
  [ -z "$frac" ] && continue
  used=$(awk -v f="$frac" 'BEGIN { printf "%.0f", (1 - f) * 100 }')
  line="$line ${D}|${R} $label $(bar "$used") ${D}↻$(dur "$secs")${R}"
done

# Trim to terminal width (ANSI-aware) so the footer never wraps or clips mid-glyph.
visible=$(printf '%s' "$line" | sed $'s/\033\\[[0-9;]*m//g')
if [ ${#visible} -gt "$width" ]; then
  line="$model ${D}|${R} Ctx $(bar "$ctx")"
  for pair in $keys; do
    key=${pair%%:*}; label=${pair#*:}
    frac=$(echo "$input" | jq -r --arg k "$key" '.quota[$k].remaining_fraction // empty')
    [ -z "$frac" ] && continue
    used=$(awk -v f="$frac" 'BEGIN { printf "%.0f", (1 - f) * 100 }')
    line="$line ${D}|${R} $label ${B}${used}%${N}"
  done
fi

printf '%s' "$line"
EOF
chmod +x ~/.config/agy/statusline.sh
jq '.statusLine = {"type":"command","command":"~/.config/agy/statusline.sh"}' "$f" > "$f.tmp" && mv "$f.tmp" "$f"
```

</details>

If `agy` is already open, run `/statusline ~/.config/agy/statusline.sh` inside `agy` so the running session picks it up immediately.

## Uninstall

To revert to `agy`'s built-in default footer at any time, run inside `agy`:

```text
/statusline delete
```

Or run this in your terminal to restore any previous backup:

```bash
f=~/.gemini/antigravity-cli/settings.json; b=~/.config/agy/statusline-setting.bak.json
if [ -f ~/.config/agy/statusline.sh.bak ]; then mv ~/.config/agy/statusline.sh.bak ~/.config/agy/statusline.sh; else rm -f ~/.config/agy/statusline.sh; fi
if [ -f "$b" ]; then
  jq --slurpfile p "$b" 'if $p[0] == null then del(.statusLine) else .statusLine = $p[0] end' "$f" > "$f.tmp" && mv "$f.tmp" "$f" && rm "$b"
else
  jq 'del(.statusLine)' "$f" > "$f.tmp" && mv "$f.tmp" "$f"
fi
```

## Good to know

- **Time-zone independent.** Because the countdown uses `reset_in_seconds` directly instead of formatting the UTC `reset_time` string, it works identically on macOS and Linux in any time zone.
- **Separate from NotebookLM.** Even if your Google AI Pro subscription unlocks both Antigravity and NotebookLM Pro, `agy` token quotas (`gemini-5h`, `3p-5h`, etc.) and NotebookLM notebook limits are tracked in separate pools.

<script>
// A copy button above every prompt and command. Blocks inside a collapsed
// "Show ..." section get their button above that section, so you can copy
// without opening it.
document.querySelectorAll('.post-content div.language-text, .post-content div.language-bash').forEach(block => {
  const code = block.querySelector('code');
  const label = block.classList.contains('language-text') ? 'Copy prompt' : 'Copy command';
  const bar = document.createElement('div');
  bar.className = 'ccf-copybar';
  const btn = document.createElement('button');
  btn.type = 'button';
  btn.className = 'ccf-copy';
  btn.textContent = label;
  bar.appendChild(btn);
  const anchor = block.closest('details') || block;
  anchor.parentNode.insertBefore(bar, anchor);
  btn.addEventListener('click', () => {
    const reset = () => setTimeout(() => { btn.textContent = label; }, 1800);
    const fallback = () => {
      const d = block.closest('details'); if (d) d.open = true;
      const r = document.createRange(); r.selectNodeContents(code);
      const sel = getSelection(); sel.removeAllRanges(); sel.addRange(r);
      btn.textContent = 'Selected: press Ctrl+C or ⌘C'; reset();
    };
    try {
      navigator.clipboard.writeText(code.textContent).then(() => { btn.textContent = 'Copied'; reset(); }, fallback);
    } catch (e) { fallback(); }
  });
});
</script>
