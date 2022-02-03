---
title: "데이터 시각화 2.1: bar plot"
excerpt: "bar"

categories:
  - boostcamp
tags:
  - [boostcamp]

toc: true
toc_sticky: true

date: 2022-02-03T18:33:00+09:00
last_modified_at: 2022-02-03T18:33:00+09:00
---

# bar plot

범주에 따른 값의 비교에 적합. (개별 비교, 그룹 비교 모두 적합)

* 막대 방향에 따라
  * 수직: 일반적
  * 수평: 범주가 많은 경우 적합<br>
    ![barh](/assets/images/post/220203/boostcamp-data-viz-2/barh.png)

    ```python
    labels = ['Game1', 'Game2', 'Game3']
    scores_0 = np.random.randint(0, 100, (3,))
    scores_1 = np.random.randint(0, 100, (3,))

    fig = plt.figure()
    plt.barh(labels, scores_0, color='royalblue')
    plt.title('Team 0')
    plt.xlabel('Score')
    plt.show()
    ```

## 종류

### Multiple Bar Plot

하나의 subplot에 하나의 그룹의 범주에 대한 값 표현.

![multiple bar plot](/assets/images/post/220203/boostcamp-data-viz-2/multiple_bar_plot.png){: .align-center}

```python
fig, axes = plt.subplots(1, 2, sharey=True)
# sharey: subplot간 y축 범위를 공유한다. get_ylim & set_ylim을 사용해도 좋다.

ax = axes[0]
ax.bar(labels, scores_0, color='royalblue')
ax.set_title('Team 0')
ax.set_xlabel('Score')

ax = axes[1]
ax.bar(labels, scores_1, color='tomato')
ax.set_title('Team 1')
ax.set_xlabel('Score')

plt.suptitle('Team Scores')
plt.show()
```

### Stacked Bar Plot

2개 이상의 그룹을 쌓아서 표현. 쌓는 순서는 꼭 유지하자. 가장 하단에 쌓인 그룹의 범주 값들은 파악하기 쉽지만 나머지 그룹들은 파악하기 어렵다.

![stacked bar plot](/assets/images/post/220203/boostcamp-data-viz-2/stacked_bar_plot.png){: .align-center}

`.bar()`에서는 `bottom` 파라미터, `.barh()`에서는 `left` 파라미터 이용.
```python
fig, axes = plt.subplots(1, 2)

ax = axes[0]
ax.bar(labels, scores_0, label='Team 0', color='royalblue')
ax.bar(labels, scores_1, label='Team 1', bottom=scores_0, color='tomato')

ax = axes[1]
ax.bar(labels, scores_1, label='Team 1', bottom=scores_0, color='tomato')
# bottom 부분을 비우고 그린다.

for ax in axes:
    ax.set_xlabel('Score')
    ax.set_title('Team scores')
    ax.legend()
    ax.margins(0.1, 0.2)
    ax.set_ylim(axes[0].get_ylim())

plt.show()
```


#### Percentage Stacked Bar Chart

![stacked percentage bar plot](/assets/images/post/220203/boostcamp-data-viz-2/stacked_percentage_bar_chart.png){: .align-center}

```python
fig, ax = plt.subplots(1, 1)
scores_sum = scores_0 + scores_1
scores_0_perc = scores_0 / scores_sum
ax.barh(labels, scores_0_perc, label='Team 0', color='royalblue')
ax.barh(labels, 1-scores_0_perc, label='Team 1', left=scores_0_perc, color='tomato')
ax.set_title('Score Percentage Stacked Bar Chart', weight='bold')
ax.set_ylabel('Games', weight='bold')
plt.show()
```

### Overlapped Bar Plot

2개 그룹만 비교하면 투명도를 이용하여 겹쳐 그린다. 3개 이상의 그룹은 파악하기 힘들다. (Area plot에 더 효과적인 방식)

![overlapped bar plot](/assets/images/post/220203/boostcamp-data-viz-2/overlapped_bar_plot.png){: .align-center}

```python
fig, ax = plt.subplots(1, 1)
ax.bar(labels, scores_0, label='Team 0', color='royalblue', alpha=.75)
ax.bar(labels, scores_1, label='Team 1', color='tomato', alpha=.75)
ax.set_xlabel('Score')
ax.set_title('Team scores')
ax.legend()
ax.margins(0.1, 0.1)
plt.show()
```

### Grouped Bar Plot

각 범주별로 그룹의 범주값 bar를 나란히 배치한다. 그룹이 너무 많다면 비중이 적은 그룹은 etc로 뭉뚱그려 표현하자.

![group bar plot](/assets/images/post/220203/boostcamp-data-viz-2/group_bar_plot.png){: .align-center}

```python
fig, ax = plt.subplots(1, 1)
n_groups = 2  # 그룹 수: 팀 수
n_labels = len(labels)  # feature 수: 게임 수
bar_width = .4
colors = ['royalblue', 'tomato']
for i, scores in enumerate([scores_0, scores_1]):
    # bar를 0, 1, 2, ... 에서 이격시켜서 배치한다. 그룹 수에 따라 달라짐.
    bar_pos = np.arange(n_labels) + (-bar_width * (n_groups-1-2*i)/2)
    ax.bar(bar_pos, scores, label=f'Team {i}', color=colors[i], width=bar_width)
ax.set_xticks(np.arange(n_labels))
ax.set_xticklabels(labels)
plt.title('Team scores')
plt.xlabel('Game')
plt.ylabel('Score')
plt.legend()
plt.show()
```

## Principle of Proportion Ink

실제 값과 그 값을 표현하는 그래픽의 면적은 비례해야한다.

![Principle of Proportion Ink](/assets/images/post/220203/boostcamp-data-viz-2/PrincipleOfProportionInk.png){: .align-center}

위 그래프에서 우측의 그래프는 `ylim`을 설정하여 y의 범위가 70부터 시작함. bar의 작은 차이를 쉽게 인지할 수 있지만 bar간의 차이를 매우 과장하여 올바르게 인지하기 힘들다. 작은 차이를 확실하게 표현하고싶다면 그래프의 세로를 더 길게해서 표현하자.

![LongLong](/assets/images/post/220203/boostcamp-data-viz-2/longlong_plot.png){: .align-center}

## 데이터 정렬

* 데이터 종류에 따라
  * 시계열 - 시간순
  * 수치형 - 크기순
  * 순서형 - 순서순
  * 명목형 - 범주의 값순으로

데이터의 정렬에 따라서도 인사이트를 발견할 수도 있으므로 데이터 시각화시에 인터렉티브를 제공하여 다양한 시각화 시도를 가능하게하거나 여러가지 방법으로 그린 plot을 제공하자.

## 공간활용

여백, 공간의 조절 - 가독성 향상

* `.xlim`, `.ylim` 조절 - plot 여백 조성
* `.spines[{'left'|'right'|'top'|'bottom'}].set_visible(False)` - 지정한 방향의 테두리 제거

```python
...
ax.spines['right'].set_visible(False)  # 지정한 방향의 테두리를 제거
ax.spines['top'].set_visible(False)

ax.margins(.1, .2)  # 좌우, 상하 여백 비율. default: .05
plt.legend()
...
```

![clean bar](/assets/images/post/220203/boostcamp-data-viz-2/clean_bar.png){: .align-center}
