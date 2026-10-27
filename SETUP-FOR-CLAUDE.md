# Instructions for Claude: join the team setup (part 1 of 2)

You are Claude Code on the machine of a new teammate (developer, tester / QA, or artist /
animator). They asked you to connect them to the team's Claude Code plugins. This public file
gets them signed in to GitHub and gives them access to the team's private repositories. Part 2
lives in the private repository `ai-foundation-team/claude-foundation`, file
`docs/guided-setup.md`; you open it in step 6.

Read this file verbatim (`curl.exe -s` on Windows, `curl -s` on macOS: `https://raw.githubusercontent.com/ai-foundation-team/join/main/SETUP-FOR-CLAUDE.md`), not through a summarising web fetch: the
commands must stay exact.

## Rules for the whole walkthrough

- Talk in the language the person writes in. Keep each message short: what you are doing, what
  they must do, how you will check it.
- **Never ask for, look at or store a password, token or one-time code.** Sign-ins happen in a
  window the person types into, never in this chat. If they paste a secret into the chat anyway,
  tell them to revoke it and create a new one.
- Run non-interactive commands yourself. A command that waits for typed input runs in a
  **separate PowerShell window that the person opens** (Start menu → type `PowerShell` → Enter;
  on macOS a Terminal window: Spotlight → type `Terminal` → Enter);
  give them the exact line to paste, then check the result yourself.
- Ask before installing software and before each step that changes their settings.
- Verify every step (the check is given with it) before moving on.
- Never commit or push anything. Only the maintainer of `ai-foundation-team` or a team lead on its approver list approves access.
- **One line is all the person needs to know**, the one that brought you here. You give them,
  word for word, everything else they type or click. To resume later they paste the same line.
- **Every run, first or resumed, starts with checks, not questions:** run the checks of steps 2
  to 5 and read the step 1 answers back from the access request
  (`gh issue list -R ai-foundation-team/join --author @me --state all --json title,body`). Ask
  only for what you could not find, and continue at the first failing check. Once the membership
  is `active` and `$HOME/claude-foundation` exists, go straight to step 6.

## Step 1. Who they are

Ask, in one message:

1. Role(s): developer, tester (QA), artist / animator. Several are fine, all three too.
2. Do they work on the client games?

Keep the answers; part 2 and the access request use them. **Resuming in a new session** (after
a restart or after waiting for the invite): read them back from the request with
`gh issue list -R ai-foundation-team/join --author @me --state all --json title,body`, or ask
again.

## Step 2. Software

**On Windows:**

Check each with its version command; offer to install only what is missing:

| Need | Check | Install (after their yes) |
| --- | --- | --- |
| Git | `git --version` | `winget install --id Git.Git -e` |
| GitHub CLI | `gh --version` | `winget install --id GitHub.cli -e` |
| Node.js 20.19+ | `node --version` | `winget install --id OpenJS.NodeJS.LTS -e` |
| Python 3.10+ (developers, QA) | `python --version` (anything but a version number, e.g. the Microsoft Store prompt, counts as missing) | `winget install --id Python.Python.3.12 -e` |
| Claude Code 2.1.289+ | `claude --version` | `claude update` |
| Chrome or Edge (artists) | ask | — |

New installs do not reach this session's `PATH`. After installing, ask the person to restart
Claude Code (close it, open a new PowerShell, run `claude`) and paste the same line as at the
start; the new session continues at the first failing check.

**On macOS** (not tested on a real Mac yet: tell the person, and report what fails to the maintainer):

| Need | Check | Install (after their yes) |
| --- | --- | --- |
| Homebrew | `brew --version` (or `/opt/homebrew/bin/brew --version`: a fresh one is not on `PATH`) | the one line from https://brew.sh, in Terminal, then the `brew shellenv` lines it prints under "Next steps" |
| Git, GitHub CLI, Node.js 20.19+ | `git --version`, `gh --version`, `node --version` | `brew install git gh node` |
| Python 3.10+ (developers, QA) | `xcode-select -p` first (without it `python3` opens an install dialog), then `python3 --version` | `brew install python` |
| Claude Code 2.1.289+ | `claude --version` | `claude update` |
| Chrome (artists) | ask | — |

After installing, the person restarts Claude Code from a new Terminal window. In part 2 the setup on a Mac
runs `bootstrap.sh` (the twin of the Windows `bootstrap.ps1`; it can also install missing Homebrew packages
when the person runs it in Terminal). On a Mac, tracker tokens and IDT Asset Pipeline do not work yet.

## Step 3. GitHub account and sign-in

No GitHub account yet? They create one at https://github.com/signup.

Check: `gh auth status`. If it is not logged in to github.com, ask the person to open a new
PowerShell window and run:

```
gh auth login --hostname github.com --git-protocol https --web
```

They copy the one-time code shown there into the browser page and approve. If it asks
"Authenticate Git with your GitHub credentials?", the answer is Yes.

Check again: `gh auth status` shows "Logged in to github.com". Read their username:
`gh api user --jq .login`.

## Step 4. Ask for access

Check membership first:

```
gh api user/memberships/orgs/ai-foundation-team --jq .state
```

- `active`: access is already there; go to step 6.
- A request is already open (`gh issue list -R ai-foundation-team/join --author @me`): do not
  file another; go to step 5 and wait for the invite.
- `pending`: an invite is waiting; go to step 5.
- An error (not found): no access yet. File the request below.

Show the person the request, and after their yes create it (GitHub notifies the maintainer and the approvers):

```
gh issue create -R ai-foundation-team/join --title "Access request: <login>" --body "GitHub username: <login>
Role(s): <roles>
Client games: <yes/no>"
```

Never put an e-mail address, phone number or anything private into the issue: it is public.
If `gh issue create` fails, write the same text for the person to send to the maintainer in chat.

Tell them what happens next: the maintainer or a team lead who approves requests labels the issue,
and an invite to the organisation `ai-foundation-team` follows automatically (GitHub e-mails it).
Membership gives read access to the team plugins and IDT Particle Studio. Client game data is
granted separately, by the maintainer only.

Then offer to wait here. Start one background command (`run_in_background`) that exits when the
invite exists, checking every two minutes for up to eight hours; its exit wakes you:

```
for i in $(seq 240); do s=$(gh api user/memberships/orgs/ai-foundation-team --jq .state 2>/dev/null) && { echo "$s"; exit 0; }; sleep 120; done; echo timeout
```

Tell them the moment it prints `pending` or `active`. If they would rather close
Claude Code, tell them to paste the same line as at the start into a new session later: you will
pick up where this left off.

## Step 5. Accept the invite

When the person says an invite came, or when you resume, check the state again (step 4
command).

- `pending`: tell them an invite to `ai-foundation-team` is waiting and, after their yes, accept
  it yourself: `gh api -X PATCH user/memberships/orgs/ai-foundation-team -f state=active`.
  If that call fails (a default sign-in may lack the scope), they press **Join** on
  https://github.com/orgs/ai-foundation-team/invitation. Check again.
- `active`: continue.

Check that the plugin repository is visible:
`gh repo view ai-foundation-team/claude-foundation --json name --jq .name` prints
`claude-foundation`.

## Step 6. Continue with part 2

```
gh repo clone ai-foundation-team/claude-foundation "$HOME/claude-foundation"
```

(`$HOME` here is the user folder: `%USERPROFILE%` on Windows, `/Users/<name>` on macOS.) If the folder already exists, run
`git -C "$HOME/claude-foundation" pull` instead. Then read
`$HOME/claude-foundation/docs/guided-setup.md` and follow it from its first step, with the
answers from step 1 of this file.
