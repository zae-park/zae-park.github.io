---
title: Python package 배포하기
date: 2023-02-01 19:29:13
tags: [package, build, PyPI, pip, poetry]
categories: dev-env
---
# Python Package 배포하기

대부분의 python 프로젝트에서 third party package는 필수적이다. 또, 배포된 프로젝트는 다시 다른 사용자에게 third party package가 된다. 그럼 내 python 프로젝트를 **어디에, 어떻게** 배포할 수 있을까?

## PyPI - The Python Package Index

![Untitled](resource/Py_package/Untitled.png)

PyPI는 다양한 python package들을 관리하는 저장소로 직접 package들을 [호스팅](https://pypi.org/simple/)하고 있다. 이 글을 작성하는 시점에서 약 43만개 정도의 package들이 배포되고 있는데, 만약 어떤 프로젝트를 배포하고 싶은 욕심이 있다면 이름을 선점하는 것도 고려해보시라. 내가 원하는 멋진 프로젝트 이름은 대게 남들이 이미 생각해봤을 법한 이름들이더라. 각설하고, 그만큼 프로젝트의 이름을 유니크하게 잘 짓는 것도 중요하다는 듯이다. 실제로 이름이 helloworld로 시작하는 package들이 이렇게나 많다.

![Untitled](resource/Py_package/Untitled_1.png)

## Packaging??

앞서 PyPI는 package의 저장소라고 했다. 이 말은 내 프로젝트가 먼저 **packaging**되어야 한다는 뜻이며, 프로젝트 내의 모듈, 기타 파일들이 **실행 혹은 설치 가능한 형태**로 제공되어야 한다는 뜻이다. PyPI에 package를 업로드하기 위해서는 `wheel`파일([PEP-427](https://peps.python.org/pep-0427/))이 필요하다. 이러한 과정을 **build** (혹은 compile + build) 한다고 하기도 한다.

### Build - setuptools

세상에는 이미 잘 만들어진 build tool이 매우 많다. 하지만 사용할 수 있는 tool이 많다는 점은 썩 그리 좋은 상황은 아니다. 어떤 tool을 써야 좋을까?

최초의 python 표준 packaging 라이브러리는 distutils이다. `setup.py`라는 **스크립트**를 사용하여 메타데이터를 저장하는 방식을 제안했으며, 몇 가지 불편한 점들을 보완하며 distribute, setuptools, distutils2와 같은 tool들이 등장하기 시작했다. 그 중 setuptools가 distribute와 통합하며 많이 사용되는 듯 했지만… 현재는 legacy 취급을 받고 있다. [setuptools를 삭제하는 PR](https://github.com/pypa/setuptools/pull/2544/files)

setuptools가 legacy로 취급받는 가장 큰 요인은 `setup.py`가 build-time dependency은 무시하고 **run-time dependency만 관리**하기 때문이다. 즉, `setup.py`의 파싱을 위해 setuptools가 재귀적으로 필요하다는 문제가 발생한다. 마치 순환논법의 오류같다.

- 뭐라구요??
    - my_package에서 **setuptools-v1.1**이 필요하다.
    - 그래서 `setup.py`에 **setuptools-v1.1**이 필요하다고 적어두었다.
    - 하지만 현재 설치된 버전은 **setuptools-v1**이여서 이 package를 build할 수 없다. 즉, `[setup.py](http://setup.py)`는 build-time dependency를 관리해주지 않는다.

```python
from setuptools import setup, find_packages

setup(
    name='my_package',
    version='0.1.2',
    author='zae-park',
    url='https://github.com/zae-park/my_package',
    description='Example package.',
    long_description='',
    package_data={'my_package': ['./resource/ex.*']},
    include_package_data=True,
    packages=find_packages(exclude=["*.tests", "*.tests.*", "tests.*", "tests"]),
    install_requires=['A', 'B', 'C'],
    zip_safe=False,
)
```

### pyproject.toml

위에서 문제가 되던 `setup.py`를 어떻게 대체할 수 있을까?

python은 [PEP-517](https://peps.python.org/pep-0517/)에서 setup.py-style 이라는 legacy를 없애고 대신 `pyproject.toml`을 사용하자고 제안했다. 메타데이터를 스크립트가 아닌 **config 파일**로 관리하자는 뜻이다. 이 config 파일에 어떤 build tool을 사용할 것인지 명시하기만 한다면, setuptools를 계속 사용하는 것도 가능하다.

큰 legacy가 잘 해결되고 있는 듯 하지만, build tool마다 원하는 config 파일의 format이 제각각인 문제가 여전히 남아있다. 이 format에 대한 논의가 종결된 지는 꽤 되었지만 ([PEP-621](https://peps.python.org/pep-0621/)) 아직 모든 build tool에 적용되지는 않았다. 현 시점에서 **flit**만 공식적으로 이를 지원하고 있다. 따라서 setuptools, poetry 등을 사용한다면 이들이 어떤 format을 원하는지 잘 살펴봐야 한다. poetry의 경우, `poetry new .` 명령어를 통해 기본 format을 제공받을 수 있다.

```toml
[tool.poetry]
name = "my_package"
version = "0.1.2"
description = "Example package."
authors = ["zae-park <zae_park@email.com>"]

[tool.poetry.dependencies]
python = "^3.8"

[tool.poetry.dev-dependencies]
pytest = "^5.2"

[build-system]
requires = ["poetry>=0.12"]
build-backend = "poetry.masonry.api"
```

## ~~선점하기~~ 배포하기

모순적인 말이지만, private repository를 배포하고 싶은 욕심이 있다. 오픈소스로 배포하면서 동시에 제한된 소수 인원만 사용할 수 있게 하고 싶은 것이다. *이게 어떻게 오픈소스…?*

어떻게 해결해야 좋을지는 차차 이야기하기로 하고, 이번 글의 실습 격으로 poetry를 사용하여 project를 배포해보자. 프로젝트의 이름은 fakepack으로 정했고, 다행히 배포중인 프로젝트들과 이름이 겹치지 않았다. 빨리 선점하자.

```toml
[tool.poetry]
name = 'fakepack'
version = '0.0.1'
description = 'Fake packaging'
authors = ["zae-park <tom941105@gmail.com>"]

[tool.poetry.dependencies]
python = "^3.8"

[tool.poetry.dev-dependencies]
pytest = "^5.2"

[build-system]
requires = ["poetry>=0.12", 'numpy']
build-backend = "poetry.masonry.api"
```

위와 같이 간단하게 pyproject.toml을 작성했다. build-system을 보면 requires에 poetry가 포함되어 build-time dependency까지 관리되는 것을 확인할 수 있다.

이제 아래의 명령어로 배포하면 끝이다.

```powershell
poetry new fakepack
poetry publish --build
```

![Untitled](resource/Py_package/Untitled_2.png)

![Untitled](resource/Py_package/Untitled_3.png)

poetry를 사용하면 twine과 같은 upload tool을 따로 사용하지 않고 이렇게 build부터 publish까지 해결할 수 있다. 이제 무료로 PyPI의 리소스를 사용하는 만큼 이 package에 대한 책임이 생겼으니 꼭 잘 마무리해야겠다.

### Reference

[Python Package 생성 및 배포하기](https://devocean.sk.com/blog/techBoardDetail.do?ID=163566)

[setup.py 멈춰!](https://tech.buzzvil.com/blog/setup.py-%EB%A9%88%EC%B6%B0/)

[파이썬 패키징의 과거, 현재, 그리고 미래](https://ryanking13.github.io/2021/07/11/python-packaging.html#fn:3)

[Poetry 에서 바로 실행 명령 가능한 패키지 만들기](https://dailyheumsi.tistory.com/252)