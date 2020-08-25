# Kaggle?
Kaggle은 설치가 필요하지 않고, 커스텀 가능한 주피터 환경을 제공하는 서비스입니다. 무료 GPU를 제공하여 데이터 분석과 머신러닝을 위해 사용하는 경우도 있지만, 더욱 유명한 건 역시 Kaggle Competition이겠죠.

머신러닝에 특화된 플랫폼이다보니 유저 중에는 데이터 분석가가 많고, 기업들은 유저들에게 데이터 셋과 목표(그리고 어마어마한 상금)를 제공하는 대회를 엽니다.

![캐글](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FQghfM%2FbtqHjqKaxVw%2FdI23JiF5GqsbfLg99mGJ2K%2Fimg.png)

오늘은 이 중 갓 열린, 3개월의 기간 동안 도전할 수 있는 Lyft 사의 자율주행을 위한 모션 예측 대회를 소개하려고 합니다.

![car](https://storage.googleapis.com/kaggle-media/competitions/Lyft-Kaggle/Motion%20Prediction/BP9I1484%20(1).jpg)

# Lyft Motion Prediction for Autonomous Vehicles
https://www.kaggle.com/c/lyft-motion-prediction-autonomous-vehicles/overview/description

위 대회는 자율주행 자동차를 위한 모션 예측 인공지능 모델을 구축하는 대회로, 상금 30,000 달러가 걸려있습니다. 다음은 대회에 대한 설명입니다.

# Description
자율주행 자동차(AVs)는 미래의 운송업을 드라마틱하게 바꿔놓을 전망입니다. 하지만, 자율주행 자동차의 이점을 현실화 하기에는 아직까지 풀지 못한 중요한 엔지니어링 문제가 남아있습니다. 그 중 하나는 AV 주변의 자동차, 자전거, 보행자의 움직임을 예측하는 모델을 만드는 것입니다.

차량 공유 기업 Lyft는 자율주행 챌린지와 완전 자율주행 시스템을 구축하기 위해 [Level 5](https://level5.lyft.com/)를 시작했습니다. ([현재 고용 중입니다!](https://www.lyft.com/careers?category=autonomous%2520vehicles)) 이들의 [이전 대회](http://kaggle.com/c/3d-object-detection-for-autonomous-vehicles/) 과제는 참가자가 3D 물체를 인식하는, 움직임을 포착하기 이전의 중요한 단계였습니다. 이제, 이러한 교통 요소의 움직임을 예측해보세요.

이 대회에서, 데이터 사이언스 스킬을 사용하여 자율주행 자동차를 위한 모션 예측 모델을 만들어보세요. 이전에 공개되었던 데이터보다 큰 [예측 데이터 셋](https://self-driving.lyft.com/level5/prediction/)이 모델 학습과 테스트를 위해 제공됩니다. 머신러닝에 대한 지식을 사용해 AV의 환경에서 자동차와 자선거, 보행자가 어디로 움직일지 예측하세요.

Lyft의 사명은 세계 최고의 운송으로 사람들의 삶을 개선하는 것입니다. 그들은 자율주행 자동차가 운송을 더 안전하고, 환경 친화적이고, 접근성 높게 만드는 미래를 믿습니다. 그들의 목표는 연구자 간 데이터를 공유하여 개발을 가속하는 것입니다. 당신의 참가를 통해 산업이 발전하여 자율주행 자동차의 이점을 더 빠르게 누릴 수 있습니다.

> 위 대회는 코딩 대회입니다. 자세한 사항은 [코드 요구사항](https://www.kaggle.com/c/lyft-motion-prediction-autonomous-vehicles/overview/code-requirements)을 확인하세요.

# Evaluation
이 대회의 목표는 다른 교통 참가자의 궤적을 예측하는 것입니다. uni-modal 모델을 사용하여 한 샘플 당 하나의 예측 결과를 생성해도 되며, multi-modal 모델을 사용하여 여러 개의 가설을 생성해도 됩니다. (3개까지) - 그 이상은 confidence vector에 기술되어 있습니다.

교통 상황에 따른 높은 multi-modality와 애매모호함 때문에, 이 대회에서 점수를 측정하기 위한 평가 기준은 여러 예측을 설명하도록 조정됩니다.

> Note: 다음 내용은 우리의 [L5Kit 레포지토리의 측정 방식 페이지](https://github.com/lyft/l5kit/blob/master/competition.md)에서 간략하게 발췌한 내용입니다.

주어진 multi-modal prediction에 대한 ground truth 데이터의 negative log-likelihood를 계산합니다. 자세히 들여다 봅시다. 샘플 궤적에 대한 ground truth 위치가 다음과 같다고 가정해봅시다.

![ground truth positions of a sample trajectory](https://latex.codecogs.com/gif.latex?%5Cbg_white%20%5Clarge%20x_1%2C%20%5Cldots%2C%20x_T%2C%20y_1%2C%20%5Cldots%2C%20y_T)

그리고 다음과 같이 표기하는 K개의 hypothese를 예측했다고 가정합니다.

![K hypotheses](https://latex.codecogs.com/gif.latex?%5Cbg_white%20%5Clarge%20%5Cbar%7Bx%7D_1%5Ek%2C%20%5Cldots%2C%20%5Cbar%7Bx%7D_T%5Ek%2C%20%5Cbar%7By%7D_1%5Ek%2C%20%5Cldots%2C%20%5Cbar%7By%7D_T%5Ek)

추가로, K개의 hypotheses를 대상으로 confidence c를 예측합니다. ground truth positions이 다음과 같은 likelihood를 갖는 시간에 따라 다차원 독립 Normal distribution을 따른다고 생각하면,

![likelihood1](https://latex.codecogs.com/gif.latex?%5Cbg_white%20%5Clarge%20p%28x_%7B1%2C%20%5Cldots%2C%20T%7D%2C%20y_%7B1%2C%20%5Cldots%2C%20T%7D%7Cc%5E%7B1%2C%20%5Cldots%2C%20K%7D%2C%20%5Cbar%7Bx%7D_%7B1%2C%20%5Cldots%2C%20T%7D%5E%7B1%2C%20%5Cldots%2C%20K%7D%2C%20%5Cbar%7By%7D_%7B1%2C%20%5Cldots%2C%20T%7D%5E%7B1%2C%20%5Cldots%2C%20K%7D%29)

![likelihood2](https://latex.codecogs.com/gif.latex?%5Cbg_white%20%5Clarge%20%3D%20%5Csum_k%20c%5Ek%20%5Cmathcal%7BN%7D%28x_%7B1%2C%20%5Cldots%2C%20T%7D%7C%5Cbar%7Bx%7D_%7B1%2C%20%5Cldots%2C%20T%7D%5E%7Bk%7D%2C%20%5CSigma%3D1%29%20%5Cmathcal%7BN%7D%28y_%7B1%2C%20%5Cldots%2C%20T%7D%7C%5Cbar%7By%7D_%7B1%2C%20%5Cldots%2C%20T%7D%5E%7Bk%7D%2C%20%5CSigma%3D1%29)

![likelihood3](https://latex.codecogs.com/gif.latex?%5Cbg_white%20%5Clarge%20%3D%20%5Csum_k%20c%5Ek%20%5Cprod_t%20%5Cmathcal%7BN%7D%28x_t%7C%5Cbar%7Bx%7D_t%5Ek%2C%20%5Csigma%3D1%29%20%5Cmathcal%7BN%7D%28y_t%7C%5Cbar%7By%7D_t%5Ek%2C%20%5Csigma%3D1%29)

이때 loss는 다음과 같아집니다.

![loss1](https://latex.codecogs.com/gif.latex?%5Cbg_white%20%5Clarge%20L%20%3D%20-%20%5Clog%20p%28x_%7B1%2C%20%5Cldots%2C%20T%7D%2C%20y_%7B1%2C%20%5Cldots%2C%20T%7D%7Cc%5E%7B1%2C%20%5Cldots%2C%20K%7D%2C%20%5Cbar%7Bx%7D_%7B1%2C%20%5Cldots%2C%20T%7D%5E%7B1%2C%20%5Cldots%2C%20K%7D%2C%20%5Cbar%7By%7D_%7B1%2C%20%5Cldots%2C%20T%7D%5E%7B1%2C%20%5Cldots%2C%20K%7D%29)

![loss2](https://latex.codecogs.com/gif.latex?%5Cbg_white%20%5Clarge%20%3D%20-%20%5Clog%20%5Csum_k%20e%5E%7B%5Clog%28c%5Ek%29%20-%5Cfrac%7B1%7D%7B2%7D%20%5Csum_t%20%28%5Cbar%7Bx%7D_t%5Ek%20-%20x_t%29%5E2%20+%20%28%5Cbar%7By%7D_t%5Ek%20-%20y_t%29%5E2%7D)

## Submission File
> Note: [L5Kit](https://github.com/lyft/l5kit)를 사용하고 있다면, 예측 결과(single and multi-modal)를 유효한 CSV로 직접 변환하는 [함수](https://github.com/lyft/l5kit/blob/20ab033c01610d711c3d36e1963ecec86e8b85b6/l5kit/l5kit/evaluation/csv_utils.py#L140)를 제공하고 있습니다.

모든 요소는 `track_id`와 `timestamp`로 구분됩니다. 각각의 궤적은 50개의 2D `(X, Y)` 예측 결과를 가지고 있습니다.

테스트 셋에서 각각의 요소에 대해 최대 3개까지 궤적을 예측할 수 있습니다. CSV 파일 포맷을 지키기 위해, single-modal이라 할지라도 3개의 궤적 필드 모두 값을 가지고 있어야 합니다. 그러나, 각각의 궤적은 고유한 confidence를 가지고 있으며, 이를 0으로 설정하면 성능을 평가할 때 하나 이상의 궤적을 완전히 무시할 수 있습니다. 세 confidence의 합은 반드시 1이 되어야 합니다.

유효한 CSV 헤더의 예시:
```csv
timestamp, track_id, conf_0, conf_1, conf_2, coord_x00, coord_y_00,...,coord_x049, coord_y_049, coord_x10, coord_y_10,...,coord_x149, coord_y_149, coord_x20, coord_y_20,...,coord_x249, coord_y_249
```

# Timeline
- 2020/11/18 - 참가 종료. 대회에 참가하려면 이 시점 이전에 반드시 대회 규정을 수락해야 합니다.
- 2020/11/18 - 팀 합병 데드라인. 참가자가 팀에 합류하거나 팀을 합칠 수 있는 마지막 날입니다.
- 2020/11/25 - 마지막 제출일. 

모든 데드라인은 별도 기재되지 않은 한 세계 시간 기준으로 오후 11:59까지입니다. 대회 주최자가 필요하다고 판단한 경우 대회 기간을 수정할 수 있습니다.

# Prizes
## 우승 상금
개인 리더보드에 우수한 성적을 기록한 참가자는 다음 상금을 받을 자격이 있습니다.

- 1등 - $12,000
- 2등 - $8,000
- 3등 - $6,000
- 4등 - $4,000

## 추가 기회
벤치마크를 능가하면 GCP 크래딧 $300를 받을 수 있습니다! 호스트 밴치마크보다 높은 점수를 받은 참가자는 다음 [설문 조사](https://www.kaggle.com/GCP_Credits_Form-Lyft2020_082420)를 작성하고 GCP 쿠폰 코드를 받을 수 있습니다. 요청 기한은 2020/10/30입니다. 제한된 수량의 쿠폰이며, 한 사람 당 하나의 쿠폰을 받을 수 있습니다.

# Code Requirements
## 해당 대회는 코딩 대회입니다.
제출물은 반드시 Jupyter Notebook이어야 합니다. Notebook 내에서 학습할 필요는 없습니다.

커밋 이후 "Submit to Competition" 버튼이 활성화 되려면 다음 조건을 반드시 만족해야 합니다:
- CPU Notebook은 실행 시간이 9 시간 이내여야 합니다.
- GPU Notebook은 실행 시간이 9 시간 이내여야 합니다.
- TPUs는 학습할 때에만 사용 가능합니다. TPU Notebook은 실행 시간이 3 시간 이내여야 합니다.
- 인터넷 연결은 허용되지 않습니다.
- 미리 학습된 모델을 비롯한, 무료 & 공공 목적으로 공개된 외부 데이터는 허가됩니다.
- 제출 파일 이름은 반드시 `submission.csv`여야 합니다.

제출하는 방법에 대해 더 알아보려면 [코딩 대회 FAQ](https://www.kaggle.com/docs/competitions#kernels-only-FAQ)를 확인하세요.