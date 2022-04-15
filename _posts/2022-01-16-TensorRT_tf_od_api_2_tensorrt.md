---
title: "Tensorflow OD API saved_model -> TensorRT"
excerpt: "saved_model을 TensorRT로 변환"

categories:
  - TensorRT
tags:
  - [TensorRT, tensorflow object detection api]

toc: true
toc_sticky: true

date: 2022-01-16T22:30:00+09:00
last_modified_at: 2022-04-15T20:12:00+09:00
---

# TF OD API saved-model to TensorRT
<br>
Jetson Xavier NX에서 saved-model을 사용할때 메모리의 대부분을 차지하고 모델 로드 시간도 매우 길어 코드 테스트시에 시간낭비가 매우 컸다.

* 개선 시도
  * TF-TRT: inference 속도는 빨라졌으나 모델 로드 시간과 메모리 사용량은 크게 차이 없음.
  * TensorRT: inference 속도, 메모리 사용량, 모델 로드 시간이 크게 개선되어 원활한 테스트가 가능했다.
* 비교 - EfficientDet-D0 (tf2.4 / TensorRT 7.1.3)

  |unit: sec|saved_model|TF-TRT|TensorRT|
  |---------|-----------|------|--------|
  |loading  |350        |350   |7.5     |
  |inference|0.11       |0.06  |0.04    |

  * inference: 이미지 1장 처리 시간

## 변환 과정

* 기준
  * 모델: TF OD API의 EfficientDet-D0 saved-model
  * 환경:
    * tf 2.4
    * TensorRT 7.1.3
* 과정: saved_model ➡ onnx ➡ TensorRT

<a href="https://stackoverflow.com/questions/66087844/jetson-nx-optimize-tensorflow-model-using-tensorrt" class="btn btn--inverse">출처: stackoverflow</a>



## saved_model ➡ onnx

tf2onnx API를 이용하면 saved_model을 간단하게 onnx로 변환할 수 있다. 

<a href="https://github.com/onnx/tensorflow-onnx" class="btn btn--info">repository: tensorflow-onnx</a>

```bash
python -m tf2onnx.convert --saved-model model/effdet_512x512/saved_model/ --output effdet_origin.onnx --opset 11
```

opset은 onnx 오퍼레이터 셋 버전으로 버전에 따라 지원하는 연산에 차이가 있다. 11, 13에서 잘 작동한다. 변환 후 onnxruntime을 통해 onnx 모델을 확인한다.


## onnx 수정

바로 onnx를 TensorRT로 변환하면 몇 가지 에러가 발생한다. 주로 TensorRT에서 지원하지않는 구조, 노드때문으로 onnx-graphsurgeon 툴을 이용해 onnx를 수정해야한다.

### 참고자료
* onnx-graphsurgeon: onnx 수정 툴
  * <a href="https://github.com/NVIDIA/TensorRT/tree/master/tools/onnx-graphsurgeon" class="btn btn--info">repository: onnx-graphsurgeon</a>
* onnx 오퍼레이터 정보: 연산을 담당하는 노드들
  * <a href="https://github.com/onnx/onnx/blob/master/docs/Operators.md" class="btn btn--info">link: onnx operators</a>
* netron: 모델의 구조 시각화 툴. 어디를 어떻게 고칠지 파악하기 위해 필요
  * <a href="https://github.com/lutzroeder/netron" class="btn btn--info">repository: netron</a>

### 에러

**Unsupported ONNX data type: UINT8**

tf od api의 모델의 입력 타입 디폴트는 uint8이므로 float32로 변경해야한다.

**[TensorRT] ERROR: INVALID_ARGUMENT: getPluginCreator could not find plugin NonMaxSuppression version 1**

<a href="https://github.com/NVIDIA/TensorRT/tree/release/7.2/plugin/batchedNMSPlugin" class="btn btn--info">batchedNMSPlugin</a>

onnx의 nms 노드를 TensorRT의 nms 플러그인으로 교체해야한다. (참고: [링크](https://github.com/NVIDIA/TensorRT/issues/795))

tf od api의 nms와 tensorRT의 nms는 같은 방식으로 작동한다.
1. 박스 디텍션과 개별 클래스의 스코어로 nms를 수행한다. (클래스의 수만큼 nms가 수행된다.) 
2. 모든 nms 결과를 모아 스코어 순으로 정렬한 후 n개의 디텍션을 취한다.

netron으로 onnx를 시각화하면 다음와 같이 클래스의 수만큼 nms 노드가 존재하는 것을 볼 수 있다.

![onnx nms](/assets/images/post/220116/TensorRT_tf_od_api_2_tensorrt/onnx_nms.png){: .align-center}
    
tensorRT의 nms 노드(BatchedNMS_TRT)는 단일 노드에서 위의 과정을 모두 처리하므로 onnx 출력 중 raw_detection_boxes와 raw_detection_scores의 shape를 수정한 후 BatchedNMS_TRT로 연결해준다. (tf od api nms: [링크](https://github.com/tensorflow/models/blob/2de518be2d6a6e3670b223a4582b1353538d3489/research/object_detection/core/post_processing.py#L1070))

![tensorrt nms](/assets/images/post/220116/TensorRT_tf_od_api_2_tensorrt/tensorRT_nms.png){: .align-center}

**[TensorRT] ERROR: Network has dynamic or shape inputs, but no optimization profile has been defined.**

dynamic shape를 사용하려면 별도 설정이 필요한지 에러가 발생한다. 이 부분에 대해서는 찾지 못해서 batch size를 1로 지정했다.

**[TensorRT] ERROR: ../builder/myelin/codeGenerator.cpp (114) - Myelin Error in addNodeToMyelinGraph: 0 (map/while/TensorArrayV2Read/TensorListGetItem operation not supported within a loop body.)**

loop 내에 Gather 노드가 허용되지 않는다고 한다. tf od api에서 구축한 전처리 과정 그래프에서 문제가 발생하여 전처리 노드를 모두 날려버리고 graphsurgeon으로 만들어 붙였다.(efficientDet 전처리: [링크](https://github.com/tensorflow/models/blob/2de518be2d6a6e3670b223a4582b1353538d3489/research/object_detection/models/ssd_efficientnet_bifpn_feature_extractor.py#L190))

![effdet preprocessing](/assets/images/post/220116/TensorRT_tf_od_api_2_tensorrt/effdet_preprocessing.png){: .align-center}

전처리 과정에 Resize node - transformationMode / tensorRT에서 지원하지 않는 리사이즈 메소드를 사용한다는 에러가 있었는데 전처리 과정을 날려버리면서 같이 사라졌다.


```python
import onnx
import onnx_graphsurgeon as gs
import numpy as np


graph = gs.import_onnx(onnx.load('effdet_origin.onnx'))
nodes = graph.nodes
tensors = graph.tensors()

# set input_tensor shape & dtype
input_tensor = tensors['input_tensor']
input_tensor.dtype = np.float32
input_tensor.shape = [1, 512, 512, 3]

# # resize mode
# # 전처리 Loop 노드 내부에 서브 그래프가 존재함. - node.attrs['body']로 접근 
# preprocessing_node = nodes[2]
# resize_node = [node for node in preprocessing_node.attrs['body'].nodes if node.op == 'Resize'][0]
# resize_node.attrs['coordinate_transformation_mode'] = 'half_pixel'

# replace preprocessing node
# efficientNet 전처리 과정 구현
scale = gs.Constant(name='scale', values=np.array([1./255.], np.float32).reshape(1,))
input_scaled = gs.Variable(name='input_scaled', dtype=np.float32)
node_scale = gs.Node(op='Mul', inputs=[input_tensor, scale], outputs=[input_scaled])
nodes.append(node_scale)

ch_offset = gs.Constant(name='ch_offset', values=np.array([0.485, 0.456, 0.406], np.float32).reshape(1, 1, 3))
input_ch_shifted = gs.Variable(name='input_ch_shifted', dtype=np.float32)
node_ch_shift = gs.Node(op='Sub', inputs=[input_scaled, ch_offset], outputs=[input_ch_shifted])
nodes.append(node_ch_shift)

ch_scale = gs.Constant(name='ch_scale', values=(1./np.array([0.229, 0.224, 0.225], np.float32)).reshape(1, 1, 3))
input_ch_scaled = gs.Variable(name='input_ch_scaled', dtype=np.float32)
node_ch_scale = gs.Node(op='Mul', inputs=[input_ch_shifted, ch_scale], outputs=[input_ch_scaled])
nodes.append(node_ch_scale)

# onnx의 Conv 노드의 입력은 NCHW 포맷이므로 이미지를 transpose한다.
input_transposed = gs.Variable(name='input_transposed', dtype=np.float32)
node_transpose = gs.Node(
  op='Transpose',
  attrs={'perm': [0, 3, 1, 2]},
  inputs=[input_ch_scaled],
  outputs=[input_transposed],
)
nodes.append(node_transpose)

# Conv 노드의 입력 중 Loop 노드로부터의 입력을 새로운 전처리 노드의 출력으로 대체한다.
conv_node = [n for n in nodes if n.name == 'StatefulPartitionedCall/EfficientDet-D0/model/stem_conv2d/Conv2D'][0]
conv_node.i(0).outputs.clear()
conv_node.inputs[0] = input_transposed

# raw_detection_boxes에 차원 추가
raw_detection_boxes = tensors['raw_detection_boxes']
raw_detection_scores = tensors['raw_detection_scores']

raw_detection_boxes_unsqueezed = gs.Variable('raw_detection_boxes_unsqueezed', dtype=np.float32)
unsqueeze_node = gs.Node(
  op='Unsqueeze',
  name='unsqueeze_raw_detection_boxes',
  attrs={
      'axes': [2]
  },
  inputs=[raw_detection_boxes],
  outputs=[raw_detection_boxes_unsqueezed],
)
graph.nodes.append(unsqueeze_node)

# nms 노드 추가
num_detections = gs.Variable('num_detections', dtype=np.int32, shape=(1, 1))
nmsed_boxes = gs.Variable('nmsed_boxes', dtype=np.float32, shape=(1, 100, 4))
nmsed_scores = gs.Variable('nmsed_scores', dtype=np.float32, shape=(1, 100))
nmsed_classes = gs.Variable('nmsed_classes', dtype=np.float32, shape=(1, 100))

nms_node = gs.Node(
  op='BatchedNMS_TRT',
  name='nms',
  attrs={
      "shareLocation": True, # 같은 박스로 모든 클래스에 대해 nms를 수행
      "numClasses": 6,
      "backgroundLabelId": -1, # 백그라운드 인덱스. 없는 경우 -1로 설정
      "topK": 4096,  # 스코어 순으로 박스를 정렬하여 상위 4096개만 연산
      "keepTopK": 100,  # nms 결과 중 스코어순으로 100개만 취함
      "scoreThreshold": 1e-8,
      "iouThreshold": 0.5,
      "isNormalized": True,  # 박스가 0~1 범위인 경우 True, 픽셀값이면 False
      "clipBoxes": True,  # 박스를 0~1 범위로 clip
      "scoreBits": 10,  # 스코어 비트 수. 높으면 nms 성능이 높은 대신 느려진다.
  },
  inputs=[raw_detection_boxes_unsqueezed, raw_detection_scores],
  outputs=[num_detections, nmsed_boxes, nmsed_scores, nmsed_classes],
)
graph.nodes.append(nms_node)

# 그래프의 아웃풋을 새로 정의
graph.outputs = [num_detections, nmsed_boxes, nmsed_scores, nmsed_classes]
# clearup: 아웃풋에 관여하지 않는 노드를 제거한다.
# toposort: 그래프의 노드들을 순서에 맞게 자동 정렬한다.
graph.cleanup().toposort()
onnx.save_model(gs.export_onnx(graph), 'effdet_modify.onnx')
```

## TensorRT 변환

nvidia에서 tensorRT를 받아 sample/python에 포함된 코드들을 참고해도 좋지만 잘 정리해주신 글이 있으니 참고하여 코드 작성. (참고: [SIA](https://blog.si-analytics.ai/33))


```python
import tensorrt as trt


# TRT 7.x
print('convert onnx to trt')
TRT_LOGGER = trt.Logger(trt.Logger.WARNING)
trt.init_libnvinfer_plugins(TRT_LOGGER, '')

EXPLICIT_BATCH = 1 << int(trt.NetworkDefinitionCreationFlag.EXPLICIT_BATCH)
with trt.Builder(TRT_LOGGER) as builder, \
        builder.create_network(EXPLICIT_BATCH) as network, \
        trt.OnnxParser(network, TRT_LOGGER) as parser:

    builder.max_workspace_size = (1 << 30)
    builder.fp16_mode = True

    with open('./effdet_origin.onnx', 'rb') as model:
        if not parser.parse(model.read()):
            for error in range(parser.num_errors):
                print (parser.get_error(error))

    engine = builder.build_cuda_engine(network)
    buf = engine.serialize()
    with open('./effdet_origin.trt', 'wb') as f:
        f.write(buf)
```