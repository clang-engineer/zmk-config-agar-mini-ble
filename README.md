# zmk-config for Agar Mini BLE


https://nickcoutsos.github.io/keymap-editor/

---

# 키맵 구성

3행 + 썸 로우 구조의 40% 키보드. 레이어 5개로 구성.

## 설계 의도

**HHKB + 40% + 세벌식 390** 조합을 쓰기 위한 키맵.

- **HHKB**: `Esc` 자리에 grave-escape, `Ctrl`을 `A` 왼쪽(Caps 자리)에 배치하는 미니멀 배열
- **40%**: 물리 키를 최소화하고 숫자·기호·방향키를 레이어로 분리
- **세벌식 390**: OS 입력기로 세벌식 390을 입력하되, 그 위에서 물리 배열·기호·레이어를 세벌식/HHKB 취향에 맞춤

## 레이어 요약

| # | 레이어 | 진입 방법 | 용도 |
|---|--------|-----------|------|
| 0 | default | 기본 | QWERTY 타이핑 |
| 1 | number | 우하단 `MO1` 홀드 / 왼쪽 스페이스 썸 홀드(`LT1`) | 숫자열 + 오른손 넘패드 |
| 2 | specialA | 오른쪽 썸 `MO2` 홀드 | 괄호·문장부호 |
| 3 | specialB | 오른쪽 썸 `MO3` 홀드 | `- = \ \`` 기호 |
| 4 | arrow | **오른쪽 Shift + 맨 우하단 키** 콤보 | 방향키 + 시스템 키 |

> 범례: `·` = 비활성(none), `▽` = 아래 레이어 통과(trans)

## Layer 0 — default (QWERTY)

```
Esc   Q    W    E    R    T      Y    U    I    O    P    '    ⌫
Ctrl  A    S    D    F    G      H    J    K    L    ;    ↵
Shift Z    X    C    V    B      N    M    ·    /    Shift MO1
           Alt  Win     LT1/Spc     Spc        MO3   MO2
```

- `Esc` = grave-escape: 단독 탭은 `Esc`, Shift+탭은 `` ` `` / `~`
- `LT1/Spc` = 탭하면 Space, 홀드하면 number 레이어

## Layer 1 — number

```
Tab   1    2    3    4    5      6    7    8    9    0    ·    ▽
Caps  ·    ·    ·    ·    ·      ·    4    5    6    ·    ▽
▽     ·    ·    ·    ·    ·      0    1    2    3    ▽    ▽
```

- 상단은 숫자열, 오른손(`H J K L` / `N M , .`)이 넘패드 `4 5 6` / `0 1 2 3`

## Layer 2 — specialA

```
▽    ·    ·    ·    ·    ·      ·    ·    ·    ·    [    ]    ▽
▽    ·    ·    ·    ·    ·      ·    ·    ·    ;    '    ▽
▽    ·    ·    ·    ·    ·      ·    ,    .    /    ▽    ▽
```

## Layer 3 — specialB

```
·    ·    ·    ·    ·    ·      ·    ·    ·    -    =    \    `
▽    ·    ·    ·    ·    ·      ·    ·    ·    ·    ·    ▽
▽    ·    ·    ·    ·    ·      ·    ·    ·    ·    ▽    ▽
```

## Layer 4 — arrow

```
Sleep ·   ·    ·    ·    ·      ·    ·    ·    ·    ↑    ·    ·
▽     ·   ·    ·    ·    ·      ·    ·    ·    ←    →    ▽
▽     ·   ·    ·    ·   Boot    ·    ·    ·    ↓    ▽    ▽
```

- 오른손에 방향키(`↑ ← ↓ →`) 배치
- `Sleep`(좌상단) = mac 화면 슬립(Ctrl+Shift+Power), `Boot`(B 자리) = 부트로더 진입
- 두 키 모두 **hold-tap 가드** 적용: 짧게 누르면 무시, 300ms 이상 꾹 눌러야 실행 → 실수 트리거 방지

---

# Agar Mini BLE ZMK 펌웨어 관리 가이드

## 부트로더 진입 방법

1. 블루투스 스위치 **OFF**
2. **ESC 누른 채로** USB 연결
3. 드라이브로 인식되면 `.uf2` 파일 복사

---

## keymap-editor로 keymap 관리하기

### 기본 흐름

1. [keymap-editor](https://nickcoutsos.github.io/keymap-editor/) 에서 keymap 수정
2. GitHub에 commit & push (keymap-editor에서 직접 가능)
3. GitHub Actions 빌드 완료 대기
4. Actions → 최근 run → Artifacts에서 `.uf2` 다운로드
5. 부트로더 진입 후 플래싱

### build.yaml 설정

```yaml
---
include:
  - board: klink
    shield: agar_mini_ble
    snippet: studio-rpc-usb-uart
    cmake-args: -DCONFIG_ZMK_STUDIO=y
```

> `CONFIG_ZMK_STUDIO=y` 는 공식 가이드 옵션이므로 그대로 유지해도 됨

---

## ZMK Studio 충돌 문제

### 원인

- `CONFIG_ZMK_STUDIO=y` 활성화 시 ZMK Studio가 keymap을 flash에 별도 저장
- 새 펌웨어를 플래싱해도 Studio가 저장한 keymap이 우선 적용됨
- 결과적으로 keymap-editor에서 수정한 내용이 반영되지 않음

### 해결 방법 1 — ZMK Studio UI에서 초기화 (간단)

1. [zmk.studio](https://zmk.studio) 접속 (Chrome/Edge)
2. USB로 키보드 연결
3. **"Restore Stock Settings"** 버튼 클릭
4. flash에 저장된 Studio keymap이 초기화되고 `.keymap` 파일 내용 적용됨

### 해결 방법 2 — settings_reset 플래싱 (Studio 접속 안 될 때)

**build.yaml에 settings_reset 추가:**

```yaml
---
include:
  - board: klink
    shield: agar_mini_ble
    snippet: studio-rpc-usb-uart
    cmake-args: -DCONFIG_ZMK_STUDIO=y
  - board: klink
    shield: settings_reset
```

**플래싱 순서:**

1. 블루투스 스위치 **OFF**
2. ESC 누른 채로 USB 연결 → 부트로더 진입
3. `settings_reset-klink-zmk.uf2` 복사 → 키보드 재시작
4. 다시 ESC + USB → 부트로더 재진입
5. `agar_mini_ble-klink-zmk.uf2` 복사

---

## 주의사항

- keymap-editor로 관리 중에는 **ZMK Studio에서 keymap을 수정하지 말 것**
- Studio에서 수정하면 다시 충돌 발생 → Restore Stock Settings 또는 settings_reset 필요
- settings_reset 후에는 BLE 페어링 정보도 초기화되므로 재페어링 필요할 수 있음
