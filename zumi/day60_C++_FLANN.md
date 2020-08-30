# 특징점 디스크립터 이미지로 저장하기
C++에서 npy를 사용하기에는 너무 추상적이어서 형식을 맞추는 데 많은 시간을 소모하였습니다. 오늘은 특징점 디스크립터를 이미지로 저장하고, 이를 불러와 사용하도록 하겠습니다. 디스크립터를 이미지로 저장하는 이유는, C++에서 특징점을 추출할 때 디스크립터가 저장되는 자료형이 cv::Mat이기 때문입니다. XCode에서 템플릿 이미지를 불러오기 위해서는 `Hero 1.png` ~ `Hero 104.png` 파일을 PROJECT > TARGETS의 Copy Files에 추가해주어야 합니다.

![heroes](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FlyKg3%2FbtqHBR1C40d%2Fxewjk807tnsxPFHHPx1vK0%2Fimg.png)

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>

#define NUM_OF_TEMPLATE 104

using namespace std;

// 템플릿 및 카메라 프레임 특징점 추출
void extract(const cv::Mat& img, cv::Mat& des, vector<cv::KeyPoint>& kp) {
    const static auto& orb = cv::ORB::create();
    orb->detectAndCompute(img, cv::noArray(), kp, des);
}

// 템플릿 특징점 추출 및 저장
void saveDescriptors() {
    // 템플릿 경로 관련 변수
    char path[20];
    const static char srcFormat[] = "Hero %d.png";
    const static char dstFormat[] = "Des %d.png";
    
    // 특징점 추출 관련 변수
    cv::Mat img, des;
    vector<cv::KeyPoint> kp;
    
    // 이미지 읽기 및 쓰기
    for (int i=1; i<=NUM_OF_TEMPLATE; i++) {
        // 이미지 읽고 특징점 추출
        snprintf(path, 13, srcFormat, i);
        img = cv::imread(path, cv::IMREAD_GRAYSCALE);
        extract(img, des, kp);
        
        // 이미지 쓰기
        snprintf(path, 13, dstFormat, i);
        cv::imwrite(path, des);
    }
    cout << "finish extraction!" << endl;
}

int main(int argc, const char * argv[]) {
    // 특징점 이미지 저장
    saveDescriptors();
    return 0;
}
```

![디스크립터 예시](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FvmS7u%2FbtqHxcFHagV%2F33yYLaDaxMmO7Lbjf9KKHK%2Fimg.png)

# 디스크립터 불러오기
ORB로 추출하고 이미지로 저장한 Descriptor는 다음과 같은 방법으로 불러올 수 있습니다. 개인적인 생각으로는 이 방식이 `cnpy`를 이용하는 방식보다 안전하고, 편한 것 같습니다. 위 코드를 통해 Descriptor 이미지를 생성한 상태라면, 템플릿 이미지와는 달리 XCode에서 Descriptor 이미지를 불러올 필요가 없습니다. Descriptor 이미지는 빌드된 파일과 같은 폴더에 존재하기 때문입니다.

![Descriptor](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fl7FTS%2FbtqHwhHuOmo%2FdsvtfOEmsldQwaMZ63QkK0%2Fimg.png)

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>

#define NUM_OF_TEMPLATE 104

using namespace std;

// 템플릿 특징점 로드
const vector<cv::Mat> loadDescriptors() {
    // 템플릿 경로 관련 변수
    char path[20];
    const static char srcFormat[] = "Des %d.png";
    
    // 특징점 디스크립터 벡터
    vector<cv::Mat> dstDes = vector<cv::Mat>(NUM_OF_TEMPLATE);
    
    // 특징점 읽기
    for (int i=1; i<=NUM_OF_TEMPLATE; i++) {
        // 이미지 읽고 특징점 추출
        snprintf(path, 13, srcFormat, i);
        dstDes[i-1] = cv::imread(path, cv::IMREAD_GRAYSCALE);
    }
    return dstDes;
}

int main(int argc, const char * argv[]) {
//    // 특징점 이미지 저장
//    saveDescriptors();
    
    // 특징점 이미지 로드
    const auto& templateDes = loadDescriptors();
    return 0;
}
```

# 실시간 영상에서 가장 유사한 템플릿 찾기
특징점 디스크립터 두 개를 매칭하는 함수를 작성하면, 이를 지난 시간에 작성한 비디오 인풋 코드와 결합하여 실시간 영상에서 top 1 유사도를 갖는 템플릿 이미지를 유추할 수 있습니다. 파이썬에서 작성한 FLANN을 이용한 매칭 코드를 C++로 옮겨봅시다.

```py
# ORB를 위한 FLANN 설정값
FLANN_INDEX_LSH = 6
indexParams = dict(algorithm=FLANN_INDEX_LSH, table_number=6, key_size=12, multi_probe_level=1)
searchParams = dict(checks=20)
# FLANN(Fast Library for Approximate Nearest Neighbors): 특징점 매칭을 위한 라이브러리
flann = cv2.FlannBasedMatcher(indexParams, searchParams)

cnt_idx_arr = []
# 104개의 특징점에 대해 반복
for idx, des1 in enumerate(templates_des):
  # 두 이미지 간의 특징점 매칭
  matches = flann.knnMatch(des1, des2, k=2)
  # 유력 특징점의 수
  cnt = 0
  # 매칭된 특징점에 대해 반복
  for i, mn in enumerate(matches):
    # 예외 처리: 1~2순위 특징점이 없는 경우
    if len(mn) < 2: continue
    m, n = mn

    # 1순위 매칭 결과가 0.55 * 2순위 매칭 결과보다 작은 경우만 취급
    if m.distance < 0.55 * n.distance:
      cnt += 1

  cnt_idx_arr.append((cnt, idx))

for idx, (cnt, num) in enumerate(sorted(cnt_idx_arr, reverse=True)[:5]):
  print("top {i}: idx: {num}, cnt: {cnt}".format(i=idx+1, num=num, cnt=cnt))
```

위 코드를 cpp 문법으로 옮긴 (+ 함수화한) 코드는 아래와 같습니다.

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>

#define NUM_OF_TEMPLATE 104

using namespace std;

/*
* ...
*/

// Good Match Count
int match(cv::Mat templateDes, cv::Mat srcDes) {
    // ORB를 위한 FLANN 설정값
    const static auto indexParams = new cv::flann::IndexParams();
    indexParams->setAlgorithm(cvflann::FLANN_INDEX_LSH);
    indexParams->setInt("table_number", 6);
    indexParams->setInt("key_size", 12);
    indexParams->setInt("multi_probe_level", 1);
    
    const static auto searchParams = new cv::flann::SearchParams();
    searchParams->setInt("checks", 20);
    
    // 두 이미지 간의 특징점 매칭
    // FLANN(Fast Library for Approximate Nearest Neighbors): 특징점 매칭을 위한 라이브러리
    vector<vector<cv::DMatch>> matches;
    const static auto flann = cv::FlannBasedMatcher(indexParams, searchParams);
    flann.knnMatch(templateDes, srcDes, matches, 2);
        
    // 매칭된 특징점에 대해 반복
    int cnt = 0;
    for (const auto& mn: matches) {
        // 예외 처리: 1~2순위 특징점이 없는 경우
        if (mn.size() < 2) continue;
        // 1순위 매칭 결과가 0.55 * 2순위 매칭 결과보다 작은 경우만 취급
        if (mn[0].distance < 0.55 * mn[1].distance)
            cnt += 1;
    }
    return cnt;
}

int main(int argc, const char * argv[]) {
    
    // ...

    // 특징점 매칭을 위한 변수 선언
    int cnt;
    cv::Mat srcDes;
    vector<cv::KeyPoint> srcKp;
    auto cntIdxArr = vector<pair<int, int>>(NUM_OF_TEMPLATE);
    vector<pair<int, int>>::iterator top1;
    
    // 프레임 특징점 추출
    extract(frame, srcDes, srcKp);
    
    // 각 템플릿마다 반복
    for (int i=0; i<NUM_OF_TEMPLATE; i++) {
        cnt = match(templateDes[i], srcDes);
        cntIdxArr[i] = make_pair(cnt, i);
    }
    
    // 가장 유사한 이미지의 Good Match, Index 출력
    top1 = max_element(cntIdxArr.begin(), cntIdxArr.end());
    cout << "cnt: " << top1->first << ", idx: " << top1->second << endl;
    
    return 0;
}
```

이를 실시간 카메라 인풋 코드와 결합한 결과는 다음과 같습니다.

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>

#define NUM_OF_TEMPLATE 104

using namespace std;

// 템플릿 및 카메라 프레임 특징점 추출
void extract(const cv::Mat& img, cv::Mat& des, vector<cv::KeyPoint>& kp) {
    const static auto& orb = cv::ORB::create();
    orb->detectAndCompute(img, cv::noArray(), kp, des);
}

// 템플릿 특징점 추출 및 저장
void saveDescriptors() {
    // 템플릿 경로 관련 변수
    char path[20];
    const static char srcFormat[] = "Hero %d.png";
    const static char dstFormat[] = "Des %d.png";
    
    // 특징점 추출 관련 변수
    cv::Mat img, des;
    vector<cv::KeyPoint> kp;
    
    // 이미지 읽기 및 쓰기
    for (int i=1; i<=NUM_OF_TEMPLATE; i++) {
        // 이미지 읽고 특징점 추출
        snprintf(path, 13, srcFormat, i);
        img = cv::imread(path, cv::IMREAD_GRAYSCALE);
        extract(img, des, kp);
        
        // 이미지 쓰기
        snprintf(path, 13, dstFormat, i);
        cv::imwrite(path, des);
    }
    cout << "finish extraction!" << endl;
}

// 템플릿 특징점 로드
const vector<cv::Mat> loadDescriptors() {
    // 템플릿 경로 관련 변수
    char path[20];
    const static char srcFormat[] = "Des %d.png";
    
    // 특징점 디스크립터 벡터
    vector<cv::Mat> dstDes = vector<cv::Mat>(NUM_OF_TEMPLATE);
    
    // 특징점 읽기
    for (int i=1; i<=NUM_OF_TEMPLATE; i++) {
        // 이미지 읽고 특징점 추출
        snprintf(path, 13, srcFormat, i);
        dstDes[i-1] = cv::imread(path, cv::IMREAD_GRAYSCALE);
    }
    return dstDes;
}

// Good Match Count
int match(const cv::Mat templateDes, const cv::Mat srcDes) {
    // ORB를 위한 FLANN 설정값
    const static auto indexParams = new cv::flann::IndexParams();
    indexParams->setAlgorithm(cvflann::FLANN_INDEX_LSH);
    indexParams->setInt("table_number", 6);
    indexParams->setInt("key_size", 12);
    indexParams->setInt("multi_probe_level", 1);
    
    const static auto searchParams = new cv::flann::SearchParams();
    searchParams->setInt("checks", 20);
    
    // 두 이미지 간의 특징점 매칭
    // FLANN(Fast Library for Approximate Nearest Neighbors): 특징점 매칭을 위한 라이브러리
    vector<vector<cv::DMatch>> matches;
    const static auto flann = cv::FlannBasedMatcher(indexParams, searchParams);
    flann.knnMatch(templateDes, srcDes, matches, 2);
        
    // 매칭된 특징점에 대해 반복
    int cnt = 0;
    for (const auto& mn: matches) {
        // 예외 처리: 1~2순위 특징점이 없는 경우
        if (mn.size() < 2) continue;
        // 1순위 매칭 결과가 0.55 * 2순위 매칭 결과보다 작은 경우만 취급
        if (mn[0].distance < 0.55 * mn[1].distance)
            cnt += 1;
    }
    return cnt;
}

int main(int argc, const char * argv[]) {
//    // 특징점 이미지 저장
//    saveDescriptors();
    
    // 특징점 이미지 로드
    const auto templateDes = loadDescriptors();
    
    // 비디오 캡쳐 초기화
    cv::Mat frame;
    cv::VideoCapture cap(0);
    if (!cap.isOpened()) {
        cerr << "에러 - 카메라를 열 수 없습니다.\n";
        return -1;
    }

    // 특징점 매칭을 위한 변수 선언
    int cnt;
    cv::Mat srcDes;
    vector<cv::KeyPoint> srcKp;
    auto cntIdxArr = vector<pair<int, int>>(NUM_OF_TEMPLATE);
    vector<pair<int, int>>::iterator top1;
    
    // 비디오 캡쳐 시작
    while (true) {
        // 카메라로부터 캡쳐한 영상을 frame에 저장
        cap.read(frame);
        if (frame.empty()) {
            cerr << "빈 영상이 캡쳐되었습니다.\n";
            break;
        }
        // 프레임 특징점 추출
        extract(frame, srcDes, srcKp);
        
        // 각 템플릿마다 반복
        for (int i=0; i<NUM_OF_TEMPLATE; i++) {
            cnt = match(templateDes[i], srcDes);
            cntIdxArr[i] = make_pair(cnt, i);
        }
        
        // 가장 유사한 이미지의 Good Match, Index 출력
        top1 = max_element(cntIdxArr.begin(), cntIdxArr.end());
        cout << "cnt: " << top1->first << ", idx: " << top1->second << endl;
        
        // ESC 키를 입력하여 루프 종료
        if (cv::waitKey(25) >= 0)
            break;
    }
    
    return 0;
}
```