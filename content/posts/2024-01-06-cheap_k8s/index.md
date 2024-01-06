+++
title = 'how to run cheap k8s'
date = 2024-01-06T18:50:30+09:00
math = true
toc = true
bold = true
draft = false
tags = ["kubernetes", "k8s", "infra", "system_engineering", "korean", "scalability", "gcp", "terraform"]
+++

## trials

쿠버네티스를 로컬에서 구동하기 위해 제가 시도해본 방법은 다음과 같습니다.

1. **라즈베리파이 클러스터 구축** -> cluster 수준으로 도입하려니 초기 비용이 많이 들기도하고 이리저리 집 안에서 선이 나뒹구며 집이 더러워지는 것을 견디기 힘들어서 포기.

2. **구형 android 기기에 termux 위에 k3s 설치** -> termux[^1]는 execve[^2]를 기반으로 한 linux emulator입니다. 따라서 쿠버네티스나 그 경량화 버전 등을 설치하기 위해서는 기기의 루트 권한이 필요합니다. 그러나 안드로이드는 기본적으로 루트 권한에 대한 접근을 막고 있어 rooting이 요구되므로 해당 방법은 현실적으로 원하는 목적을 달성하기에 기술적 비용이 너무 많이 투입됩니다. ROI가 나오지 않아 포기.

3. **개인 컴퓨터에 VM 기반 마스터 노드 + 워커 노드 구성** -> 이 방법은 무난하게 동작했고, 관련 도서에서도 vagrant[^3] 스크립트를 통해 이미 구성해놓았기도 해서 편리한 편입니다. 그러나 제 노트북이 발열이 심해져서 사용을 포기했습니다. 쿠버네티스를 구성하기 위한 최소 리소스의 조건이 1 GB RAM, 2 CPU 이므로 노트북의 리소스를 쿠버네티스에 할당하면서 개발 작업을 수행하기에는 사용하기에는 무리가 있었습니다. 포기.

4. **개인 컴퓨터를 마스터 노드로 VM을 워커 노드로 구성** -> 로컬 환경에 마스터 노드에 필요한 리소스를 설치하는 것이 탐탁치 않았습니다. 특히 BSD 기반인 mac에서 굳이 m1 arm용으로 빌드된 바이너리를 하나씩 찾아서 구성하는 과정이 험난할 것으로 예상되었으므로 이것도 ROI 문제로 포기.

[^1]: https://termux.dev/en/
[^2]: https://www.man7.org/linux/man-pages/man2/execve.2.html
[^3]: https://www.vagrantup.com/

결국 간단한 클라우드 기반의 쿠버네티스를 구축하기로 결정했습니다. digital ocean, vultr 등 control plane을 무료로 제공하면서 사용을 유도하는 (비교적 중소 규모의) 클라우드 업체가 존재하지만, 사용하지 않기로 하였습니다. DB 등은 managed service를 이용하게 될텐데 의도치 않은 multi cloud 환경을 굳이 만들고 싶진 않았습니다.

따라서 1계정 당 1개의 control plane을 무료로 제공해주는 gcp 기반으로 구축하기로 결정하였습니다.

## 비용 최소화를 위한 구성

...keep going!
