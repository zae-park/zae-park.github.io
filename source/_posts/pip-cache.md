---
title: pip cache
date: 2023-01-16 23:13:31
tags: [pip, environment]
categories: dev-env
mathjax: true 
---

# pip cache

이것저것 쑤셔보며 개발환경을 마구 생성하던 어느 날 갑자기 C 드라이브의 용량이 부족하다는 경고를 받았다. 1TB, 2TB SSD를 2개나 사용하고, 개발용 하드와 DB용 하드를 따로 사용하고 있어서 평생 마주하지 않을 줄 알았는데… 대체 어떤 파일들이 범인일까?

![](resource/pip_cache/Untitled.png)

C 드라이브 내의 폴더들의 용량을 찍어보며 범인을 색출했더니…
당연히 downloads 폴더가 주범이였지만, 생각지도 못한 공범이 있었다.

![pip.png](resource/pip_cache/pip.png)

이게 무엇??

찾아보니 pip를 사용해서 다운받은 package들을 캐싱해놓은 파일이라 한다. 하위 디렉토리는 3가지가 있는데 내부 tree 구성이 조금씩 다르다.

- http - 이름이 `[0-9a-z]` char인 계층 노드로 구성되어 있고 leaf에는 hash 값이 적힌 파일.
- selfcheck - 단일 계층이고 여러 hash file들로 구성.
- wheels - 이름이 `ASCII` char인 계층 노드로 구성되어 있고 leaf에는 [hash]-[whl 파일].

그렇다면 이 캐시들은 왜 필요할까? 만약 캐시가 쌓이고 쌓이면 C 드라이브에 누적되는 게 좋을까? 캐시를 지워야하는 경우는 없을까?

### 캐싱 옵션

```
# pip -h

--cache-dir <dir>           Store the cache data in <dir>.
--no-cache-dir              Disable the cache.
```

pip에서 제공하는 documentation을 참조하면 `--cache-dir` , `--no-cache-dir` 의 명령어로 캐싱 여부를 선택할 수 있다. 여기까지는 pip 유저라면 초반에 많이 봐왔을 document이다.

그런데 package를 캐시하려면 인자로 `<dir>` 이 필요하다. 여태 사용해본적 없는 명령어인데 내 C 드라이브에 22GB나 들어찼다는 것은 default path가 있다는 뜻이겠다. 그럼 path를 지정할 수 있지 않을까?

```
# pip cache -h

Usage:
  pip cache dir
  pip cache info
  pip cache list [<pattern>] [--format=[human, abspath]]
  pip cache remove <pattern>
  pip cache purge

Description:
  Inspect and manage pip's wheel cache.

  Subcommands:

  - dir: Show the cache directory.
  - info: Show information about the cache.
  - list: List filenames of packages stored in the cache.
  - remove: Remove one or more package from the cache.
  - purge: Remove all items from the cache.

  ``<pattern>`` can be a glob expression or a package name.

Cache Options:
  --format <list_format>      Select the output format among: human (default) or abspath
```

더 자세한 documentation를 참조하면 위와 같이 아주 친절히 알려준다. 

- pip cache dir
    
    `c:\***\***\appdata\local\pip\cache` - 용량을 은근 잡아먹던 캐시 경로가 나온다.
    
- pip cache info
    
    이름에 맞게 자세한 정보들을 제공한다. 정말로 22GB를 잡아먹고 있으면서 몇개의 http 파일과 wheel 파일을 캐싱중인지도 보여준다. 이런!
    
    ```
    Package index page cache location: c:\***\***\appdata\local\pip\cache\http
    Package index page cache size: 23772.7 MB
    Number of HTTP files: 832
    Wheels location: c:\***\***\appdata\local\pip\cache\wheels
    Wheels size: 2.5 MB
    Number of wheels: 30
    ```
    
- pip cache list (& remove)
    
    이 명령어들은 `pattern`이라는 추가 인자를 받는데, glob 표현식 혹은 package name이라 한다. 또, list에서는 `format` 인자를 추가로 받는데, abs는 cache된 package들의 절대경로, human은 package들의 이름이다.
    
    ```
    # pip cache list gem
    
    Cache contents:
     - gem-0.1.12-py3-none-any.whl (15 kB)
    ```
    
    ```
    # pip cache list --format abspath
    
    c:\users\user\appdata\local\pip\cache\wheels\fe\df\1c\53eb0b5ea5052d2d39138ea18c08b07e81502f7c2edf0a4064\gem-0.1.12-py3-none-any.whl
    ```
    
    ```
    # pip cache remove gem
    
    Files removed: 1
    ```
    
- pip cache purge
    
    설명 그대로 캐시를 삭제한다. 나는 이 캐시파일을 다른 경로로 옮기고 싶으니 궁금한 사람은 직접해보세요.
    

### 캐싱 경로 변경

목표는 `pip cache dir` 명령어에서 변경한 경로가 출력되는 것이다. 남은 용량을 고려하면 E 드라이브로 경로를 바꾸는 것이 좋아보이지만… 실제로 C 드라이브와 D 드라이브가 같은 SSD이니 캐시의 장점을 유지하기 위해 같은 디바이스인 D 드라이브로 경로를 수정해준다.

안타깝게도 pip 명령어로는 경로를 수정할 수 없다. 그럼 어딘가에 저장된 file을 수정해야 할까? `pip config debug -v` 로 config 파일의 위치를 알 수 있지만 안타깝게도 효과는 없었다. 정답은 **환경변수**이다. 아래와 같이 새 환경변수를 추가하고 `pip cache dir` 로 수정된 경로를 확인해보시라!

![env_path.png](resource/pip_cache/env_path.png)

### 여담

게임기가 아닌 컴퓨터를 처음 만지던 시절 환경변수를 왜 추가해야하는지, 시스템 변수와 사용자 변수 중 어디에 추가해야하는지 아무도 알려주지 않더라. 그냥 추가하라 하거나 상관없다고 하거나.

예를 들면, 시스템 변수는 전역변수이고 사용자 변수는 로컬 변수이다. 시스템 변수에 등록된 환경변수들은 같은 PC의 다른 사용자들도 사용하는 변수이고, 사용자 변수는 현재 사용자만 사용할 수 있다. 당연히 전역변수는 조심히 설정해야 한다. 필요한 경우에만.