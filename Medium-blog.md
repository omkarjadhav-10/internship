# How I Set Up My Developer Terminal on Windows 11 

---

## The Moment I Realized Something Was Off

It was my first week as a software engineering intern. I'd just gotten my Windows 11 machine set up, cloned the company repo, and opened PowerShell for the first time. The prompt just said `PS C:\Users\Omkar>`. That was it. No color. No branch name. No hints about what I was doing or where I was.

I typed `git status` and got a wall of white text. I ran `ls` and got a basic list with no icons, no sizes that made sense, nothing useful. I looked over at a senior dev's screen during a meeting, his terminal looked like a different application entirely. Colors everywhere, git branch in the prompt, icons next to file names, autocomplete suggestions appearing as he typed.

I asked him what he was using. He said, "Same PowerShell, just configured properly." That sentence stuck with me. Same tool. Completely different experience. That's when I decided to actually set up my environment properly.

---

## Who Is This For?

If you're a developer on Windows 11 who's still using the default PowerShell prompt with no customization, this is for you. Especially if you're an intern or a new developer setting up your first real dev machine. It's also useful if you've been meaning to improve your terminal setup but didn't know where to start or what tools to even look for.

You don't need to be experienced. You just need about 45 minutes and a willingness to tweak things.

---

## What's Wrong With the Default Setup?

Nothing is technically broken with the default Windows terminal. But here's what you're missing:

- PowerShell 5 (the default) is old. PowerShell 7 exists and it's faster, more compatible, and actively maintained.
- There's no git branch in the prompt, so you're constantly typing `git branch` to figure out where you are.
- There's no autosuggestions, meaning you re-type the same long commands every single day.
- No syntax highlighting, so you can't tell if you've made a typo until you actually run the command.
- Navigation is painful. The built-in `ls` and `cd` commands are basic.
- No icons on files. No shortcuts for Docker, npm, or git.

None of these are dealbreakers individually. Together, they add up to a lot of friction and friction kills focus.

---

## What a Professional Terminal Actually Looks Like

Before jumping to the steps, it's worth understanding what we're building toward. A well-configured terminal has a few distinct layers.

First, you have a modern shell, PowerShell 7 instead of the ancient version that ships with Windows. Then you have a smart themed prompt (Oh My Posh) that shows you git branch, folder path, Node version, and time at a glance. The font matters too; icons in the terminal require a Nerd Font, otherwise you just get broken squares everywhere. Then there are modern CLI replacements: tools like `eza` instead of `ls`, `bat` instead of `cat`, `zoxide` instead of constantly typing full paths. And finally, autosuggestions and syntax highlighting baked into your shell profile.

It sounds like a lot. But each piece is simple to add, and they all work together.

---

## Let's Build It

This is the actual step-by-step walkthrough from my setup. I've included every command in the right order, plus a couple of moments where things went sideways and how I fixed them.

---

### Step 1 - Install PowerShell 7

Windows 11 comes with PowerShell 5 baked in. It works, but it's old — think of it as the terminal equivalent of running IE11. PowerShell 7 (you'll see it called `pwsh`) is the modern, actively maintained version. It's faster, more compatible, and required for Oh My Posh to work properly.

Open Windows Terminal as Administrator for this step only. Right-click the Windows Terminal icon and choose "Run as administrator". Then run:

```powershell
winget install Microsoft.PowerShell
```

It takes about a minute. Once it's done, verify it installed correctly:

```powershell
pwsh --version
```

You should see something like `PowerShell 7.5.x`. If you see that, you're good.

Now set it as your default. Open Windows Terminal Settings with `Ctrl + ,`, click "Startup" in the sidebar, and change the Default profile to PowerShell 7. Hit Save.

> **Tip:** From this point forward, always use PowerShell 7, not the old "Windows PowerShell" entry in the dropdown. They look similar but they're different shells.

---

### Step 2 — Install Oh My Posh

Oh My Posh is what transforms your boring prompt into something actually useful. It shows your current folder, git branch, Node.js version, last command timing — all right there in the prompt, color-coded and readable.

For this step, close any Administrator windows. Open a fresh Windows Terminal without right-clicking, just a regular user session. Then run:

```powershell
winget install JanDeDobbeleer.OhMyPosh -s winget
```

Once it's installed, you need to download the themes separately. The winget version doesn't bundle them. Run these three commands one at a time:

```powershell
# Create the themes folder
New-Item -ItemType Directory -Force -Path "$env:LOCALAPPDATA\Programs\oh-my-posh\themes"

# Download the themes archive from GitHub
Invoke-WebRequest -Uri "https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/themes.zip" -OutFile "$env:TEMP\posh-themes.zip"

# Extract themes into the folder
Expand-Archive -Path "$env:TEMP\posh-themes.zip" -DestinationPath "$env:LOCALAPPDATA\Programs\oh-my-posh\themes" -Force
```

Verify the themes downloaded correctly:

```powershell
$env:POSH_THEMES_PATH = "$env:LOCALAPPDATA\Programs\oh-my-posh\themes"
Get-ChildItem $env:POSH_THEMES_PATH | Measure-Object
```

You should see a `Count` of 150 or higher. If it shows 0, something went wrong with the download run the three commands again.

---

### Step 3 — Install MesloLGS Nerd Font

Here's where I hit my first wall. I'd installed Oh My Posh and was feeling good about it. Then I reloaded my terminal and the prompt looked like... complete nonsense. Squares everywhere. Little question marks. Arrows that didn't render. It looked worse than the default.

Turns out, Oh My Posh uses special icons and symbols that require a "Nerd Font" to display correctly. Your terminal's default font doesn't have those glyphs. Without the right font, you get garbage characters instead.

The fix is simple. Run this in PowerShell 7 (as a regular user, not Admin):

```powershell
oh-my-posh font install meslo
```

Wait for it to finish. You'll see a list of variants getting installed, including MesloLGS Nerd Font Mono.

Now go set it in Windows Terminal. Press `Ctrl + ,` to open Settings, find "PowerShell 7" under Profiles in the sidebar, click "Appearance", and set the Font face to exactly:

```
MesloLGS Nerd Font Mono
```

Set the font size to 13. Save. Then fully close and reopen Windows Terminal. When it opens again, the icons should render properly.

> **Watch out:** If Windows Terminal refuses to open after changing the font, the font name was entered incorrectly. Navigate to:
> `C:\Users\YourUsername\AppData\Local\Packages\Microsoft.WindowsTerminal_8wekyb3d8bbwe\LocalState\`
> and open `settings.json` in Notepad. Find the font face line and temporarily change it back to `"Consolas"`, save, reopen Terminal, then try again.

---

### Step 4 — Configure Windows Terminal Appearance

Press `Ctrl + ,`, scroll to the very bottom of the Settings sidebar, and click "Open JSON file".

Replace everything with:

```json
{
  "$schema": "https://aka.ms/terminal-profiles-schema",
  "copyFormatting": "none",
  "copyOnSelect": false,
  "defaultProfile": "{caa5347a-4147-4032-9929-3f9c0e20fb6e}",
  "keybindings": [
    { "id": "Terminal.CopyToClipboard", "keys": "ctrl+c" },
    { "id": "Terminal.PasteFromClipboard", "keys": "ctrl+v" },
    { "id": "Terminal.DuplicatePaneAuto", "keys": "alt+shift+d" }
  ],
  "profiles": {
    "defaults": {
      "font": { "face": "MesloLGS Nerd Font Mono", "size": 13 },
      "opacity": 85,
      "useAcrylic": true,
      "padding": "12"
    },
    "list": [
      {
        "commandline": "C:\\Program Files\\PowerShell\\7\\pwsh.exe",
        "guid": "{caa5347a-4147-4032-9929-3f9c0e20fb6e}",
        "hidden": false,
        "name": "PowerShell 7",
        "startingDirectory": "%USERPROFILE%",
        "colorScheme": "One Half Dark"
      }
    ]
  },
  "schemes": [],
  "themes": []
}
```

---

### Step 5 — Create Your PowerShell Profile

```powershell
Test-Path $PROFILE
New-Item -Path $PROFILE -Type File -Force
notepad $PROFILE
```

Paste:

```powershell
$env:POSH_THEMES_PATH = "$env:LOCALAPPDATA\Programs\oh-my-posh\themes"
oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH\tokyonight_storm.omp.json" | Invoke-Expression

Set-PSReadLineOption -PredictionSource History
Set-PSReadLineOption -PredictionViewStyle InlineView
Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward

Invoke-Expression (& { (zoxide init powershell | Out-String) })

function ls  { eza --icons --group-directories-first $args }
function ll  { eza -la --icons --git --group-directories-first $args }
function cat { bat $args }

function gs { git status }
function ga { git add . }
function gp { git push }
function gl { lazygit }

function dcu { docker-compose up }
function dcd { docker-compose down }

function nd { npm run dev }
function ni { npm install $args }

function .. { cd .. }
```

---

### Step 6 — Install Scoop

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
scoop --version
```

---

### Step 7 — Install CLI Tools

```powershell
scoop bucket add extras
scoop install eza bat fd ripgrep zoxide fzf lazygit
```

---

### Step 8 — Reload Profile

```powershell
. $PROFILE
```

**Quick Checks:**

```powershell
oh-my-posh --version

ll
cat $PROFILE
gl
z downloads
```

You're done.

---

## How I Actually Use These Day-to-Day

The aliases I added to my profile make these tools part of my muscle memory now:

```powershell
# File listing
function ll { eza -la --icons --git }
function lt { eza --tree --icons }

# Git shortcuts
function gs { git status }
function gl { lazygit }
function gp { git push }

# Docker shortcuts
function dcu { docker compose up -d }
function dcd { docker compose down }

# NPM shortcuts
function nd { npm run dev }
function nb { npm run build }
```

Now instead of typing `git status` fifty times a day, it's just `gs`. Instead of `docker compose up -d`, it's `dcu`. These feel like small wins, but they compound over weeks.

---

## Before vs. After

**Before** setting this up, my terminal had:

- A plain white prompt with no context
- Manual navigation by typing full paths
- No idea what git branch I was on without checking
- A plain `dir` command for file listing
- Constant retyping of commands I'd run dozens of times

**After:**

- Themed prompt that shows branch, folder, and Node version at all times
- Autosuggestions as I type based on my history
- Icon-based file listing with git status per file
- Smart navigation with zoxide
- Production shortcuts for git, Docker, and npm

The difference in feel is significant. It's not just prettier, it's faster and less mentally taxing.

---

## What I Learned From This

Setting this up took me about 45 minutes total. But since then, I've saved that time back many times over just from faster navigation and fewer repeated commands.

There's something else though. A well-configured environment changes how you feel about working. When your tools are clean and responsive, you feel more like a professional using them. There's less friction between what you want to do and actually doing it.

Developers, especially early-career developers tend to underestimate the terminal. It's easy to think of it as just a box you type commands into. But you're in there for hours every day. How it looks and feels actually matters.

---

> **Your terminal is where you spend hours every day.**
> Treat it like a professional workspace not just a default window.
