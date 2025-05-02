---
layout: post
category: [ctf]
title:  "Defcon 2025 jxl4fun 풀이"
date:   2025-05-01
author: SeHwa
og_image: assets/img/posts/15/cover.jpg
---

4/12 부터 이틀동안 했던 Defcon 2025 CTF 에서 풀었던 jxl4fun 이라는 문제 풀이를 정리해두기로 했다. 길게 쓰기엔 시간도 아깝고 번거로우므로 나중에 개인적인 참고용으로만 짧게 적어둔다. 문제를 풀긴 했으나 CTF 가 끝나고 공식 풀이를 보니 아래의 방식은 언인텐이었다. 인텐 풀이는 아래의 공식 GitHub 에 있다.

<br>

jxl4fun 문제는 아래에 있다.

<div class="sx-button">
  <a href="https://github.com/Nautilus-Institute/quals-2025/tree/main/jxl4fun" class="sx-button__content github">
    <img src="/assets/img/icons/github.svg"/>
    <p>Nautilus-Institute/quals-2025</p>
  </a>
</div>

<br>

문제 파일과 함께 [JPEG XL](https://en.wikipedia.org/wiki/JPEG_XL) 포맷의 샘플 이미지 파일 `jxl_art_by___wb__.jxl` 가 하나 주어진다. ImageMagick 으로 png 로 변환해서 보든 그냥 뷰어로 보든 어쩄든 열어서 보면 아래와 같은 1024x1024 크기의 프랙탈 이미지가 보인다.

<div markdown=1 class="sx-center">
<div class="sx-picture">
<a href="/assets/img/posts/15/1.jpg" data-lity>
  <img src="/assets/img/posts/15/1.jpg" style="width:500px" />
</a>
</div>
</div>

그런데 jxl 파일은 저 픽셀 이미지를 아무리 압축해도 절대로 도달할 수 없을 만큼 극히 짧은 데이터가 전부이다.(벡터 포맷이라고 가정해도 무리인 수준)

<div markdown=1 class="sx-center">
<div class="sx-picture">
<a href="/assets/img/posts/15/2.jpg" data-lity>
  <img src="/assets/img/posts/15/2.jpg" style="width:500px" />
</a>
</div>
</div>

즉 무언가 연산을 통해 저런 이미지를 그려내는 게 JPEG XL 포맷에 있을 것이라 생각할 수 있다. 저 샘플 이미지 파일명을 보고 구글에 jxl art 라고 검색해보니 [이 GitHub](https://github.com/surma/jxl-art) 를 찾을 수 있었는데, 여기에 링크된 [이 사이트](https://jxl-art.surma.technology/)를 들어가보면 아래와 같은 코드가 보인다.

```
Bitdepth 8
Orientation 7
RCT 6

if y > 150
  if c > 0
    - N 0
    if x > 500
      if WGH > 5
        - AvgN+NW + 2
        - AvgN+NE - 2
      if x > 470
        - AvgW+NW -2
        if WGH > 0
          - AvgN+NW +1
          - AvgN+NE - 1
  if y > 136
    if c > 0
      if c > 1
        if x > 500
          - Set -20
          - Set 40
        if x > 501
          - W - 1
          - Set 150
      if x > 500
        - N + 5
        - N - 15
    if W > -50
      - Weighted -1
      - Set 320
```

이 샘플 코드를 저 사이트에서 실행하면 이미지를 렌더링해서 표시한다. 사이트에서 Help 를 누르면 [이 페이지](https://jxl-art.surma.technology/wtf)로 연결되는데, 원리에 대한 기술적인 설명이 나와있고 위의 코드가 `Prediction Tree` 라고 불린다는 것을 알 수 있다. 이 페이지에 설명이 자세하게 나와있으므로 추후 문제를 풀기 위한 Prediction Tree 관련 지식은 이 페이지의 내용만으로도 충분하다. 그리고 상관은 없지만 나중에 찾아보니 예상대로 [Prediction Tree 는 튜링 완전](https://dbohdan.com/jpeg-xl)이라고 한다.

<br>

주어진 문제 파일은 공식 레퍼런스인 [libjxl](https://github.com/libjxl/libjxl) 을 수정하고 빌드한 릴리즈로, 라이브러리 파일들이 있고 실행 바이너리는 jxl 파일을 jpg, png 등의 이미지 파일로 변환하는 `djxl` 만 주어진다. 빌드한 커밋도 주어지므로 따로 원본을 빌드해서 주어진 파일들과 디핑할 수 있는데, 팀원들이 디핑한 결과 대략 아래와 같은 차이가 있었다.

<br>

### 1.  **`[djxl]`** flag 를 읽어서 할당한 힙에 쓴 다음 해제

<div markdown=1 class="sx-center">
<div class="sx-picture">
<a href="/assets/img/posts/15/3.jpg" data-lity>
  <img src="/assets/img/posts/15/3.jpg" style="width:400px" />
</a>
</div>
</div>

main 함수에서 `/flag_intro` 파일을 읽어서 malloc 으로 할당한 메모리에 쓰고 해당 메모리를 다시 free 하는 코드가 추가되었다.

<br>

### 2.  **`[libjxl.so.0.12]`** 새로운 Predictor 일부 추가

<div markdown=1 class="sx-center">
<div class="sx-picture">
<a href="/assets/img/posts/15/4.jpg" data-lity>
  <img src="/assets/img/posts/15/4.jpg" style="width:600px" />
</a>
</div>
</div>

[원본 코드](https://github.com/libjxl/libjxl/blob/9845054a9d104062bb8712f3a04996a415e63774/lib/jxl/modular/encoding/context_predict.h#L442)의 PredictOne 함수는 위와 같이 되어있고, 위의 상수들은 아래와 같이 정의되어 있다.

<div markdown=1 class="sx-center">
<div class="sx-picture">
<a href="/assets/img/posts/15/5.jpg" data-lity>
  <img src="/assets/img/posts/15/5.jpg" style="width:350px" />
</a>
</div>
</div>

그런데 주어진 libjxl.so.0.12 파일에서는 아래와 같이 새로운 case 들이 추가되어있다. (`jxl::PredictNoTreeWP` 와 `jxl::PredictNoWP` 함수)

<div markdown=1 class="sx-center">
<div class="sx-picture">
<a href="/assets/img/posts/15/6.jpg" data-lity>
  <img src="/assets/img/posts/15/6.jpg" style="width:400px" />
</a>
</div>
</div>

<div markdown=1 class="sx-center">
<div class="sx-picture">
<a href="/assets/img/posts/15/7.jpg" data-lity>
  <img src="/assets/img/posts/15/7.jpg" style="width:400px" />
</a>
</div>
</div>

g_pallet 도 원본 코드에선 존재하지 않고, 이 함수들의 초기 호출시에 malloc 으로 할당하는 메모리 주소를 저장한다. 코드를 보면 새로 추가된 15번 Predictor 의 경우 심플하게 g_pallet 메모리의 특정 index 에서 2bytes 를 읽어서 저장하는데, 만약 이 값을 우리가 읽을 수 있고 index 에 별도의 검증이 없다면 OOB read 가 가능할 것임을 짐작할 수 있다. 16번의 경우 반대로 OOB write 의 가능성을 짐작할 수 있지만 이 문제에선 필요하지 않으므로 생략한다.

<br>

Predictor 는 위에서 링크한 [이 사이트](https://jxl-art.surma.technology/wtf)에 잘 설명되어 있다.

<div markdown=1 class="sx-center">
<div class="sx-picture">
<a href="/assets/img/posts/15/8.jpg" data-lity>
  <img src="/assets/img/posts/15/8.jpg" style="width:800px" />
</a>
</div>
</div>

이제 추가된 Predictor 를 테스트하기 위해, Prediction Tree 를 jxl 파일로 만들어주는 프로그램을 찾아서 새로운 Predictor 를 소스 코드에 추가해서 빌드를 해야한다. 검색해보면 `jxl_from_tree` 라는 툴이 있는데, 이는 공식 libjxl 에 포함되어 있으므로([#](https://github.com/libjxl/libjxl/blob/9845054a9d104062bb8712f3a04996a415e63774/tools/jxl_from_tree.cc)) libjxl 을 다시 빌드하면 된다.

우선 위에서 본 Predictor 리스트가 정의된 [이 부분](https://github.com/libjxl/libjxl/blob/9845054a9d104062bb8712f3a04996a415e63774/lib/jxl/modular/options.h#L21)에 새로운 Predictor 를 15, 16번으로 추가하고 Best/Variable 도 그에 맞춰서 조정하고 아래의 갯수 계산 부분까지 수정해주고, jxl_from_tree 에서 Predictor 를 파싱할 때 사용되는 [이 부분](https://github.com/libjxl/libjxl/blob/9845054a9d104062bb8712f3a04996a415e63774/tools/jxl_from_tree.cc#L120)에도 새로 추가할 Predictor 를 넣고 아무 적당한 이름으로 지정해주면 된다.

빌드하면 이제 Prediction Tree 코드를 jxl 파일로 만들 수 있고, 추가한 새로운 Predictor 도 사용할 수 있다. 가령 아래와 같은 코드를 만들어볼 수 있다.

```
Bitdepth 16

if y > 1
  - Set 0
  if y > 0
    if x > 0
      - Set 0
      - MyPredictor15 +0
    if x > 0
      - Set 0
      - Set 7777
```

Prediction Tree 코드 설명을 읽어보면, 조금 제한적이고 번거롭기는 하지만 조건문을 여러 개 사용해서 원하는 좌표의 픽셀만 정확하게 선택해서 해당 좌표에 특정 Predictor 를 설정하는 것이 가능하다. Set 을 이용하면 해당 픽셀에 특정 컬러값을 설정할 수도 있다. 컬러 채널도 조건문으로 설정할 수는 있지만 굳이 필요하진 않다.

추가된 15번 Predictor 에서 index 값이 어떤 값으로 설정되는지를 확인해보면, 해당 좌표의 바로 위 픽셀의 값을 가져오는 것을 확인할 수 있다. 디버깅을 하면서 실제로 index 값이 어떻게 설정되는지 확인하면 확실하게 알 수 있다. 이런 부분들은 소스 분석보다는 Prediction Tree 를 계속 만들어보면서 파악했다. 각 픽셀당 Prediction Tree 가 실행되는 순서는 (0, 0) 좌표부터 각 가로줄을 왼쪽부터 오른쪽으로 순서대로 실행하는 것으로 추정했다.

실제로 g_pallet 메모리의 특정 index 에서 2bytes 를 읽은 값은 해당 15번 Predictor 가 설정된 좌표의 픽셀 값으로 설정된다는 것도 확인할 수 있다. 그러면 이제 대략 아래와 같은 구상을 할 수 있다.

<div markdown=1 class="sx-center">
<div class="sx-picture">
<a href="/assets/img/posts/15/9.jpg" data-lity>
  <img src="/assets/img/posts/15/9.jpg" style="width:800px" />
</a>
</div>
</div>

(0, 0) 좌표부터 가로 2줄을 위의 이미지처럼 구성해보았다. 이렇게 하면 (g_pallet + 1) 부터 (g_pallet + 512) 까지의 값들이 2번째 줄의 Predictor 15 위치들에 픽셀값으로 각각 설정되어 돌아올 것이라 예상했다. 다른 Predict 영향을 방지하기 위해 한 칸씩 띄워주었다.

실제로 이렇게 해보니 첫 번째는 잘 되었지만 2번째부터는 index 설정이 제대로 되지 않았는데, 디버깅해본 결과 2번째부터는 Predictor 15 픽셀의 대각선 11시 방향 픽셀의 값을 index 로 사용하는 것을 볼 수 있었다. 원인을 분석하기엔 귀찮아서 굳이 분석하지 않았다. 그래서 최종적으로 아래와 같이 설계했다.

<div markdown=1 class="sx-center">
<div class="sx-picture">
<a href="/assets/img/posts/15/10.jpg" data-lity>
  <img src="/assets/img/posts/15/10.jpg" style="width:680px" />
</a>
</div>
</div>

위와 같은 Prediction Tree 코드를 생성하기 위한 Python 코드를 구현했다.

```python
import sys
sys.setrecursionlimit(100000)

f = open("tree.txt", "w")
f.write("Bitdepth 16\n")

def if_r(depth):
    if depth > 1020:
        f.write("- Set 0\n")
        return

    f.write("if x > " + str(depth-1) + "\n")
    if_r(depth+1)
    if depth % 2 == 0:
        f.write("if y > -1\nif y > 0\nif y > 1\n- Set 0\n- Mrbin2 +0\n- Set 0\n- Set 0\n")
    else:
        f.write("- Set " + str((int(sys.argv[1])) - (depth // 2)) + "\n")

if_r(0)
f.close()
```

목표는 flag_intro 데이터가 할당됐던 메모리 위치를 찾아서 값을 가져오는 것이다. 사실 위 코드는 y=0,1 로 두 줄만 사용하고 있지만, 1024x1024 의 공간이 있으므로 픽셀을 세로로 더 사용하면 한 번에 가져올 수 있는 메모리 데이터량은 충분하다. index 를 조절하면서 계속 탐색해도 되지만, 팀원이 g_pallet 과 flag_intro 가 할당되는 메모리 주소가 출력된다는 것을 이용해서 상대적인 거리를 알 수 있다고 하여 그 방식으로 빠르게 찾기로 했다.

이 거리는 Prediction Tree 코드 사이즈에 따라 더 늘어나지만 위처럼 index 만 바꿔주는 경우는 거의 변하지 않으므로 일단 만들어 놓고 거리를 계산해서 index 를 조정하면 된다.

그리고 서버에 날려서 받아오는 png 이미지에서 픽셀값은 아래처럼 추출할 수 있다.

```python
import cv2
import struct

img = cv2.imread("result.png", cv2.IMREAD_UNCHANGED)
data = b""
for i in range(1, 1022, 2):
    data += struct.pack(">H", img[1, i][0])

open("flag", "wb").write(data)
```

미리 예측한 index 값에서 500 씩 조정하면서 서버에 날려서 메모리 데이터에 flag 가 포함되어 있는지 눈으로 찾아본 결과, 예측한 오프셋에서 그리 멀지않은 -249500 오프셋에서 flag 를 찾을 수 있었다.

<div markdown=1 class="sx-center">
<div class="sx-picture">
<a href="/assets/img/posts/15/11.jpg" data-lity>
  <img src="/assets/img/posts/15/11.jpg" style="width:700px" />
</a>
</div>
</div>

```
flag{WellingtonMint6248n25:l0FhNUOB0UvPy71ByDx6p2y1UfKLFBwVfEyJyQyh1WMmbkHRJVPXQtdcC7_yV6zFi5t7RHtawF1wYhvAP0dLyQ}
```