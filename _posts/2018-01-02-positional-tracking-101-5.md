---

published: true
layout: post
title: "포지셔널 트래킹 101 - 5강. '실내 자율주행 로봇'과 PLS"
author: JH
categories: VR Indoor-positioning IPS
tags:  VR tech polariant virtual-reality positional-tracking 측위

---

지난 4강에서는 PLS(Polarized Light Sensing) 기술에 대해 자세히 알아보았다.

  * [포지셔널 트래킹 101 - 1강. '측위가 뭐임?']({{site.baseurl}}/vr/indoor-positioning/ips/2016/09/10/positional-tracking-101-1.html)
  * [포지셔널 트래킹 101 - 2강. '실내 무선 측위기술'의 원리]({{site.baseurl}}/vr/indoor-positioning/ips/2017/10/22/positional-tracking-101-2.html)
  * [포지셔널 트래킹 101 - 3강. '실내 무선 측위기술'의 방식]({{site.baseurl}}/vr/indoor-positioning/ips/2017/12/10/positional-tracking-101-3.html)
  * [포지셔널 트래킹 101 - 4강. 'PLS(Polarized Light Sensing) 기술'과 실내 측위]({{site.baseurl}}/vr/indoor-positioning/ips/2017/12/16/positional-tracking-101-4.html)


이번 시간에는 '포지셔널 트래킹 101'의 완결편으로 실내 측위와 결부되어 최근에 큰 주목을 받고 있는 '실내 자율주행 로봇'의 메카니즘을 알아보고 PLS 기술과 결부되어 시너지를 낼 수 있는 점과 더 나아가 PLS 기술 단독으로 이 분야에 확실히 기여할 수 있는 큰 비전을 공유하고자 한다.

'실내 자율주행 로봇'을 설명하기 전에 앞서 수많은 미디어들에서 말하는 '자율주행(Self-drive)'이 어떤 기술 요소로 구성되어 있는지부터 살펴보고 가자.

![자율주행(Self-drive)의 요소 기술]({{site.baseurl}}/images/self-driving-1.png)

자율주행을 가능케하는 기술은 크게 세가지의 영역으로 나눌 수 있다. 위의 그림에서 보이는 것처럼 SLAM(Simultaneous Localization and Mapping) 영역, Odometry 영역 그리고 Ground truth 영역이 그 것이다.

쉽게 말하면 로봇이 자율주행하기 위해서는 자체의 위치가 지도 상에서 어디인지를 파악(SLAM)하고 부드러운 주행을 위해 바퀴의 회전수 혹은 imu 센서를 통해 얻은 주행 정보(Odometry)를 얻으며 마지막으로 이 모든 정보를 정확하게 확인할 수 있는 참 값(Ground truth)을 필요로 하는 것이다. 다른 것은 직관적으로 이해가 가는데 굳이 SLAM 용어에 'Simultaneous(동시의)'가 붙는 이유는 무엇일까?

그것은 바로 로봇이 '임의의 공간'을 주행한다고 가정하기 때문이다. 정확한 지도가 주어져 있는 상황 혹은 항상 약속된 위치에서 로봇의 위치 추정을 시작한다면 Localization만 수행하면 되지만 지도도 주어져 있지 않고 지도에서의 로봇의 위치도 알 수 없을 때 로봇이 주변 환경을 센서로 감지해가면서 지도를 만들고 자신의 위치까지 추정하는 작업 해야하므로 '동시적'으로 해야한다는 것이다. 직관적으로도 알 수 있듯이 Localization이나 Mapping 둘 중 한가지만 해도 계산량이 벅찬데, 두 가지를 한꺼번에 하는 SLAM은 컴퓨팅 파워의 측면에서도 상당한 오버헤드가 있다.

![Google's Self-driving car](http://www.outsidethebeltway.com/wp-content/uploads/2010/10/google-car-drives-itself-nyt.jpg)

실외에서의 로봇이라고 하면 '자동차'이고 위에 보이는 그림처럼 자율주행차에 달려있는 수 많은 센서가 위에서 언급한 세 가지 영역에 속하는 것이다.

그런데, 이 자율주행을 '실내'에서 가능하게 하려면 저 세 가지 영역 중에 딱 한가지가 부족하다. 그것이 무엇일까?
바로 'Ground truth' 영역이다. 실외에서는 GPS가 그 역할을 도맡아주지만 실내에서는 동작하지 않고, 이를 대체할만한 기술이 마땅히 없었다. 바로 이 영역을 PLS 기술이 대체할 수 있는 것이다.

![실내 자율주행 로봇의 요소 기술]({{site.baseurl}}/images/self-driving-2.png)

위에 언급한 Ground truth의 부재로인한 현존하는 실내 자율주행에서의 문제점은 무엇이 있을까?

**1. 지도 작성용 로봇을 따로 두거나 지도 작성용 기능 별도 추가**

위에서 언급했듯이 SLAM에서 Localization과 Mapping을 동시에 한다는 것은 상당한 오버헤드가 있고 이를 가능케하는 센서 조합이 현재 수천만원을 호가함에 따라 상용화에 어려움이 있다. 그러므로 먼저 상세하게 지도작성을 해놓는 '지도 작성용' 로봇을 주기적으로 작동시키나 원하는 주행 형태에 따라 사람이 지도를 작성해두고, 실제 돌아다니는 로봇은 선작성된 지도를 기반으로 위치 측위를 하는 것이다. 쉽게 생각하면 구글맵이나 네이버지도가 없는 상황에서 '자율주행'을 하는 상황이 '실내'에서 벌어지고 있다고 생각하면 된다. 게다가 GPS마저 동작하지 않는다면 얼마나 어려운 상황이겠는가.

**2. 임의의 위치에서의 초기 시작시 '태그(Tag)'가 필요**

선작성된 지도가 있는 상황에서 그 지도를 가지고 localization을 바로 시작하기 위해서는 지도상에서의 첫 초기 위치를 로봇이 알아야하고 이를 위해서는 태그(Tag)나 랜드마크가 꼭 필요하다. 태그 형태는 RFID일 수도 있고 아래 동영상처럼 QR 코드 비슷하게 생긴 에이프릴 태그(April tag)일 수 있다. 하기 동영상은 실내 자율주행 청소용 로봇(Autoscrubber)으로 사업화를 하고 있으며 얼마 전 소프트뱅크 비전 펀드(Vision Fund)로부터 114 million USD를 투자 받은 유망한 스타트업 [Brain corp.](https://www.braincorp.com/)의 데모이다.

{% include youtube_player.html id="htcQgXQZ7sA" %}

**3. 비슷한 구조가 반복되는 환경이나 잦은 가구 배치의 변경에 매우 취약**

실내 자율주행 로봇의 SLAM 기술에서 라이다(Lidar)의 역할은 주요 동선의 '벽'간의 거리를 정밀하게 측위하는 것이다. '벽'간의 거리를 통해서 로봇 주변의 벽의 형태를 가지고 전체 맵에서 위치를 찾아내는데, 비슷한 구조가 반복되는 환경이나 잦은 가구 배치로 인하여 벽이 계속 변화하는 경우, 선작성된 지도와 로봇이 측위하는 벽의 형태가 판이하게 다르므로 localization의 어려움이 있다.

{% include youtube_player.html id="MgABYkCo3x4" %}

위의 동영상에서 27초 부근에서 갑자기 추정하는 위치의 후보가 늘어나는 모습을 볼 수 있는데 바로 이 점이 문제점인 것이다.

Ground truth의 부재가 만들어내는 위의 세 가지 문제는 임의의 위치에서 자율적으로 돌아다니고 더 나아가 관제센터에서 정밀하게 위치를 제어하기 위해서는 필히 해결되어야하는 점이며 이에 PLS 기술이 명확히 기여할 수 있는 것이다. 단일 광원하에서 빛이 도달하는 범위내에서는 수많은 로봇의 절대적인 위치를 각각 계산해낼 수 있는 본 기술은 실내 자율 주행의 'Ground truth'를 제공해줌으로서 '실내의 GPS'로 거듭날 것이다.
