---
layout: post
title: [Image processing] OpenCV + Python 조합으로 아이돌영상 Object tracking !
comments: true
---



***수 많은 멤버가 나오는 영상이나 직캠에서 내가 좋아하는 멤버만 골라서 볼 수 있다면 기쁘지 아니한가?***



# 1. 설명



[OpenCV](https://opencv.org/)는 영상처리를 대중화시킨 1등 공신이다. 영상처리 입문은 즉 OpenCV 입문일 정도이다. 최근에는 오류감지 Detection 영역에서 Machine/Deep Learning과 결합해 좋은 성과를 거두고 있고, 현재까지도 꾸준히 발전하고 있는 라이브러리이다.

더욱이, [BSD 라이선스](https://namu.wiki/w/BSD 라이선스)를 사용하므로 상업적으로도 이용 가능하다.

이를 활용해 좋아하는 아이들 영상에서 특정 Member를 대상으로 추척해, 해당 인물만 보여주는 간단한 프로그램을 만들 것이다.



#### **아래 사진과 같이!**

![IMAGE 054](https://user-images.githubusercontent.com/52769104/103894573-78183880-5132-11eb-84c6-0dc89d236345.png)



# 2. 사용 Tools, Libraries와 참고/사용영상

- Python 3.8
- OpenCV-Python 4.5.1.48 (opencv 사용을 위해 필요)
- OpenCV-Contrib-Python 4.5.1.48 (Traking 알고리즘 사용을 위해 필요)
- Numpy 1.19.5
- [[K-Choreo 6K] 빵빵즈 직캠 'Psycho(원곡:Red Velvet)' (00s Choreography) l @MusicBank 200626](https://www.youtube.com/watch?v=5GPdIfn7Rzc) (https://www.youtube.com/watch?v=5GPdIfn7Rzc)
- [물체추적으로 BTS 오빠를 따라다니는 직캠을 만들어보자 - Python](https://www.youtube.com/watch?v=cx7VONjFEE0) (https://www.youtube.com/watch?v=cx7VONjFEE0)





# 3. 진행 순서

- [ ] **필요 라이브러리 로드 **

- [ ] **영상을 띄울 Window 세팅 **

- [ ] **ROI 설정 **

- [ ] **Object Tracking **

- [ ] **Tracking Window 재설정 **

- [ ] **영상 저장 범위 설정**

- [ ] **영상 저장**





# 4. Codes



 우선 필요한 Libraries 를 Import 한다.

```python
import cv2
import numpy as np
```



이후 영상을 띄울 Window(창)을 세팅하며 동시에 **ROI (Region of Interest)**를 지정해 어느 오브젝트를 추적할 지 지정해 줘야 한다.

OpenCV에서는 ROI를 지정할 수 있는 함수를 기본적으로 제공한다.

해당 실습에선 영상의 첫번째 프레임에 ROI를 설정한다.



```python
import cv2
import numpy as np

video_path = '00s.mp4'
cap = cv2.VideoCapture(video_path)

# 못 불러오고 오류가 나면 종료
if not cap.isOpened():
    exit()

# ROI Setting
play, img = cap.read() # 첫번째 프레임을 읽어온다. 이미지 형식

cv2.namedWindow('Select Window') # ROI 영역의 이름이 된다.
cv2.imshow('Select Window', img) # 첫번째 프레임의 이미지를 보여준다.

# 직사각형 모양의 ROI를 첫번째 이미지로 불러 온 프레임에 셋팅한다.
rect = cv2.selectROI('Select Window', img, fromCenter=False, showCrosshair=True)
cv2.destroyWindow('Select Window') # ROI 선택 후 윈도우를 닫는다.

# cap 변수의 비디오를 불러오면 재생, 못 불러오면 exit
while True:
    play, img = cap.read()

    if not play:
        exit()

    cv2.imshow('img', img)  # 윈도우에 이미지를 출력한다.
    if cv2.waitKey(1) == ord('q'): # 키 입력을 1 millisecond 간 기다린다. q를 누를경우
        break  # break으로 바로 빠져나간다.
        # 이것을 써주지 않으면 백그라운드에서 플레어가 돌다가 자동 종료된다.
```

![IMAGE 050](https://user-images.githubusercontent.com/52769104/103894607-86665480-5132-11eb-99cf-0f25a157f4f5.png)

ROI 가 지정된 모습. 지정 후 키보드의 Space bar를 눌러 빠져온다.

**이렇게 되면 ROI의 정보가 rect 변수에 저장된 상태이다.**

ROI는 높이, 길이, 넓이, 좌, 우 등의 설정값으로 위치를 설정 가능하다.



다음 단계로 Object Tracker를 설정하는 것이다. 프레임 단위로 이루어 진 영상은 첫 번째 프레임만 ROI와 Object Tracker를 지정해주면 이후 프레임은 자동으로 비슷한 이미지를 찾아나간다.



OpenCV의 Object Tracker의 종류(알고리즘)는 총 7가지가 있는데, 성능이 좋으면 속도가 느리고, 성능이 나쁘면 속도가 빠른 상관관계가 있다. 

아래 이미지 출처의 사이트에 들어가보면 각 Tracker 알고리즘 별 성능비교 Youtube 링크도 제공하고 있으니 비교를 위해서 접속해 보는 것을 추천한다.

![IMAGE 051](https://user-images.githubusercontent.com/52769104/103894631-8cf4cc00-5132-11eb-8372-4bb626f22db9.png)

**이미지 출처 : https://rosia.tistory.com/243**



코드는 다음과 같다.

```python
tracking = cv2.TrackerCSRT_create()  #CSRT 트래커를 초기화한다.

# 오브젝트 트래커가 img 윈도우 명의 rect를 따라가게 설정
tracking.init(img, rect)

# 영상을 한 프레임씩 읽어올 때마다 ROI가 설정된 프레임과 비교해서 비슷한 물체 위치를 찾아 반환 (Update)

while True:

	if not play: 이후에...
	
    good, roibox = tracking.update(img)

    # roibox에 설정된 좌, 높이, 넓이, 길이 값을 v 함수에 담고, 이 v함수는 int형으로 변환되는 for문
    left, top, w, h = [int(v) for v in roibox]
    # ROI설정된 직사각형 범위의 좌, 우 값을 프레임마다 넘겨줌. 박스색깔은 흰색, 그리고 굵기 설정.
    cv2.rectangle(img, pt1=(left, top), pt2=(left+w, top+h), color=(255,255,255), thickness=3)

```







완성된 영상을 저장해본다.

가장 중요한 부분은 저장될 영상의 크기이다. 
**이 부분에서 많은 오류가 발생하니 적절한 값을 구해준다.**

직사각형 박스의 중심을 구해 알맞게 저장해야 영상이 튀지 않는다.

```python
output_size = (187, 333)  #저장될 동영상의 해상도

# Video 설정
fourcc = cv2.VideoWriter_fourcc('m', 'p', '4', 'v')
# 저장형식과 frame per second는 불러온 동영상과 같이 저장
out = cv2.VideoWriter('%s_output.mp4' % (video_path.split('.')[0]), fourcc, cap.get(cv2.CAP_PROP_FPS), output_size)

...

center_x = left + w / 2
    center_y = top + h / 2

    result_top = int(center_y - output_size[1] /2)
    result_bottom = int(center_y + output_size[1] / 2)
    result_left = int(center_x - output_size[0] / 2)
    result_right = int(center_x + output_size[0] / 2)

    # .copy() 를 추가해 줌 으로써 저장될 영상에 하얀 사각형이 나오지 않는다.
    result_img = img[result_top:result_bottom, result_left:result_right].copy()

...

    # 새로운 저장 창의 이름이 result_img
    cv2.imshow('result_img', result_img)

    #저장 ~!
    out.write(result_img)
```



최종 완성된 결과물과 사용한 코드 전체이다.



```python
import cv2
import numpy as np

video_path = '100s.mp4'
cap = cv2.VideoCapture(video_path)

output_size = (187, 333)  #저장될 동영상의 해상도

# Video 설정
fourcc = cv2.VideoWriter_fourcc('m', 'p', '4', 'v')
# 저장형식과 frame per second는 불러온 동영상과 같이 저장
out = cv2.VideoWriter('%s_output.mp4' % (video_path.split('.')[0]), fourcc, cap.get(cv2.CAP_PROP_FPS), output_size)


# 못 불러오고 오류가 나면 종료
if not cap.isOpened():
    exit()

tracking = cv2.TrackerCSRT_create()  #CSRT 트래커를 초기화한다. 가장 BEST 효율


# ROI Setting
play, img = cap.read() # 첫번째 프레임을 읽어온다. 이미지 형식

cv2.namedWindow('Select Window') # ROI 영역의 이름이 된다.
cv2.imshow('Select Window', img) # 첫번째 프레임의 이미지를 보여준다.

# 직사각형 모양의 ROI를 첫번째 이미지로 불러 온 프레임에 셋팅한다.
rect = cv2.selectROI('Select Window', img, fromCenter=False, showCrosshair=True)
cv2.destroyWindow('Select Window') # ROI 선택 후 윈도우를 닫는다.

# 오브젝트 트래커가 img 윈도우 명의 rect를 따라가게 설정
tracking.init(img, rect)

# cap 변수의 비디오를 불러오면 재생, 못 불러오면 exit
while True:
    play, img = cap.read()

    if not play:
        exit()

    # 영상을 한 프레임씩 읽어올 때마다 ROI가 설정된 프레임과 비교해서 비슷한 물체 위치를 찾아 반환 (Update)
    good, roibox = tracking.update(img)

    # roibox에 설정된 좌, 높이, 넓이, 길이 값을 v 함수에 담고, 이 v함수는 int형으로 변환되는 for문
    left, top, w, h = [int(v) for v in roibox]

    center_x = left + w / 2
    center_y = top + h / 2

    result_top = int(center_y - output_size[1] /2)
    result_bottom = int(center_y + output_size[1] / 2)
    result_left = int(center_x - output_size[0] / 2)
    result_right = int(center_x + output_size[0] / 2)

    # .copy() 를 추가해 줌 으로써 저장될 영상에 하얀 사각형이 나오지 않는다.
    result_img = img[result_top:result_bottom, result_left:result_right].copy()

    # ROI설정된 직사각형 범위의 좌, 우 값을 프레임마다 넘겨줌. 박스색깔은 흰색, 그리고 굵기 설정.
    cv2.rectangle(img, pt1=(left, top), pt2=(left+w, top+h), color=(255,255,255), thickness=3)

    # 새로운 저장 창의 이름이 result_img
    cv2.imshow('result_img', result_img)

    #저장 ~!
    out.write(result_img)

    cv2.imshow('img', img)  # img 윈도우에 이미지를 출력한다.
    if cv2.waitKey(1) == ord('q'): # 키 입력을 1 millisecond 간 기다린다. q를 누를경우
        break  # break으로 바로 빠져나간다.
        # 이것을 써주지 않으면 백그라운드에서 플레어가 돌다가 자동 종료된다.
```



![result10](https://user-images.githubusercontent.com/52769104/103895600-ff19e080-5133-11eb-90d3-bddba2dd032a.gif)



영상처리에서 가장 많이 사용하는 Open Source Library인 **'OpenCV'**에 대한 설명이었다.

**다음번에는 이를 응용해 Deep Learning과 결합하여, 숨겨진 물체까지 Detection 하는 영상처리 프로그램을 구현하는 것으로 다뤄 볼 것이다.**

