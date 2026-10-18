# 프로젝트 8. 생성형 AI를 활용한 제조 결함 데이터 생성

## 개요
- 목표: Stable Diffusion으로 문장에서 그림을 만들고 마스크 부분만 다시 그려 보고(8-1), AnomalyAny로 정상 사진과 결함 문장에서 결함 사진을 만든다(8-2). 이어서 최신 이미지 모델인 FLUX.2 [klein]과 비교하고, 만든 결함을 PaDiM으로 확인한다.
- 모델: Stable Diffusion 1.5 (Text-to-Image, Inpainting), [AnomalyAny](https://github.com/EPFL-IMOS/AnomalyAny) (CVPR 2025), 비교용 FLUX.2 [klein] 4B (distilled, base), 확인용 PaDiM
- 실행 환경: Google Colab A100 GPU

## 파일 구성
| 파일 | 내용 |
|---|---|
| [`08-1_interpreter.ipynb`](08-1_interpreter.ipynb) | Stable Diffusion 실습 코드 + 정리 + Test 1~2 분석 |
| [`08-2_interpreter.ipynb`](08-2_interpreter.ipynb) | AnomalyAny 결함 생성 실습 코드 + 정리 + Test 1~2 분석 + 개인 회고 (8-1·8-2) |

## 노트북 구성
### 08-1
1. 실행 준비 (패키지 설치)
2. Text-to-Image: 문장으로 그림 만들기
3. Inpainting: 마스크 부분만 다시 그리기
4. 추가 실험
   - Test 1: 같은 프롬프트·입력 이미지·마스크로 SD 1.5와 FLUX.2 [klein] 비교 (프롬프트를 따르는 정도, 마스크 밖 변화)
   - Test 2: FLUX.2 [klein] distilled(4단계)와 base(50단계)의 시간과 결과 비교

### 08-2
1. 실행 준비 (패키지 설치, AnomalyAny 코드 받기)
2. 결함 생성: 탁자 색 바램, 헤이즐넛 균열, PCB 균열
3. 추가 실험
   - Test 1: AnomalyAny와 FLUX.2 [klein] 인페인팅이 균열을 넣으면서 기판을 얼마나 그대로 두는지 비교
   - Test 2: 정상 pcb4로 학습한 PaDiM(6번과 같은 설정)이 어느 쪽을 더 불량으로 보는지, 이상 맵이 균열 자리를 가리키는지 확인
4. 개인 회고 (KPT, AAR)

## 실행 방법
1. Colab에서 노트북 열기 (런타임: A100 GPU)
2. 08-2는 `normal.png`(VisA pcb4 `Normal/0001`)와 `fc75.png`([6번 레포의 FC-75 인쇄 템플릿](https://github.com/kcyoow/6_AnomalyDetection_Practice/blob/main/scripts/fc75.png))를 `/content`에 올리기
3. 위에서부터 차례로 실행

## 회고
### KPT
- Keep: 새 모델(FLUX)을 쓸 때 원본 실습과 같은 프롬프트·이미지·마스크로 비교해서 차이를 바로 볼 수 있었다. 다음에도 모델을 비교할 때는 조건을 같게 맞춰 볼 것이다.
- Problem: 스테이블 디퓨전은 시드를 고정하지 않아서 이전 실험에서 같은 코드인데도 말이 나왔다 안 나왔다 해서, 한 장만으로 비교하기는 어려웠고, AnomalyAny는 결함을 넣으면서 기판 전체를 다시 그려 FC-75 인쇄가 사라졌고, PaDiM이 불량이라고 본 것도 균열보다는 망가진 USB 단자 때문이었다. FLUX는 기판은 그대로 뒀지만 같은 문장으로는 아무리 마스크를 넖혀도 마스크 안에 균열을 그리지 않았다.
- Try: 모델을 비교할 때는 시드를 고정해서 진행 할 것이고, 인페인팅을 할 때는 사진 전체 설명 대신 마스크 안에 그릴 결함을 문장으로 적어서 다시 생성해 볼 것이고, 생성 결과는 점수만 보지 않고 이상 맵 위치와 사진을 같이 확인할 것이다.

### AAR
- 목표: 생성형 AI로 제조 결함 데이터를 만드는 실습을 해 보고, 최신 모델인 FLUX.2와 비교해 기판을 지키면서 결함을 넣을 수 있는지 확인한다.
- 결과: FLUX는 프롬프트를 잘 따랐고(말 탄 우주인, 보물 상자) 마스크 밖도 SD보다 덜 바꿨으며, distilled는 base보다 몇십배 빨랐다. AnomalyAny는 기판 전체가 바뀌어 인쇄가 사라졌고, FLUX는 기판은 그대로였지만 균열이 그려지지 않아 PaDiM도 정상으로 봤다.
- 차이 원인: 요번에 사용된 스테이블 디퓨전이 오래된 모델이라 문장을 덜 따랐고, 시드를 정하지 않아 결과가 매번 달랐다. AnomalyAny는 마스크 없이 사진 전체를 다시 그리는 방식이라 기판이 망가졌다. FLUX 인페인팅은 마스크 안을 문장대로 다시 그리는데, AnomalyAny와 같은 사진 전체 설명 문장을 써서 마스크 안을 원래 IC로 채웠다.
- 다음 액션: FLUX에 결함만 설명하는 문장과 실제 결함 사진을 줘서 다시 생성하고, 만든 결함으로 이상 탐지 모델을 학습·평가해 쓸모를 확인한다.
