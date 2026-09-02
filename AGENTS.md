# AGENTS.md

이 repo는 **Alfred 워크플로**다. `info.plist` 하나가 워크플로 본체이고, AppleScript도 그 안에
`alfred.workflow.action.applescript` 오브젝트의 `applescript` 값으로 들어 있다. 별도 스크립트 파일이 없다.

## Alfred와의 연결 (symlink)

이 폴더는 Alfred의 workflows 경로에 **symlink로 마운트**되어 동작한다:

```
~/Library/Application Support/Alfred/Alfred.alfredpreferences/workflows/user.workflow.5B8B4561-C8EA-434E-A496-D9BC087258CE
  → /Users/harry/para/120_areas/128_tools/128.10_alfred-plugins/clear-notification-for-alfred
```

- 링크는 `install.sh`로 생성·검증한다 (idempotent, 기존 폴더/타 링크가 있으면 덮어쓰지 않고 종료).
- 새 머신에서는 repo clone 후 `./install.sh` 한 번 실행.
- repo 파일을 수정하면 그대로 Alfred가 읽는 워크플로가 바뀐다 — 별도 export/import 불필요.
- UUID(`5B8B4561-...`)는 Alfred가 폴더명으로 워크플로를 식별하므로 절대 바꾸지 말 것. bundleid는 `info.plist` 안에 따로 있음.

## info.plist 편집

Alfred UI로 편집해도 되고 `plistlib`으로 직접 써도 된다. 단 **Alfred를 종료한 뒤** 외부에서 쓸 것 —
Alfred가 워크플로를 메모리에 들고 있다가 종료 시 되쓰면 편집분이 날아간다. 쓴 다음 Alfred를 다시 띄우면 반영된다.

키워드는 오브젝트 하나당 하나뿐이라, `cl`과 `clear notification` 두 개의 keyword 오브젝트가 같은
AppleScript 액션으로 연결돼 있다. 키워드를 더 붙이려면 keyword 오브젝트를 추가하고 `connections`에
같은 destinationuid를 넣으면 된다.

## AppleScript 가 의존하는 UI 계층

macOS 업데이트로 깨질 수 있는 유일한 부분이다. 현재(macOS 26) 계층은:

```
window "Notification Center" → group 1 → group 1 → scroll area 1
  → group / AXNotificationCenterBanner   ← 액션 보유자
      → static text (actions=AXShowMenu)  ← 여기엔 Close 액션이 없다
```

배너 그룹 **자신**의 액션 이름이 `Name:Clear All...` / `Name:Close...` 로 노출된다. 자식(static text)이
아니라 그룹 자신에게 `perform` 해야 한다. 또 `perform (actions of ... whose name starts with ...)`
같은 복수형 필터는 `-1708`로 실패하므로, 액션을 순회하며 하나씩 `perform` 한다.

계층이 바뀌었을 때 다시 뜨는 법 — 알림을 하나 띄워 배너가 살아 있는 동안:

```bash
osascript -e 'display notification "x" with title "probe"'; sleep 2
osascript -e 'tell application "System Events" to tell process "NotificationCenter"
  set sa to scroll area 1 of group 1 of group 1 of window "Notification Center"
  return name of every action of (item 1 of UI elements of sa)
end tell'
```

`window "Notification Center"`는 배너가 화면에 떠 있는 동안에만 존재한다. 배너가 사라진 뒤 조회하면
`-1728`이 나는데 정상이다 — 프로브는 반드시 알림을 띄운 직후에 돌릴 것.

## 릴리스

`main`은 개발, `prod`가 빌드 브랜치다. `main` -> `prod` 로 승격 push 하면
`.woodpecker/release.yml`이 `.alfredworkflow`를 zip으로 말아 Forgejo 릴리스로 올린다.

- 태그는 `info.plist`의 `version`에서 딴다(`v1.0`). **같은 태그의 릴리스가 이미 있으면
  파이프라인이 실패한다** — 릴리스 전에 `version`을 올릴 것.
- zip에는 Alfred가 읽는 파일만 담는다(개발용 스크립트·문서 제외).
- 사용자 설치는 `scripts/install.sh` 원라이너가 최신 릴리스를 받아 UUID 폴더에 직접 푼다.
