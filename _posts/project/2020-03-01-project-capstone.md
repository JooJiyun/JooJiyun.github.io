---
title: "2019 Capstone"
excerpt: "2019 capstone project"

categories:
  - Project
tags:
  - OCR
  - NAS
last_modified_at: 2020-03-01T16:09:00
---

<h1>STRNAS: </h1><h3>OCR을 위한 NAS 모델 구축</h3>
<br>

<h4>서론</h4>
* OCR: Optical Character Recognition, 간단하게 사진등에 있는 글자를 읽는 작업을 말한다.<br>
크게 글자가 있는 부분을 확인하는 localization과 글자를 읽는 recognition을 하는 부분으로 나눌 수 있다.

![full_pipeline](https://joojiyun.github.io/assets/post_project/ocr_pipe.png){: .align-center}  
<br>
* 우리는 recognizer부분에 해당하는 모델을 찾는 NAS 모델 구축을 목표로 하였다.<br>
대체적으로 recognizer 모델들은 아래와 같은 pipeline을 가졌기 때문에 각각의 부분에 대하여 search를 먼저 실험하도록 하였다.

![flow](https://joojiyun.github.io/assets/post_project/ocr_part.png){: .align-center}    

* NAS: Neural Architecture Search, 간단하게 딥러닝 모델을 찾는 모델을 말한다.<br>
대체적으로 각각의 layer에서 어떤 연산을 하는 것이 더 좋은 결과를 내는가를 반영하여 모델을 찾아간다.
![nas](https://joojiyun.github.io/assets/post_project/nas.png){: .align-center}   

<br>
* 또한 기존의 NAS 모델들은 CNN, RNN등에만 초점을 맞췄었기 때문에 OCR같은 복잡한 모델에 적용시켜보는 것은 의미가 있을 것이라 생각하였다.
<br><br><br><br><br>

<h4>과정</h4>
실험들은 OCR, NAS 모델의 baseline을 아래의 모델들로 정한뒤 진행하였다.
<br>
 - OCR baseline: ASTER, MORAN -> clova AI
 <h5>
 	ASTER: An Attentional Scene Text Recognizer with Flexible Rectification (1 Sept 2019)<br>
 	MORAN: A Multi-Object Rectified Attention Network for Scene Text Recognition (10 Jan 2019)<br>
 	Clova: What Is Wrong With Scene Text Recognition Model Comparisons? Dataset and Model Analysis (3 Apr 2019)
 </h5>
 프로젝트 진행 당시 ASTER와 MORAN이 SOTA 모델이었기때문에 baseline으로 삼았었으나 pytorch로 완벽하게 성능을 구현해내지 못했고 Clova benchmark 모델은 기존의 여러 OCR 모델들을 recifier/extractor/rnn-based encode&decoder 부분으로 나누어 같은 환경에서 맞춰 실험했기 때문에 각각의 부분에 search를 하기 용이해 보여 바꾸게 되었다.
 - NAS baseline: ProxylessNAS
 <h5>
 	ProxylessNAS: Direct Neural Architecture Search on Target Task and Hardware (28 Sep 2018)
 </h5>
 NAS 모델 특성상 많은 리소스와 시간이 필요했고 우리의 컴퓨터 사양으로 한학기 프로젝트로 가능할만 한 모델은 proxyless NAS가 거의 유일했다.
<br><br>

여러 OCR모델들이 rectifier의 부분에 집중을 하는 모습을 보였고 우리도 rectifier, CNN extractor, RNN-based encoder&decoder 순으로 search를 진행하기로 하였다.<br>
각각의 부분에 대하여 search가 가능하게 된다면 한번에 search가 가능하게 하고자하였다.<br>
그 외에도 latency(연산마다 필요한 시간)등을 search할 때 반영시켜 좀 더 효율적인 모델을 찾을 수 있도록 하였다.
<br><br><br><br><br>

<h4>결과</h4>
* OCR baseline구현:

![base_line](https://joojiyun.github.io/assets/post_project/base_line.png){: .align-center}  

 -- 글자의 종류개수, dataset에서 localization boundary등 세부설정들이 다르다는 것들 알고 고쳐봤지만 논문의 수치는 구현해내지 못했다.<br>
 -- 심지어 MORAN의 경우에는 공식 페이지의 원래 코드를 돌렸음에도 불구하고 논문만큼의 결과는 내지 못했고 공식 홈페이지에 의하면 논문의 수치가 나올확률은 1/3이라고 하였다.<br>
--> baseline을 바꾸기로 결정.<br>

* rectifier search: serach를 해서 별로 선능이 더 향상되지 않았다.
 -- 최근 논문에서도 recifier가 잘 작동하지 못하는 것을 보면 잘 작동하는 rectifier를 찾는 일은 어려운 task다.
 -- rectifier가 먼저 찾아지고 나서 뒷부분들이 정해져야하기 때문에 rectifier 찾기가 더 어려운 것이라 생각한다.

* CNN extractor search:
![benchmark](https://joojiyun.github.io/assets/post_project/benchmark.png){: .align-center}  
(NVIDIA TESLA P40 GPU)
![searched](https://joojiyun.github.io/assets/post_project/searched.png){: .align-center} 
(NVIDIA GEFORCE RTX 2080 Ti)
위의 표가 benchmark에서의 성능에 대한 지표이고, 아래 표가 CNN단의 search로 찾아낸 모델의 성능에 대한 지표이다.<br>
gpu의 성능이 부족함에도 불구하고 시간측면이나 parameter수가 적지만 정확도는 best모델에 비해 2%정도 떨어진다.<br>

* RNN based ecoder & edcoder: search를 하기에 무리가 있었다.<br>
rnn내부를 커스터마이징 하기에는 최적화를 할 수 없어 한번 트레이닝을 하는데도 많은 시간과 메모리가 필요하고 결국 할 수 없었다.

* 결국 CNN부분 밖에 search를 진행하지 못했고 search가 잘 작동했는지 알아보기 위한 ablation study를 진행해보았다.
<br>우리의 결과 외의 점, 선등은 benchmark의 결과들이다.
![result](https://joojiyun.github.io/assets/post_project/result.png){: .align-center}  
<br><br><br>
<!--<font size="2em" color="gray">tag의 OCR, NAS를 클릭하면 더 많은 내용을 볼 수 있습니다.</font>-->