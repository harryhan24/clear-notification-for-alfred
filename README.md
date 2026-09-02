# Clear Notification for Alfred

An Alfred workflow that closes every notification banner currently on screen. Type `cl` (or `clear notification`) and the stack in the top-right corner is gone.

It drives the Accessibility API directly: the `NotificationCenter` process exposes a `Clear All` / `Close` action on each banner group, and the workflow performs it — no clicking, no hovering, no UI scripting of the Notification Center panel.

## Features

- `cl` — short keyword for the common case
- `clear notification` — spelled-out keyword, same action
- Grouped notifications are cleared in one shot via `Clear All`; single banners fall back to `Close`
- Repeats until the banner stack is empty (max 20 rounds), so a pile of stacked alerts clears in one invocation
- Silent when there is nothing to clear

## Limits

- Only clears banners **currently displayed**. Once macOS auto-dismisses a banner, `window "Notification Center"` no longer exists and there is nothing to act on — this does not empty the Notification Center panel's history.
- The UI element path is hardcoded (`scroll area 1 of group 1 of group 1 of window "Notification Center"`). A macOS update can move it; see AGENTS.md for how to re-derive it.

## Installation

### 개발 체크아웃 연결

repo를 클론한 뒤 심링크를 건다:

```bash
./install.sh
```

Alfred workflows 폴더에 이 repo를 심링크로 마운트한다. idempotent하고, 기존 폴더나 다른 심링크가 있으면 덮어쓰지 않고 종료한다.

### 권한

첫 실행 때 Alfred에 **손쉬운 사용(Accessibility)** 권한이 필요하다. 시스템 설정 > 개인정보 보호 및 보안 > 손쉬운 사용에서 Alfred를 허용할 것. 권한이 없으면 조용히 아무 일도 일어나지 않는다.

## Requirements

- macOS (tested on macOS 26 / Darwin 25.6)
- Alfred 5 with the Powerpack
