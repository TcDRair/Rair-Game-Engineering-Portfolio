# BuilderRPG · 2D 지도에서 3D 지형으로

랜덤 섬 지도를 생성하는 데서 시작해, 지도 데이터를 Unity Terrain과 프롭 배치로 연결했습니다. 시드·크기·육지 비율·강 수를 바꾸며 생성 결과를 비교하는 에디터 도구도 만들었습니다.

## 발전 과정

| 시기 | 작업 | 달라진 점 |
|---|---|---|
| 2022.05 | [랜덤 섬 생성 도입](https://github.com/TcDRair/BuilderRPG/commit/5659253581e45125758172db8cbb417c63e1d2bd) | 건설 프로토타입에 절차적 섬 생성 연결 |
| 2022.10 | [Terrain 변환 실험](https://github.com/TcDRair/BuilderRPG/commit/3ec06e03bf8c4d772dfafe45e3b41dc877721e05) | 지도·Height Map 출력 분리, 변환기 골격 추가 |
| 2023.06 | [프롭 자동 생성](https://github.com/TcDRair/BuilderRPG/commit/77a8dbc8473ff4f06c3d3568a93c1bdced701061) | 지형 데이터와 바이옴에 따른 오브젝트 배치 |
| 2023.07 | [Terrain·물 레이어](https://github.com/TcDRair/BuilderRPG/commit/b90a71cf95) | 지도 정보를 지형과 물 표현으로 연결 |
| 2023.08 | [부드러운 필드맵·물](https://github.com/TcDRair/BuilderRPG/commit/68f6c0bf5a) | 지형 표현 개선과 URP 전환 |
| 2026.07 | [Terrain 적용 검증](https://github.com/TcDRair/BuilderRPG/commit/7df42d4cba37debb9473dcd8009e1d55f6bffcb1) | 높이·해수면·레이어·프롭 접지까지 확인 |

2022~2024년 게임 시스템은 직접 구현했습니다. 2026년 복구·테스트·실행 경로 정비와 문서화에는 AI를 활용했습니다.

## 공간 탐색 범위 줄이기

Voronoi 라이브러리가 서로 다른 변에서 반환하는 같은 위치의 코너를 하나로 합쳐야 했습니다. 기존 구현은 x좌표를 키로 붙인 리스트를 LINQ로 검색해, 조회마다 전체 목록을 순회했습니다.

코너를 정수 좌표 셀별 `Dictionary<(int, int), List<Corner>>`에 보관하도록 바꿨습니다. 해당 셀과 주변 8개 셀에서 후보를 찾고, 원래 좌표 사이의 거리로 같은 코너인지 판정했습니다.

| 이전 | 변경 후 |
|---|---|
| 전체 코너 리스트에서 x좌표 조건 검색 | 인접 9개 버킷을 딕셔너리로 조회 |
| 전체 목록을 조회마다 순회 | 주변 버킷의 밀도에 따라 후보 수 결정 |

당시 동일 조건의 전후 실행 시간 측정 기록은 남아 있지 않습니다.

## 지도 데이터에서 Terrain으로

| 채널 | 정보 | 사용처 |
|---|---|---|
| R | 고도 | 하이트맵 |
| G | 습도 | 지형 데이터 전달 |
| B | 바이옴 | 지형 레이어·프롭 선택 |
| A | 육지 여부 | 점유 격자 등 후속 처리 |

![바이옴과 강을 표시한 지도](https://raw.githubusercontent.com/TcDRair/BuilderRPG/ff4592bf830bcffe0cffec7a73430ea75390d08b/docs/images/island-04-mapviz.png)

그래프의 속성을 RGBA 텍스처에 담아 Terrain 변환기로 넘겼습니다. 변환기는 높이 적용, 경계 스무딩, 프롭 배치와 8레이어 스플랫맵 적용을 담당합니다. 생성 과정을 코루틴으로 나누고 진행률·취소·재시작을 지원했습니다.

![Terrain 지상 시점](https://raw.githubusercontent.com/TcDRair/BuilderRPG/ff4592bf830bcffe0cffec7a73430ea75390d08b/docs/images/island-03-ground.png)

## 에디터와 런타임의 실행 경로 분리

생성 절차는 `IEnumerator`로 반환하고 실행 주체를 분리했습니다. 에디터에서는 `EditorCoroutine`, 플레이 모드와 빌드에서는 `MonoBehaviour.StartCoroutine`으로 같은 절차를 구동합니다.

PNG 에셋 저장은 `SaveHook`으로 분리했습니다. 런타임에서는 저장을 건너뛰고 메모리의 텍스처를 적용합니다. 새 게임 시작 UI와의 연결은 남아 있습니다.

## Terrain 적용 단계의 검증

후기 복구 과정에서는 그래프 테스트가 통과해도 Terrain 적용이 실패했습니다. 코드가 쓰는 스플랫맵은 8레이어였지만 유효 레이어는 2개만 남아, 갱신에 실패한 뒤 이전 알파맵이 표시되고 있었습니다. 높이·해수면 오프셋과 오브젝트 접지도 어긋나 있었습니다.

Terrain을 메모리에 만들고 실제 API 적용 결과를 확인하는 테스트를 추가했습니다.

## 기반과 구현 범위

Amit Patel의 Polygon Map Generation과 Stafford Williams의 Unity 이식이 기반입니다. 그래프·고도·강·습도 전파와 바이옴 분류는 선행 구현을 활용했고, 공간 해시·목표 육지 비율 조절·해양 깊이·데이터 텍스처·Terrain 변환과 생성 도구를 확장했습니다.

지형 생성·이동·상호작용을 다룬 프로토타입으로, 지형 프롭과 상호작용 시스템 사이에는 미연결 경로가 남아 있습니다.

- [상세 기술 문서와 기여 범위](https://github.com/TcDRair/BuilderRPG/blob/ff4592bf830bcffe0cffec7a73430ea75390d08b/docs/01-절차적-섬-생성.md)
- [지도 생성기 포크](https://github.com/TcDRair/polygon-map-unity/tree/ff9bbef7229bb0b6cab45437a828e27b4d56774d)
- [공통 생성 절차](https://github.com/TcDRair/BuilderRPG/blob/ff4592bf830bcffe0cffec7a73430ea75390d08b/Assets/Game%20Assets/Scripts/Fields/Map/MapGeneration.cs)

[포트폴리오로 돌아가기](../README.md)
