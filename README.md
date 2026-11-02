# Rair · 게임 개발 포트폴리오

C# · Unity로 게임 시스템과 개발 도구를 만들었습니다. 여러 프로젝트에서 수행한 작업 가운데, 설계 선택과 구현 과정, 검증 결과를 기술적으로 되돌아볼 수 있는 사례를 골랐습니다.

![BuilderRPG에서 생성한 섬](https://raw.githubusercontent.com/TcDRair/BuilderRPG/ff4592bf830bcffe0cffec7a73430ea75390d08b/docs/images/island-01-overview.png)

BuilderRPG의 절차적 섬 생성 결과. 시드 `20260730`, 맵 크기 512×512, 목표 육지 비율 45%, 강 14개로 생성한 지형입니다.

## 주요 사례

| 사례 | 다룬 문제 | 작업 방식 |
|---|---|---|
| [BuilderRPG · 2D 지도에서 3D 지형으로](docs/builder-rpg.md) | 공간 탐색 비용, 지형 데이터 전달, Terrain·프롭 적용 | 초기 게임 시스템 직접 구현, 후기 복구·테스트 정비에 AI 활용 |
| [여러 프로젝트에서 만든 개발 도구](docs/editor-tools.md) | 상호작용 편집, UI 색상 관리, 반복 실행과 참조 확인 | 초기 도구 직접 구현, Color Styler에 Copilot 제안 활용, 후기 정비에 AI 활용 |
| [LiC2 · 에이전트 역할 조율과 스프린트 운영](docs/lic2.md) | 하네스, 스프린트 계약과 핸드오프, 평가·재검증 | Planner·Generator·Evaluator 세션 수동 조율, 직접 플레이와 코드 확인 |

## 프로젝트와 원본

- [BuilderRPG](https://github.com/TcDRair/BuilderRPG): 절차적 지형과 캐릭터·상호작용 시스템을 다룬 개인 프로젝트입니다. [상세 기술 문서](https://github.com/TcDRair/BuilderRPG/tree/master/docs)에서 구현과 알려진 한계를 확인할 수 있습니다.
- Traveler’s Choice·CTS: 게임 프로토타입을 개발하며 UI 편집과 반복 실행을 돕는 도구를 만들었습니다. 현재 원본 저장소는 비공개이며, 주요 내용을 [개발 도구 사례](docs/editor-tools.md)에 정리했습니다.
- LiC·LiC2: AI와 1인칭 생존·제작 프로토타입을 개발했습니다. LiC2에서 역할별 세션과 스프린트 계약·평가 기록을 구체화했습니다. 현재 원본 코드와 기획 저장소는 비공개이며, [LiC2 사례](docs/lic2.md)에 조율 과정과 검증 결과를 정리했습니다.

프로젝트마다 완성도와 기여 방식이 다릅니다. 각 사례에 선행 작업, 직접 구현한 범위, AI 활용 범위와 검증의 한계를 함께 밝혔습니다.
