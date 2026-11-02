# BuilderRPG · 2D 지도에서 3D 지형으로

랜덤 섬 지도를 생성하는 데서 시작해, 그 데이터를 Unity Terrain과 프롭 배치에 사용하는 파이프라인으로 확장했습니다. 지도 생성 알고리즘 자체보다, 기존 구현을 게임에 사용할 수 있는 형태로 바꾸며 해결한 문제에 초점을 맞췄습니다.

## 발전 과정

| 시기 | 커밋에서 확인한 작업 | 달라진 점 |
|---|---|---|
| 2022.05 | [랜덤 폴리곤 섬 생성 도입](https://github.com/TcDRair/BuilderRPG/commit/5659253581e45125758172db8cbb417c63e1d2bd) | 기존 건설 프로토타입에 절차적 섬 생성 연결 |
| 2022.10 | [Terrain 변환 실험 시작](https://github.com/TcDRair/BuilderRPG/commit/3ec06e03bf8c4d772dfafe45e3b41dc877721e05) | 지도와 별도로 Height Map 출력을 다루고 TerrainGenerator 골격 추가 |
| 2023.06 | [Terrain 프롭 자동 생성](https://github.com/TcDRair/BuilderRPG/commit/77a8dbc8473ff4f06c3d3568a93c1bdced701061) | 지형 데이터와 바이옴에 따라 오브젝트를 배치하는 경로 확장 |
| 2023.07 | [Terrain·물 레이어 생성](https://github.com/TcDRair/BuilderRPG/commit/b90a71cf95) | 지도 정보를 지형과 물 표현으로 연결 |
| 2023.08 | [부드러운 필드맵·물](https://github.com/TcDRair/BuilderRPG/commit/68f6c0bf5a) | 지형 표현을 다듬고, 이후 URP 전환과 그리드 작업으로 진행 |
| 2026.07 | [Terrain 적용 단계 검증 보강](https://github.com/TcDRair/BuilderRPG/commit/7df42d4cba37debb9473dcd8009e1d55f6bffcb1) | 그래프 결과뿐 아니라 높이·해수면·레이어·프롭 접지까지 확인 |

2022~2024년 게임 시스템은 직접 구현했습니다. 2026년의 복구, 어셈블리 정비, 테스트와 문서화에는 AI 도구를 활용했습니다. 위 이력은 서로 다른 시기의 작업을 구분해 보여 줍니다.

## 공간 탐색 범위를 줄이기

Voronoi 라이브러리는 같은 위치의 코너를 서로 다른 변에서 별개 객체로 반환합니다. 그래프를 만들 때 이 코너들을 하나로 합치는 정규화가 필요했습니다.

기존 구현은 x좌표를 키로 붙인 리스트를 LINQ로 검색했습니다. 키가 붙어 있어도 자료구조가 리스트이므로, 조회마다 전체 원소를 훑었습니다. 맵 크기를 키우면 이 반복 비용도 커졌습니다.

좌표를 정수 격자에 나누고, `Dictionary<(int, int), List<Corner>>`에 코너를 보관하도록 바꿨습니다. 조회할 때는 해당 셀과 주변 8개 셀만 확인하고, 실제 거리로 같은 코너인지 판정했습니다.

| 이전 | 변경 후 |
|---|---|
| 조회마다 전체 코너 리스트에서 x좌표 조건 검색 | 인접 9개 버킷을 딕셔너리로 조회 |
| 후보가 맵 전체 코너 수에 따라 증가 | 후보가 주변 버킷의 밀도에 따라 결정 |

원본과 변경 후의 정량 실행 시간을 같은 조건에서 측정한 기록은 없습니다. 따라서 배속 수치 대신 검색 대상과 자료구조가 어떻게 달라졌는지로 개선을 설명합니다.

## 지도와 Terrain 사이의 데이터

| 채널 | 저장한 정보 | 사용처 |
|---|---|---|
| R | 고도 | Terrain 하이트맵 |
| G | 습도 | 지형 데이터 전달 |
| B | 바이옴 | 지형 레이어·프롭 선택 |
| A | 육지 여부 | 점유 격자 등 후속 처리 |

사람이 보는 바이옴 지도와 변환기가 읽는 데이터 텍스처를 분리했습니다. 그래프의 필요한 속성을 RGBA 텍스처로 넘겨, 생성 단계와 Terrain 적용 단계의 연결 지점을 만들었습니다.

![바이옴 지도](https://raw.githubusercontent.com/TcDRair/BuilderRPG/ff4592bf830bcffe0cffec7a73430ea75390d08b/docs/images/island-04-mapviz.png)

바이옴과 강을 표시한 지도입니다.

![Terrain 지상 시점](https://raw.githubusercontent.com/TcDRair/BuilderRPG/ff4592bf830bcffe0cffec7a73430ea75390d08b/docs/images/island-03-ground.png)

지도 데이터를 Terrain과 바이옴별 프롭 배치에 적용한 결과입니다.

Terrain 변환기는 높이 적용, 경계 스무딩, 프롭 배치, 8레이어 스플랫맵 적용을 담당합니다. 생성은 코루틴으로 나누고 에디터에서 단계별 진행률과 취소·재시작을 다뤘습니다.

## 실제 적용까지 확인해야 했던 이유

후기 복구 과정에서는 그래프 단계 테스트가 통과해도 Terrain 적용이 실패할 수 있음을 확인했습니다. 코드가 쓰는 스플랫맵은 8레이어인데 유효 Terrain 레이어가 2개만 남아, 화면에 이전 알파맵이 표시되는 문제가 있었습니다. 높이와 해수면 오프셋, 씬 오브젝트의 접지 상태도 함께 어긋나 있었습니다.

AI와 진행한 정비에서는 Terrain을 메모리에 만들고 실제 API 적용 결과를 검사하는 테스트를 추가했습니다. 이 기록은 초기 구현의 기능과 후속 검증에서 드러난 한계를 함께 보여 줍니다.

## 기여 범위와 현재 상태

Amit Patel의 Polygon Map Generation과 Stafford Williams의 Unity 이식이 기반입니다. Voronoi 그래프, 고도·강·습도 전파와 바이옴 분류의 골격은 선행 작업에 해당합니다.

제가 확장한 부분은 공간 해시, 목표 육지 비율 조절, 해양 깊이, 데이터 텍스처, Terrain 변환과 에디터 생성 흐름입니다. 지형 프롭과 상호작용 시스템 사이에는 아직 연결되지 않은 부분이 있으며, 생성·이동·상호작용이 존재하는 것이 완성된 게임 루프를 뜻하지는 않습니다.

- [전체 기술 문서와 기여 범위](https://github.com/TcDRair/BuilderRPG/blob/ff4592bf830bcffe0cffec7a73430ea75390d08b/docs/01-절차적-섬-생성.md)
- [포크한 지도 생성기](https://github.com/TcDRair/polygon-map-unity/tree/ff9bbef7229bb0b6cab45437a828e27b4d56774d)
- [시스템 연결 상태](https://github.com/TcDRair/BuilderRPG/blob/ff4592bf830bcffe0cffec7a73430ea75390d08b/README.md)

[포트폴리오로 돌아가기](../README.md)
