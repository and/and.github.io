---
layout: post
title: "See your Antigravity usage limits in the agy footer"
date: 2026-10-17 21:50:00 +0530
categories: [utilities]
permalink: /utilities/agy-usage-footer/
excerpt: "A status line for the Antigravity CLI (agy) that always shows how full your context is and how much of your 5-hour and weekly plan limits you have used, with bars that turn yellow, orange and red as you get close."
---

I kept typing `/usage` in the Antigravity CLI (`agy`) to check how close I was to my plan limits. Now the
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
/* The prompts are prose: wrap them instead of scrolling sideways */
.post-content div.language-text pre{white-space:pre-wrap;overflow-wrap:anywhere}
.ccf-copybar{display:flex;margin-top:.75rem;margin-bottom:.5rem}
.ccf-copy{font:inherit;font-size:.9rem;font-weight:600;padding:.4rem .9rem;border-radius:8px;border:1px solid currentColor;background:transparent;color:inherit;cursor:pointer}
.ccf-copy:hover{background:rgba(127,127,127,.12)}
.ccf-copy:focus-visible{outline:2px solid currentColor;outline-offset:2px}
</style>
<div class="ccf-term" role="img" aria-label="Footer: Claude Opus 5.5 (Medium), Context 10%, Session 74% resets 02:37, Weekly 37% resets Sat 21:37"><span class="ccf-seg">Claude Opus 5.5 (Medium)</span> <span class="ccf-seg">| Context: <span class="ccf-g">█</span><span class="ccf-t">█████████</span> <b>10%</b></span> <span class="ccf-seg">| Session: <span class="ccf-o">███████</span><span class="ccf-e ccf-o">▍</span><span class="ccf-t">██</span> <b>74%</b> (resets 02:37)</span> <span class="ccf-seg">| Weekly: <span class="ccf-g">███</span><span class="ccf-e ccf-g">▋</span><span class="ccf-t">██████</span> <b>37%</b> (resets Sat 21:37)</span></div>

- **Context** is how much of the current conversation's context window is used.
- **Session** is your rolling 5-hour plan limit for the active model's group, with the time it resets.
- **Weekly** is your 7-day plan limit for the active model's group, with the day and time it resets.

When you switch models with `/model`, Session and Weekly automatically switch to the matching quota group:
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

`agy` has a built-in `/statusline` command that runs a script of your choice to draw the footer. On every refresh it sends that script some JSON about the session, including `.model.display_name`, `.context_window.used_percentage`, and `.quota` buckets (each with a `remaining_fraction` from `0` to `1` and `reset_in_seconds`). The script reads those numbers with `jq`, converts `remaining_fraction` into percentage used (`(1 - remaining_fraction) * 100`), and prints one line.

## Before you start

- The script needs `jq`: `brew install jq` on macOS, `sudo apt install jq` on Debian and Ubuntu.

## Install

Pick one way. Both make the same changes, so you can install one way and uninstall the other.
Both back up any status line you already have, as `~/.config/agy/statusline.sh.bak` and
`~/.config/agy/statusline-setting.bak.json`. Installing again never overwrites that backup.

### Ask `agy`

Paste this prompt into `agy`. It shows you each change before making it, checks for `jq`, and
tests the footer at the end.

<details markdown="1">
<summary>Show the install prompt</summary>

~~~~text
Install a usage footer for my Antigravity CLI (agy) status line. Show me each change before you make it.

1. If ~/.config/agy/statusline.sh exists and ~/.config/agy/statusline.sh.bak does not, copy statusline.sh to statusline.sh.bak.
2. Write the script below to ~/.config/agy/statusline.sh, exactly as given, and make it executable (chmod +x).
3. Check that jq is installed. If it is not, tell me the command to install it on my system.
4. Test the script by piping this sample JSON into it, and show me the output:
{"model":{"display_name":"Claude Opus 5.5 (Medium)"},"context_window":{"used_percentage":10},"quota":{"3p-5h":{"remaining_fraction":0.26,"reset_in_seconds":16860},"3p-weekly":{"remaining_fraction":0.63,"reset_in_seconds":601200}}}
5. Tell me to run `/statusline ~/.config/agy/statusline.sh` inside agy to turn it on.

```bash
#!/bin/bash
# Antigravity CLI (agy) status line: model | Context | 5h Session limit | 7d Weekly limit, each with a bar
input=$(cat)

B=$'\033[1m'; N=$'\033[22m'

# 10-block bar: filled part on a solid dark gray track, so it reads as progress.
# Fill is light gray, then yellow above 50%, orange above 70%, red above 80%.
# Leftover under 10% becomes a partial block (eighths) drawn over the track.
T=$'\033[38;5;238m'; R=$'\033[39;49m'
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

# Epoch seconds to local time; date -r is macOS, date -d @ is Linux
fmt() { date -r "$1" "$2" 2>/dev/null || date -d "@$1" "$2" 2>/dev/null; }

model=$(echo "$input" | jq -r '.model.display_name // .model.id // "agy"')
ctx=$(printf '%.0f' "$(echo "$input" | jq -r '.context_window.used_percentage // 0')")

# Pick the 5h and 7d quota buckets for the active model's group
case "$model" in
  *Claude*|*claude*|*GPT*|*gpt*|*Opus*|*Sonnet*)
    five_key="3p-5h"; week_key="3p-weekly" ;;
  *Argon*|*argon*)
    five_key="gemini-5h"; week_key="gemini-argon-wtu-weekly" ;;
  *)
    five_key="gemini-5h"; week_key="gemini-weekly" ;;
esac

now=$(date +%s)
read -r five_frac five_secs <<< "$(echo "$input" | jq -r --arg k "$five_key" \
  '.quota[$k] | if . then "\(.remaining_fraction) \(.reset_in_seconds // empty)" else empty end')"
read -r week_frac week_secs <<< "$(echo "$input" | jq -r --arg k "$week_key" \
  '.quota[$k] | if . then "\(.remaining_fraction) \(.reset_in_seconds // empty)" else empty end')"

line="$model | Context: $(bar "$ctx")"
if [ -n "$five_frac" ]; then
  five=$(awk -v f="$five_frac" 'BEGIN { printf "%.0f", (1 - f) * 100 }')
  line="$line | Session: $(bar "$five")"
  [ -n "$five_secs" ] && line="$line (resets $(fmt "$((now + five_secs))" +%H:%M))"
fi
if [ -n "$week_frac" ]; then
  week=$(awk -v f="$week_frac" 'BEGIN { printf "%.0f", (1 - f) * 100 }')
  line="$line | Weekly: $(bar "$week")"
  [ -n "$week_secs" ] && line="$line (resets $(fmt "$((now + week_secs))" '+%a %H:%M'))"
fi

printf '%s' "$line"
```
~~~~

</details>

### Or paste a terminal command

This writes the script to `~/.config/agy/statusline.sh` and points `statusLine` in
`~/.gemini/antigravity-cli/settings.json` at it. Your other settings stay as they are.

<details markdown="1">
<summary>Show the install command</summary>

```bash
mkdir -p ~/.config/agy ~/.gemini/antigravity-cli
f=~/.gemini/antigravity-cli/settings.json; [ -f "$f" ] || echo '{}' > "$f"
# Back up your current status line once (a second install will not overwrite the backup)
[ -f ~/.config/agy/statusline.sh ] && [ ! -f ~/.config/agy/statusline.sh.bak ] && cp ~/.config/agy/statusline.sh ~/.config/agy/statusline.sh.bak
[ -f ~/.config/agy/statusline-setting.bak.json ] || jq '.statusLine // null' "$f" > ~/.config/agy/statusline-setting.bak.json
cat > ~/.config/agy/statusline.sh <<'EOF'
#!/bin/bash
# Antigravity CLI (agy) status line: model | Context | 5h Session limit | 7d Weekly limit, each with a bar
input=$(cat)

B=$'\033[1m'; N=$'\033[22m'

# 10-block bar: filled part on a solid dark gray track, so it reads as progress.
# Fill is light gray, then yellow above 50%, orange above 70%, red above 80%.
# Leftover under 10% becomes a partial block (eighths) drawn over the track.
T=$'\033[38;5;238m'; R=$'\033[39;49m'
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

# Epoch seconds to local time; date -r is macOS, date -d @ is Linux
fmt() { date -r "$1" "$2" 2>/dev/null || date -d "@$1" "$2" 2>/dev/null; }

model=$(echo "$input" | jq -r '.model.display_name // .model.id // "agy"')
ctx=$(printf '%.0f' "$(echo "$input" | jq -r '.context_window.used_percentage // 0')")

# Pick the 5h and 7d quota buckets for the active model's group
case "$model" in
  *Claude*|*claude*|*GPT*|*gpt*|*Opus*|*Sonnet*)
    five_key="3p-5h"; week_key="3p-weekly" ;;
  *Argon*|*argon*)
    five_key="gemini-5h"; week_key="gemini-argon-wtu-weekly" ;;
  *)
    five_key="gemini-5h"; week_key="gemini-weekly" ;;
esac

now=$(date +%s)
read -r five_frac five_secs <<< "$(echo "$input" | jq -r --arg k "$five_key" \
  '.quota[$k] | if . then "\(.remaining_fraction) \(.reset_in_seconds // empty)" else empty end')"
read -r week_frac week_secs <<< "$(echo "$input" | jq -r --arg k "$week_key" \
  '.quota[$k] | if . then "\(.remaining_fraction) \(.reset_in_seconds // empty)" else empty end')"

line="$model | Context: $(bar "$ctx")"
if [ -n "$five_frac" ]; then
  five=$(awk -v f="$five_frac" 'BEGIN { printf "%.0f", (1 - f) * 100 }')
  line="$line | Session: $(bar "$five")"
  [ -n "$five_secs" ] && line="$line (resets $(fmt "$((now + five_secs))" +%H:%M))"
fi
if [ -n "$week_frac" ]; then
  week=$(awk -v f="$week_frac" 'BEGIN { printf "%.0f", (1 - f) * 100 }')
  line="$line | Weekly: $(bar "$week")"
  [ -n "$week_secs" ] && line="$line (resets $(fmt "$((now + week_secs))" '+%a %H:%M'))"
fi

printf '%s' "$line"
EOF
chmod +x ~/.config/agy/statusline.sh
jq '.statusLine = {"type":"command","command":"~/.config/agy/statusline.sh"}' "$f" > "$f.tmp" && mv "$f.tmp" "$f"
```

</details>

If `agy` is already open, run `/statusline ~/.config/agy/statusline.sh` inside `agy` so the running session picks it up immediately.

## Uninstall

This puts back the status line you had before, or reverts to `agy`'s built-in default if you had none.

Inside `agy`, you can run:

~~~~text
/statusline delete
~~~~

Or run this in your terminal:

```bash
f=~/.gemini/antigravity-cli/settings.json; b=~/.config/agy/statusline-setting.bak.json
# Put back your old script, or remove this one if you had none
if [ -f ~/.config/agy/statusline.sh.bak ]; then mv ~/.config/agy/statusline.sh.bak ~/.config/agy/statusline.sh; else rm -f ~/.config/agy/statusline.sh; fi
# Put back your old statusLine setting, or remove it if you had none
if [ -f "$b" ]; then
  jq --slurpfile p "$b" 'if $p[0] == null then del(.statusLine) else .statusLine = $p[0] end' "$f" > "$f.tmp" && mv "$f.tmp" "$f" && rm "$b"
else
  jq 'del(.statusLine)' "$f" > "$f.tmp" && mv "$f.tmp" "$f"
fi
```

## Good to know

- **It works on macOS and Linux.** Reset times are calculated from `reset_in_seconds` using your computer's local time zone.
- **Separate from NotebookLM.** Even if your Google AI Pro subscription unlocks both Antigravity and NotebookLM, `agy` model limits and NotebookLM notebook limits are tracked in separate pools.

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
