# zmk-config for Agar Mini BLE


https://nickcoutsos.github.io/keymap-editor/

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
