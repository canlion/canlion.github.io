<!-- ---
title: "Tensorflow OD API saved_model -> TensorRT"
excerpt: "saved_model을 TensorRT로 변환"

categories:
  - TensorRT
tags:
  - [TensorRT, tensorflow object detection api]

toc: true
toc_sticky: true

date: 2022-01-16 22:30
last_modified_at: 2022-01-16 22:30
--- -->

# TF OD API saved_model -> TensorRT

Jetson Xavier NX에서 saved_model을 사용할 일이 있었는데 메모리의 거의 대부분을 차지하고 로딩시간도 매우 길어서 코드를 테스트할때마다 시간 낭비가 매우 컸다.

* **개선 시도: saved_model 변환**
  * TF-TRT 모델은 inference 속도는 빨라졌지만 로딩 시간과 메모리 사용량은 크게 차이가 없었다.
  * TensorRT 모델은 inference 속도, 메모리 사용량, 로딩 시간까지 크게 개선되어 원활하게 테스트를 할 수 있었다.

**모델 성능 비교** - EfficientDet-D0 (tf2.4 / TensorRT 7.1.3)

|unit: sec|saved_model|TF-TRT|TensorRT|
|---------|-----------|------|--------|
|loading  |350        |350   |7.5     |
|inference|0.11       |0.06  |0.04    |

* inference: 이미지 1장 처리 시간

## 모델 변환 과정

{% capture notice-env %}
* **기준**
  * 모델: TF OD API의 EfficientDet-D0
  * 환경:
    * tf 2.4
    * TensorRT 7.1.3
{% endcapture %}

<div class="notice--warning">{{ notice-env | markdownify }}</div>

**요약: saved_model ➡ onnx ➡ TensorRT**
{: .text-center}
{: .notice--info}

<a href="https://stackoverflow.com/questions/66087844/jetson-nx-optimize-tensorflow-model-using-tensorrt" class="btn btn--inverse">출처: stackoverflow</a>

<br>
**변환 과정**
1. saved_model ➡ onnx:
  * <a href="https://github.com/onnx/tensorflow-onnx" class="btn btn--info">repository: tensorflow-onnx</a>
2. onnx 수정:
  * TensorRT에서 허용하지 않는 구조, 노드 수정
3. 수정한 onnx ➡ TensorRT



## saved_model ➡ onnx

tf2onnx API를 이용하면 saved_model을 간단하게 onnx로 변환할 수 있다.

<a href="https://github.com/onnx/tensorflow-onnx" class="btn btn--info">repository: tensorflow-onnx</a>

```bash
python -m tf2onnx.convert --saved-model model/effdet_512x512/saved_model/ --output effdet_origin.onnx --opset 11
```

opset은 onnx 오퍼레이터 셋 버전으로 버전에 따라 지원하는 연산에 차이가 있다. 11, 13을 시도해보았고 잘 작동한다. 변환 후 onnxruntime을 통해 onnx 모델을 확인한다.


## onnx 수정

onnx를 TensorRT로 변환하면 몇가지 오류를 뿜으며 실패한다. 주로 TensorRT에서 지원하지않는 구조, 노드때문으로 onnx-graphsurgeon 툴을 이용해 onnx를 수정해야한다. 먼저 TensorRT로 변환시에 어떤 오류가 발생하는지 확인하려면 다음 챕터를 먼저 참고

* **onnx 수정 전 참고**:
  * onnx-graphsurgeon: onnx 수정 툴
    * <a href="https://github.com/NVIDIA/TensorRT/tree/master/tools/onnx-graphsurgeon" class="btn btn--info">repository: onnx-graphsurgeon</a>
  * onnx 오퍼레이터 정보: 연산을 담당하는 노드들
    * <a href="https://github.com/onnx/onnx/blob/master/docs/Operators.md" class="btn btn--info">link: onnx operators</a>
  * netron: 모델의 구조 시각화 툴. 어디를 어떻게 고칠지 파악하기 위해 필요
    * <a href="https://github.com/lutzroeder/netron" class="btn btn--info">repository: netron</a>

### TensorRT 변환시 발생하는 문제와 해결책
* **Unsupported ONNX data type: UINT8**
  * 보통 TF OD API를 통해 saved_model을 생성하면 입력 타입이 uint8으로 설정
  * float32로 입력 타입을 바꾼다.
* 전처리 과정 관련 - **아래 이유로 전처리 서브그래프를 제거하고 다시 짜서 이어붙인다.**
  * netron을 통해 onnx를 시각화하면 전처리 과정이 서브그래프로 분리되어있어 그래프와 어떻게 연결되있는지 모르겠다. 그리고 전처리 과정에서 나온 출력이 후처리까지 연결되있는데 이것도 어떤 쓰임새인지 모르겠다.
  * **[TensorRT] ERROR: ../builder/myelin/codeGenerator.cpp (114) - Myelin Error in addNodeToMyelinGraph: 0 (map/while/TensorArrayV2Read/TensorListGetItem operation not supported within a loop body.)**
    * 전처리 과정이 loop로 이루어지는데 내부에 gather노드가 포함되어있다
  * **Resize node - transformationMode** 관련
    * TensorRT에서 지원하지않는 resize method가 기본값으로 설정되었다.
* **[TensorRT] ERROR: INVALID_ARGUMENT: getPluginCreator could not find plugin NonMaxSuppression version 1**
  * onnx의 nms노드를 TensorRT에서 지원하지 않는다. <a href="https://github.com/NVIDIA/TensorRT/issues/795" class="btn btn--info">link: TensorRT issue</a>
  * onnx의 nms노드를 TensorRT의 nms노드로 교체한다. <a href="https://github.com/NVIDIA/TensorRT/tree/release/7.2/plugin/batchedNMSPlugin" class="btn btn--info">link: TensorRT plugins</a>
    * BatchedNMS_TRT
    * BatchedNMSDynamic_TRT
* **[TensorRT] ERROR: Network has dynamic or shape inputs, but no optimization profile has been defined.**
  * 입력으로 dynamic shape를 사용하려면 별도의 설정이 필요한데 모른다.
  * batch size 1로 설정

### NMS 노드 관련하여
