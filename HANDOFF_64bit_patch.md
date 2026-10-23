# 서비스종료 게임 64bit 패치 매뉴얼 (HANDOFF)

## 목표 (두 APK 공통)
서비스 종료로 서버가 없는 게임을 **오프라인 + 64bit 전용폰**에서 돌아가게 패치.
1. 초기 서버 인증 제거/스킵
2. 캐시샵(인앱결제) 서버 검증 스킵 + 구매 즉시 지급(0원 처리)
3. 64bit 전용폰 실행 + 최적화

대상 APK (Downloads):
- `언체인드 _ 불가능한 탈출.apk`
- `방탈출_ 더룸즈 - 프리미엄.apk`

## 분석 결과
| | 언체인드 | 더룸즈 프리미엄 |
|---|---|---|
| 패키지 | (미확인) | com.ENR_GAMEDAY.Thirteen_PUZZLE_ROOMS |
| 엔진 | Unity 2017.3.1f1, **Mono** | Unity 2019.1.6f1, **IL2CPP** |
| ABI | **armeabi-v7a만** (32bit) | arm64-v8a + armeabi-v7a (**64bit 이미 있음**) |
| 게임 코드 | `assets/bin/Data/Managed/Assembly-CSharp.dll` (IL 수정 쉬움) | `lib/arm64-v8a/libil2cpp.so` + `global-metadata.dat` (네이티브 패치) |
| 비고 | Unity IAP, ChannelPurchase/Stores(중국계 스토어 SDK) 포함 | AdMob, Google Play Games, Unity IAP. 자체 서버 로직은 아직 못 찾음 |
| 매니페스트 | - | targetSdk 27, minSdk 18, 64bit 막는 요소 없음 |

## 해야 할 작업

### A. 더룸즈 (먼저)
1. .NET 6 런타임 설치 후 Il2CppDumper 실행
   `Il2CppDumper.dll libil2cpp.so(arm64) global-metadata.dat 출력폴더`
2. dump.cs에서 `Shop.BuyFilm`, `Shop.BuyPackage`, `IAPManager`(ProcessPurchase 등), `AdManager`(RemoveAd, ShowRewardAd), GPGSManager 확인 → RVA 확보
3. arm64 `libil2cpp.so`를 해당 RVA에서 패치 (결제 없이 성공 처리 / 광고 스킵 / 인증 반환값 true)
4. apktool로 리빌드 → uber-apk-signer로 zipalign + 서명 → 64bit 폰에서 테스트
5. (필요 시) targetSdk 상향, armeabi-v7a lib 제거로 용량 최적화

### B. 언체인드 (나중)
1. `Assembly-CSharp.dll` IL 패치 (dnSpy/ILSpy 계열 또는 직접): 서버 인증 스킵, 결제 검증 스킵, 즉시 지급
2. **64bit 포팅이 최대 난관**: Unity 2017.3 Mono는 arm64 빌드가 없음
   → Unity 2018.2+ Mono arm64의 `libunity.so`/`libmono.so` + `UnityEngine.*.dll` 전체 교체 필요 (실기기 테스트 필수, 성공 보장 안 됨)
3. 리빌드 + 서명

## 도구 (작업 폴더 `tools\`)
- JDK 17 (`tools\jdk`), apktool 3.0.3 jar, uber-apk-signer 1.3.0 jar, Il2CppDumper net6 v6.7.46
- **미설치: .NET 6 런타임** (업무 PC는 정책상 설치 차단됨 → 개인 PC에서 진행)
- 디코딩 결과: `apkwork\rooms_d`, 압축해제: `apkwork\rooms_x`, `apkwork\unchained_x`

## 진행 상태
- [x] 도구 다운로드, 더룸즈 디코딩, 매니페스트/메타데이터 문자열 1차 분석
- [ ] Il2CppDumper 덤프 (.NET 6 필요)
- [ ] 더룸즈 패치/리빌드/서명
- [ ] 언체인드 전체
