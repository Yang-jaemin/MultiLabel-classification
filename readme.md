# Multilabel Classification

## 프로젝트 개요

- 감염자의 입, 호흡기로부터 나오는 비말, 침 등으로 인해 다른 사람에게 쉽게 전파가 될 수 있기 때문에 감염 확 산 방지를 위해 무엇보다 중요한 것은 모든 사람이 마스크로 코와 입을 가려서 혹시 모를 감염자로부터의 전파 경로를 원천 차단하는 것입니다. 하지만 넓은 공공장소에서 모든 사람들의 올바른 마스크 착용 상태를 검사하 기 위해서는 추가적인 인적자원이 필요할 것입니다.

  따라서, 우리는 카메라로 비춰진 사람 얼굴 이미지 만으로 이 사람이 마스크를 쓰고 있는지, 쓰지 않았는지, 정 확히 쓴 것이 맞는지 자동으로 가려낼 수 있는 시스템이 필요합니다.

  문제 정의

  1. 마스크 착용 여부 : 미착용, 올바르지 않은 착용, 정상 착용 2. 성별 : 남, 여
  2. 연령대 : 30이하, 30초과 60이하, 60초과

  마스크 착용 여부, 성별, 연령대 총 3, 2, 3가지 분류 문제를 분류하여 총 3 * 2 * 3 = 18 개의 클래스로 분류 예측 을 해결해야 합니다.

## 데이터

- Train data
  - 4500명의 7장(마스크 미착용 1장, 올바르지 않은 착용 1장, 정상 착용 5장)의 이미지 데이터
  - 사이즈 : 384 x 512
-  Train.csv 
  - id, gender, race, age, path으로 구성된 csv 파일
- Test data
  - 12600장의 Label이 알려지지 않은, 마스크 착용여부와 성별, 연령 미상의 랜덤한 이미지 데이터
- Output
  - 각 ImageID들에 대해 총 18개의 클래스 중 해당 이미지에 대한 예측 클래스 ans로 구성된 csv파일



## EDA

<img width="831" alt="스크린샷 2023-08-18 오후 9 16 24" src="https://github.com/Yang-jaemin/MultiLabel-classification/assets/108872973/cfc35d07-d427-4ff1-8ee2-b1d6b5661ad0">

- Female에 치중된 데이터
- 10-20-50대의 데이터 많음, 30-40-60대의 데이터 적음
- 40-50대에서 특히 두드러지는 성별 불균형

<img width="328" alt="스크린샷 2023-08-18 오후 9 17 28" src="https://github.com/Yang-jaemin/MultiLabel-classification/assets/108872973/f10c2ee9-466e-41ee-a127-6c01a47ce331">

- 19-20세에 치중된 30대 이하 데이터 -> 30대 이하 특징을 잡아내기에는 오히려 나쁘지 않을 것 같음
- 매우 적은 데이터를 가지는 30대-40대와 50-60대에 치중된 데이터 -> 30~60대와 60대 이상을 구분하는 특징을 잡아내기에 어려움이 있을 것
- 특히 60세에만 집중되어 있는 60대 이상의 데이터 - > 60대 이상에 대한 특징을 잡아내는데에 어려움이 있을 것이라고 추정

EDA 결론: 핵심적으로 추정되는 문제는 30~60대와 60대 이상, 특히 60대 이상에 대한 분류 성능이라고 판단



## 모델 결정 및 구현

EfficientNet, ResNet, ViT

- 동일한 데이터 및 전처리 조건 하에 하이퍼 파라미터와 pretrained 여부를 조정해가며 성능을 확인

- ViT는 학습 시간이 너무 오래 걸리며 pretrained 모델 역시 타 모델에 비해 동일 epoch에 대해서 떨어지는 성능을 보임
- EfficientNet과 ResNet의 대조 중 ResNet50이 가장 높은 성능을 보임



## 실험

Data Augmentation 부분에서 가장 좋은 결과를 낸 모델의 경우 GausianNoise, RandomHorizontalFlip으로 유의미한 성능 향상이 보였으며, Gusianblur, ColorJitter등 추가적으로 적용하였을 경우 눈에 띄는 성능향상을 보이지 않았음

-> 인물의 이미지라는 점에서 좌우 반전은 성능향상에 유효하다고 판단하였고, 주름이나 얼굴 조형 등의 특징 참조에 있어서 가우시안 노이즈 추가가 유효할 것이라고 판단하였음

<img width="722" alt="스크린샷 2023-08-18 오후 9 39 58" src="https://github.com/Yang-jaemin/MultiLabel-classification/assets/108872973/c8771081-bae8-48c1-9864-de79c7b1226a">

<img width="723" alt="스크린샷 2023-08-18 오후 9 39 42" src="https://github.com/Yang-jaemin/MultiLabel-classification/assets/108872973/985a8c91-ec1b-47c0-97be-413adea8dc1e">



- 배경 제거의 경우 분명히 효과를 보일 것이라고 추정하였고 각 개별 이미지에서 추출한 feature로 GradCAM을 통해 확인해본 결과 분명 배경에 대한 참조가 이루어지고 있는 것을 확인

- 어째서인지 이미지의 배경제거를 통한 성능 개선 효과는 전혀 이루어지지 않았음. 성능 개선 효과가 없는지에 대한 파악이 이루어지지 못한 점이 아쉬움

- 의복 및 기타 신체부위를 통한 성능 개선 효과가 있는지 때조해보기 위해 Segmentation을 통해 의상 영역 역시 제외하여 얼굴 영역만을 추출하여 성능 개선을 시도하였으나 시간의 제약과 구현의 어려움으로 인해 실패함

<img width="761" alt="스크린샷 2023-08-18 오후 9 43 47" src="https://github.com/Yang-jaemin/MultiLabel-classification/assets/108872973/eeba54f7-e8dc-42e4-ab9f-7cc05769916f">

- Model이 어떤 부분을 보고 예측하는지 알아보기 위해 GradCAM을 사용함
- 마스크를 예측할 때 입 주변에서 가장 큰 반응을 보임
- 성별을 예측할 때는 머리카락 또는 머리 주변에서 가장 큰 반응을 보임
- 나이를 예측할 때는 이마 주변에서 가장 큰 반응을 보임



이를 통해 얼굴이 아닌 주변 배경은 제거를 하는 것이 불필요한 노이즈를 줄일 수 있을 것 같다고 판단했고

같은 얼굴 부분에서 특징을 뽑아낸다면 모델을 분리해서 학습할 필요가 없다고 생각하여

 하나의 모델로 학습하는 것이 적절하다고 판단함<img width="396" alt="스크린샷 2023-08-18 오후 9 48 17" src="https://github.com/Yang-jaemin/MultiLabel-classification/assets/108872973/e5f147b1-ec25-40c1-8cb4-3e5977e67eef">
- Rembg 라이브러리를 이용하여 모든 데이터의 배경을 제거했고 60세 데이터를 증강 시키고자 60세 얼굴의 코 위 부분을 잘라내어 다른 데이터에 합성시켜서 데이터를 증강시킴 -> 약간의 성능 향상이 있었음

<img width="389" alt="스크린샷 2023-08-18 오후 9 49 40" src="https://github.com/Yang-jaemin/MultiLabel-classification/assets/108872973/1e5b54ad-2ce1-4492-9328-0d6fd5aebca0">

​																					< 데이터 증강 전 ><img width="385" alt="스크린샷 2023-08-18 오후 9 50 23" src="https://github.com/Yang-jaemin/MultiLabel-classification/assets/108872973/e0174815-45b7-4a90-9f0c-fc6f06ce4506">
​											<데이터 증강 후>
